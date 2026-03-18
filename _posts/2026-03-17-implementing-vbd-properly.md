---
title: 'Implementing VBD Damping Properly'
date: 2026-03-17
permalink: /posts/2026/03/implementing-vbd-properly/
tags:
  - physics-simulation
  - VBD
  - computer-graphics
  - SIGGRAPH
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
---

Vertex Block Descent (VBD) is a physics solver we published at SIGGRAPH 2024 for elastic body dynamics. It offers unconditional stability, excellent GPU parallelism, and fast convergence to implicit Euler solutions. While the paper covers the formulation comprehensively, actually implementing VBD correctly---especially the damping---turns out to be subtler than it first appears. This post discusses the key pitfalls and how to get them right, based on lessons learned during development with NVIDIA Warp.

## What Is VBD, in a Nutshell?

VBD solves the variational form of implicit Euler:

$$\mathbf{x}^{t+1} = \underset{\mathbf{x}}{\operatorname{argmin}} \; G(\mathbf{x}) = \frac{1}{2h^2} \| \mathbf{x} - \mathbf{y} \|_M^2 + E(\mathbf{x})$$

Instead of assembling and solving a massive global linear system (as Newton's method would), VBD updates **one vertex at a time**, solving a tiny 3&times;3 local system:

$$\mathbf{H}_i \, \Delta\mathbf{x}_i = \mathbf{f}_i$$

where $\mathbf{H}_i$ is the local Hessian and $\mathbf{f}_i$ is the total force on vertex $i$, both assembled only from force elements that touch vertex $i$. This is essentially **block Gauss-Seidel** on the vertex positions. Each local solve is cheap (a 3&times;3 analytical inverse), and because we color vertices rather than elements, we typically need only 6--9 colors for parallelization---an order of magnitude fewer than element-based coloring.

The critical guarantee: every local solve that reduces $G_i$ also reduces the global energy $G$, giving us **unconditional stability** even with a single iteration per time step.

## The Damping Trap: Why Na&iuml;ve Rayleigh Damping Breaks Physics

The paper describes Rayleigh stiffness-proportional damping as modifying the force and Hessian:

$$\mathbf{f}_i = -\frac{m_i}{h^2}(\mathbf{x}_i - \mathbf{y}_i) - \sum_{j \in \mathcal{F}_i} \frac{\partial E_j}{\partial \mathbf{x}_i} - \left(\sum_{j \in \mathcal{F}_i} \frac{k_d}{h} \frac{\partial^2 E_j}{\partial \mathbf{x}_i^2}\right)(\mathbf{x}_i - \mathbf{x}_i^t)$$

This looks straightforward: take the stiffness Hessian, scale by $k_d/h$, multiply by the displacement (which approximates $h \cdot v$), and add to the force. However, there is a critical implementation subtlety that is easy to miss.

### The Bug: Damping That Kills Free Fall

A na&iuml;ve implementation might do something like:

```python
displacement = x_prev - x_current
h_d = hessian * (damping / dt)
f_d = h_d * displacement
```

This applies damping proportional to the **absolute velocity** of the vertex. The problem is immediate: a freely falling object has nonzero absolute velocity, so this damping fights gravity. Objects sink slower than they should. Stacked objects behave as if embedded in molasses.

The mathematical reason: for full Rayleigh damping, the damping force on vertex $i$ is:

$$\mathbf{f}_{d,i} = -\beta \sum_j \mathbf{K}_{ij} \mathbf{v}_j$$

If the entire system translates rigidly ($\mathbf{v}_j = \mathbf{v}$ for all $j$), then $\mathbf{f}\_{d,i} = -\beta (\sum\_j \mathbf{K}\_{ij}) \mathbf{v}$. For any translation-invariant energy, $\sum\_j \mathbf{K}\_{ij} = \mathbf{0}$, so the damping force vanishes. But in VBD, we only have the **diagonal block** $\mathbf{K}\_{ii}$, and $\mathbf{K}\_{ii} \mathbf{v} \neq \mathbf{0}$ in general.

### The Fix: Damp the Internal Variable, Not the Position

The solution is to formulate damping in terms of **internal variables**---quantities that are inherently invariant to rigid motion.

**For volumetric elasticity**, the internal variable is the deformation gradient $\mathbf{F} = \mathbf{D}\_s \mathbf{D}\_m^{-1}$, where $\mathbf{D}\_s = [\mathbf{x}\_1 - \mathbf{x}\_0, \mathbf{x}\_2 - \mathbf{x}\_0, \mathbf{x}\_3 - \mathbf{x}\_0]$. Its rate of change is:

$$\dot{\mathbf{F}} = \dot{\mathbf{D}}_s \mathbf{D}_m^{-1}$$

where $\dot{\mathbf{D}}\_s = [\mathbf{v}\_1 - \mathbf{v}\_0, \mathbf{v}\_2 - \mathbf{v}\_0, \mathbf{v}\_3 - \mathbf{v}\_0]$. Notice: $\dot{\mathbf{F}}$ depends only on **relative velocities**. If all four vertices move with the same velocity, $\dot{\mathbf{D}}\_s = \mathbf{0}$, so $\dot{\mathbf{F}} = \mathbf{0}$. No damping.

The damping stress is then:

$$\mathbf{P}_{\text{damp}} = k_d \cdot \frac{\partial^2 E}{\partial \mathbf{F}^2} : \dot{\mathbf{F}}$$

And the force on vertex $i$ is assembled as $\mathbf{f}\_{d,i} = -V\_0 \, \mathbf{G}\_i^T \text{vec}(\mathbf{P}\_{\text{damp}})$, where $\mathbf{G}\_i = \partial \text{vec}(\mathbf{F}) / \partial \mathbf{x}\_i$ is the 9&times;3 matrix mapping vertex displacements to flattened deformation gradient changes.

**For dihedral-angle bending**, the internal variable is the dihedral angle $\theta$ between two adjacent triangles. The angular velocity is:

$$\dot{\theta} = \sum_{j=0}^{3} \frac{\partial \theta}{\partial \mathbf{x}_j} \cdot \mathbf{v}_j$$

Since $\theta$ depends only on relative positions, we have $\sum\_j \frac{\partial \theta}{\partial \mathbf{x}\_j} = \mathbf{0}$. For rigid translation ($\mathbf{v}\_j = \mathbf{v}$), $\dot{\theta} = \mathbf{v} \cdot \sum\_j \frac{\partial \theta}{\partial \mathbf{x}\_j} = 0$. The damping force is:

$$\mathbf{f}_{d,i} = -c \, \dot{\theta} \, \frac{\partial \theta}{\partial \mathbf{x}_i}$$

Think of it like a door hinge with a damper: the damper resists opening/closing, but if you translate the entire door frame, the damper does nothing.

**For collision damping**, the internal variable is the gap distance $d$ between contact points. Using barycentric weights $b\_j$ that sum to zero ($\sum\_j b\_j = 0$), the gap rate is:

$$\dot{d} = \sum_j b_j \, (\hat{\mathbf{n}} \cdot \mathbf{v}_j)$$

Again, rigid translation produces $\dot{d} = \hat{\mathbf{n}} \cdot \mathbf{v} \cdot \sum\_j b\_j = 0$.

### The General Principle

For **any** energy $E = f(q)$ where $q$ is a translation-invariant internal variable:

$$\sum_j \frac{\partial q}{\partial \mathbf{x}_j} = \mathbf{0}$$

This guarantees $\dot{q} = 0$ for rigid translation, and therefore zero damping force. If $q$ is also rotation-invariant (like edge lengths and dihedral angles), then rigid rotation is also undamped.

The pattern for VBD is always:
1. **Force**: compute using all vertices in the stencil (exact relative velocity information)
2. **Hessian**: use only the diagonal block (the standard VBD approximation)

This asymmetry is fundamental to VBD: forces are exact, Hessians are approximate. Off-diagonal coupling is recovered through iteration.
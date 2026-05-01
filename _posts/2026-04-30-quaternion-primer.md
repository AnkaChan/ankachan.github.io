---
title: 'Quaternion Math for Rigid Body Simulation'
date: 2026-04-30
permalink: /posts/2026/04/quaternion-primer/
tags:
  - physics-simulation
  - rigid-body
  - computer-graphics
  - math
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
---

A practical primer covering exactly the quaternion operations used in rigid body simulation, with reference to the [Newton](https://github.com/newton-physics/newton) AVBD implementation. No proofs, just what you need to read the code.

---

## What is a quaternion

Four numbers: three "vector" components and one "scalar" component.

$$\mathbf{q} = (x,\, y,\, z,\, w) \quad\text{or equivalently}\quad \mathbf{q} = (\mathbf{v},\, w)$$

A **unit quaternion** ($\lVert\mathbf{q}\rVert = 1$) represents a 3D rotation. The identity rotation is $(0, 0, 0, 1)$.

---

## Quaternion multiplication

Given $\mathbf{a} = (\mathbf{a}\_v,\, a\_w)$ and $\mathbf{b} = (\mathbf{b}\_v,\, b\_w)$:

$$\mathbf{a} \otimes \mathbf{b} = \big(a_w\,\mathbf{b}_v + b_w\,\mathbf{a}_v + \mathbf{a}_v \times \mathbf{b}_v,\;\; a_w\,b_w - \mathbf{a}_v \cdot \mathbf{b}_v\big)$$

This is **not commutative**: $\mathbf{a}\otimes\mathbf{b} \neq \mathbf{b}\otimes\mathbf{a}$ in general. Order matters, just like matrix multiplication. In fact quaternion multiplication corresponds exactly to multiplying the equivalent $3\times 3$ rotation matrices.

---

## Conjugate and inverse

$$\mathbf{q}^* = (-\mathbf{v},\, w) = (-x,\, -y,\, -z,\, w)$$

For a unit quaternion, $\mathbf{q}^{-1} = \mathbf{q}^*$. This represents the **opposite rotation**.

---

## How a quaternion encodes a rotation

A rotation by angle $\theta$ about unit axis $\hat{\mathbf{n}}$ is:

$$\mathbf{q} = \big(\sin(\theta/2)\,\hat{\mathbf{n}},\;\; \cos(\theta/2)\big)$$

The half-angle appears because quaternions rotate vectors via the **sandwich product** (next section), which applies the rotation from both sides---left and right---each contributing half the angle.

This means $\mathbf{q}$ and $-\mathbf{q}$ represent the **same rotation** (double cover of $SO(3)$). This is why code often checks `if q.w < 0: q = -q` to pick the shorter path.

---

## Rotating a vector

To rotate vector $\mathbf{v}$ by quaternion $\mathbf{q}$, embed $\mathbf{v}$ as a pure quaternion $(\mathbf{v}, 0)$ and sandwich:

$$\mathbf{v}' = \big(\mathbf{q}\otimes(\mathbf{v}, 0)\otimes\mathbf{q}^*\big)_\text{vec}$$

In code this is `quat_rotate(q, v)`. The efficient formula (no full quaternion multiply) is:

$$\mathbf{t} = 2\,(\mathbf{q}_v \times \mathbf{v}), \qquad \mathbf{v}' = \mathbf{v} + w\,\mathbf{t} + \mathbf{q}_v \times \mathbf{t}$$

The **inverse rotation** (world to body) is `quat_rotate(conjugate(q), v)`, which in Warp is `quat_rotate_inv(q, v)`.

---

## Composing rotations

To apply rotation $\mathbf{a}$ **then** rotation $\mathbf{b}$:

$$\mathbf{q}_\text{combined} = \mathbf{b} \otimes \mathbf{a}$$

The rotation applied **first** goes on the **right**. Same convention as matrices: $(\mathbf{B}\mathbf{A})\mathbf{v} = \mathbf{B}(\mathbf{A}\mathbf{v})$.

---

## Relative rotation

Given two orientations $\mathbf{q}\_\text{cur}$ and $\mathbf{q}\_\text{target}$, the rotation **from current to target** is:

$$\mathbf{q}_\delta = \mathbf{q}_\text{cur}^{-1} \otimes \mathbf{q}_\text{target}$$

This $\mathbf{q}\_\delta$ is in $\mathbf{q}\_\text{cur}$'s **body frame**. If you stand in the body frame of $\mathbf{q}\_\text{cur}$, $\mathbf{q}\_\delta$ tells you how much more to rotate to reach $\mathbf{q}\_\text{target}$.

If you instead compute:

$$\mathbf{q}_{\delta,\text{world}} = \mathbf{q}_\text{target} \otimes \mathbf{q}_\text{cur}^{-1}$$

you get the same physical rotation, but expressed in **world frame**.

This is the key body-vs-world distinction in the Newton code:

```python
# Body-frame delta (Newton uses this)
q_delta = quat_inverse(rot_current) * rot_star

# World-frame delta (the AVBD demo uses this)
q_delta = rot_current * quat_inverse(rot_star)
```

---

## Quaternion to rotation vector

Extract the axis and angle from a quaternion:

$$\theta = 2\,\arccos(w), \qquad \hat{\mathbf{n}} = \frac{\mathbf{v}}{\sin(\theta/2)}$$

The **rotation vector** packs both into one $\mathbb{R}^3$ vector:

$$\boldsymbol{\theta} = \hat{\mathbf{n}}\,\theta$$

Its magnitude is the angle, its direction is the axis. This is what `quat_to_axis_angle` followed by `axis * angle` does in Newton, and it is the natural quantity for the inertial spring $\mathbf{f}\_\text{ang} = \mathbf{I}\_\text{world}\,\boldsymbol{\theta}/h^2$.

---

## Rotation vector back to quaternion

Given rotation vector $\boldsymbol{\theta} \in \mathbb{R}^3$:

$$\theta = \lVert\boldsymbol{\theta}\rVert, \qquad \hat{\mathbf{n}} = \boldsymbol{\theta}/\theta, \qquad \mathbf{q} = \big(\sin(\theta/2)\,\hat{\mathbf{n}},\;\cos(\theta/2)\big)$$

For small angles, the **small-angle approximation** avoids the trig:

$$\mathbf{q} \approx \text{normalize}\!\big(\boldsymbol{\theta}/2,\; 1\big)$$

This is the `_USE_SMALL_ANGLE_APPROX` path in Newton's `solve_rigid_body`.

---

## Angular velocity and dq/dt

If a body has world-frame angular velocity $\boldsymbol{\omega}$, its orientation changes as:

$$\dot{\mathbf{q}} = \tfrac{1}{2}\,\widetilde{\boldsymbol{\omega}} \otimes \mathbf{q}$$

where $\widetilde{\boldsymbol{\omega}} = (\boldsymbol{\omega}, 0)$ is $\boldsymbol{\omega}$ embedded as a pure quaternion (zero scalar part).

**Why?** A small rotation by angle $\lVert\boldsymbol{\omega}\rVert\,\Delta t$ about axis $\boldsymbol{\omega}/\lVert\boldsymbol{\omega}\rVert$ is the quaternion:

$$\delta\mathbf{q} = \Big(\sin\!\big(\tfrac{\lVert\boldsymbol{\omega}\rVert\Delta t}{2}\big)\,\frac{\boldsymbol{\omega}}{\lVert\boldsymbol{\omega}\rVert},\;\; \cos\!\big(\tfrac{\lVert\boldsymbol{\omega}\rVert\Delta t}{2}\big)\Big) \;\approx\; \big(\boldsymbol{\omega}\,\Delta t/2,\; 1\big)$$

The new orientation is $\delta\mathbf{q}\otimes\mathbf{q}$ (left-multiply = world frame), so:

$$\mathbf{q}(t+\Delta t) - \mathbf{q}(t) = (\delta\mathbf{q} - \mathbf{1})\otimes\mathbf{q} = \big(\boldsymbol{\omega}\,\Delta t/2,\; 0\big)\otimes\mathbf{q}$$

$$\dot{\mathbf{q}} = \tfrac{1}{2}\,(\boldsymbol{\omega}, 0) \otimes \mathbf{q}$$

**Euler integration** of this gives:

$$\mathbf{q}^{n+1} = \text{normalize}\!\big(\mathbf{q}^n + \tfrac{h}{2}\,\widetilde{\boldsymbol{\omega}} \otimes \mathbf{q}^n\big)$$

which is exactly `wp.normalize(r0 + wp.quat(w1, 0.0) * r0 * 0.5 * dt)` in Newton.

**Body-frame angular velocity** would use right-multiplication instead:

$$\dot{\mathbf{q}} = \tfrac{1}{2}\,\mathbf{q} \otimes (\boldsymbol{\omega}_\text{body}, 0)$$

---

## Rotation matrix from quaternion

Sometimes you need the $3\times 3$ rotation matrix, e.g. to compute $\mathbf{I}\_\text{world} = \mathbf{R}\,\mathbf{I}\_\text{body}\,\mathbf{R}^T$. Given $\mathbf{q} = (x, y, z, w)$:

$$\mathbf{R} = \begin{bmatrix} 1-2(y^2+z^2) & 2(xy-wz) & 2(xz+wy) \\\\ 2(xy+wz) & 1-2(x^2+z^2) & 2(yz-wx) \\\\ 2(xz-wy) & 2(yz+wx) & 1-2(x^2+y^2) \end{bmatrix}$$

In Warp this is `quat_to_matrix(q)`.

---

## Summary: operations used in Newton's rigid body solver

| Code | Math | What it does |
|------|------|-------------|
| `quat_rotate(q, v)` | $\mathbf{R}\,\mathbf{v}$ | Rotate vector to world frame |
| `quat_rotate_inv(q, v)` | $\mathbf{R}^T\mathbf{v}$ | Rotate vector to body frame |
| `quat_inverse(q_cur) * q_star` | $\mathbf{R}\_\text{cur}^T\,\mathbf{R}\_\star$ | Relative rotation in body frame |
| `quat_to_axis_angle(q)` $\to$ `axis*angle` | $\boldsymbol{\theta} = \log(\mathbf{q})$ | Quaternion to rotation vector |
| `quat(half_w, 1.0)` normalized | $\exp(\Delta\boldsymbol{\omega}/2)$ | Small rotation vector to quaternion |
| `dq * q_current` | $\delta\mathbf{R}\,\mathbf{R}\_\text{cur}$ | Apply world-frame rotation increment |
| `r0 + wp.quat(w1,0)*r0*0.5*dt` | $\text{normalize}(\mathbf{q} + \tfrac{h}{2}\widetilde{\boldsymbol{\omega}}\otimes\mathbf{q})$ | Integrate angular velocity one step |
| `quat_to_matrix(q)` | $\mathbf{R}$ | $3\times 3$ rotation matrix for $\mathbf{R}\,\mathbf{I}\,\mathbf{R}^T$ |

---

*See also: [Rigid Body Dynamics with VBD, Section I](/posts/2026/03/vbd-rigid-body-section-1/) for the full AVBD derivation that uses these operations.*

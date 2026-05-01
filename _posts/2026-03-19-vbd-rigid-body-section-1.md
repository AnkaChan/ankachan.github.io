---
title: 'Rigid Body Dynamics with VBD, Section I: Free Bodies'
date: 2026-03-19
permalink: /posts/2026/03/vbd-rigid-body-section-1/
tags:
  - physics-simulation
  - VBD
  - rigid-body
  - computer-graphics
  - SIGGRAPH
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
---

In the [VBD paper](https://doi.org/10.1145/3658179) (SIGGRAPH 2024), we briefly discuss extending Vertex Block Descent to rigid body simulation. The idea is natural: instead of updating a single vertex with 3 DoF, you update an entire rigid body with 6 DoF. But the details matter. This post walks through the full derivation---from the continuous Newton-Euler equations, to discrete backward Euler as a nonlinear system, to the Schur complement solve you actually run each iteration---with reference code from [Newton](https://github.com/newton-physics/newton), which implements this approach under the name AVBD (Augmented VBD).

This is Section I: free (unconstrained) rigid bodies. Section II will cover articulated bodies with joints.

> **Prerequisite:** This post assumes familiarity with quaternion math for rigid body rotation---in particular the rotation-vector exponential map, quaternion multiplication, and how angular velocity integrates orientation. If you're rusty on any of this, I recommend reading [Quaternion Math for Rigid Body Simulation]({% post_url 2026-04-30-quaternion-primer %}) first.

---

## Continuous Rigid Body Dynamics

A rigid body has two coupled equations of motion. For the **translational** DoF:

$$m \ddot{\mathbf{x}}_\text{com} = \mathbf{f}$$

where $m$ is the total mass, $\mathbf{x}\_\text{com}$ is the world-space center-of-mass position, and $\mathbf{f}$ includes gravity, contact forces, and applied forces.

For the **rotational** DoF, the Newton-Euler equation in the **body frame** (where inertia is constant) is:

$$\mathbf{I}_\text{body}\,\dot{\boldsymbol{\omega}} + \boldsymbol{\omega} \times (\mathbf{I}_\text{body}\,\boldsymbol{\omega}) = \boldsymbol{\tau}$$

where $\boldsymbol{\omega}$ is the angular velocity in the body frame, $\boldsymbol{\tau}$ is the torque mapped to the body frame, and $\mathbf{I}\_\text{body}$ is the constant body-frame inertia tensor. In the world frame this is equivalently $\mathbf{I}\_\text{world}\,\dot{\boldsymbol{\omega}}\_\text{world} = \boldsymbol{\tau}\_\text{world} - \boldsymbol{\omega}\_\text{world} \times \mathbf{I}\_\text{world}\,\boldsymbol{\omega}\_\text{world}$ with $\mathbf{I}\_\text{world} = \mathbf{R}\,\mathbf{I}\_\text{body}\,\mathbf{R}^T$.

The full state at step $n$ is: position $\mathbf{x}^n \in \mathbb{R}^3$, orientation $\mathbf{R}^n \in SO(3)$, linear velocity $\mathbf{v}^n$, body-frame angular velocity $\boldsymbol{\omega}^n$, mass $m$, and body inertia $\mathbf{I}\_\text{body}$.

---

## Discretizing with Backward Euler

### Step 1: Pose Increments as DoFs

Rather than solving for $\mathbf{x}^{n+1}$ and $\mathbf{R}^{n+1}$ directly, we introduce **pose increments** as the unknowns:

$$\Delta\mathbf{x} \in \mathbb{R}^3, \qquad \Delta\boldsymbol{\theta} \in \mathbb{R}^3 \;\text{(rotation vector)}$$

The new pose is then:

$$\mathbf{x}^{n+1} = \mathbf{x}^n + \Delta\mathbf{x}$$

$$\mathbf{R}^{n+1} = \exp(\widehat{\Delta\boldsymbol{\theta}})\,\mathbf{R}^n$$

where $\widehat{\Delta\boldsymbol{\theta}}$ is the skew-symmetric matrix of $\Delta\boldsymbol{\theta}$. This is the standard left-perturbation on $SO(3)$. We will find $\Delta\mathbf{x}$ and $\Delta\boldsymbol{\theta}$ by enforcing implicit Euler as a 6-equation residual system.

### Step 2: Translational Residual

Start from the standard backward Euler update:

$$m\,\frac{\mathbf{v}^{n+1} - \mathbf{v}^n}{h} = \mathbf{f}(\mathbf{x}^{n+1}, \mathbf{R}^{n+1})$$

Use the kinematic relation $\mathbf{x}^{n+1} = \mathbf{x}^n + h\,\mathbf{v}^{n+1}$ to eliminate $\mathbf{v}^{n+1} = \Delta\mathbf{x}/h$:

$$m\,\frac{\Delta\mathbf{x}/h - \mathbf{v}^n}{h} = \mathbf{f}(\mathbf{x}^n + \Delta\mathbf{x},\; \mathbf{R}^{n+1}(\Delta\boldsymbol{\theta}))$$

Rearranging to residual form:

$$\mathbf{r}_\text{lin}(\Delta\mathbf{x}, \Delta\boldsymbol{\theta}) \;=\; \frac{m}{h^2}\!\left(\Delta\mathbf{x} - h\mathbf{v}^n\right) - \mathbf{f}(\mathbf{x}^n + \Delta\mathbf{x},\; \mathbf{R}^{n+1}) \;=\; \mathbf{0}$$

### Step 3: Rotational Residual (Body Frame)

Recall Newton-Euler equation:

$$\mathbf{I}_\text{body}\,\dot{\boldsymbol{\omega}} + \boldsymbol{\omega} \times (\mathbf{I}_\text{body}\,\boldsymbol{\omega}) = \boldsymbol{\tau}$$

Convert to the stand ODE form of $\dot{\boldsymbol{\omega}}=f(\boldsymbol{\omega}, t)$, we have:
$$
\dot{\boldsymbol{\omega}}= I^{-1}_\text{body}(\boldsymbol{\tau} -  \boldsymbol{\omega} \times (\mathbf{I}_\text{body}\,\boldsymbol{\omega}))
$$

Work in the body frame where $\mathbf{I}\_\text{body}$ is constant. Backward Euler on the angular velocity gives:

$$\mathbf{I}_\text{body}\,\frac{\boldsymbol{\omega}^{n+1} - \boldsymbol{\omega}^n}{h} + \boldsymbol{\omega}^{n+1} \times (\mathbf{I}_\text{body}\,\boldsymbol{\omega}^{n+1}) = \boldsymbol{\tau}^{n+1}$$

The rotation increment $\Delta\boldsymbol{\theta}$ integrates to $\mathbf{R}^{n+1}$, so we identify:

$$\boldsymbol{\omega}^{n+1} \approx \frac{\Delta\boldsymbol{\theta}}{h}$$

(constant angular velocity over the step whose integrated angle equals $\Delta\boldsymbol{\theta}$). Substituting and multiplying through by $h$:

$$\mathbf{I}_\text{body}\!\left(\frac{\Delta\boldsymbol{\theta}}{h^2} - \frac{\boldsymbol{\omega}^n}{h}\right) + \frac{1}{h^2}\,\Delta\boldsymbol{\theta} \times (\mathbf{I}_\text{body}\,\Delta\boldsymbol{\theta}) = \boldsymbol{\tau}^{n+1}$$

Rearranging to residual form:

$$\mathbf{r}_\text{rot}(\Delta\mathbf{x}, \Delta\boldsymbol{\theta}) \;=\; \mathbf{I}_\text{body}\!\left(\frac{\Delta\boldsymbol{\theta}}{h^2} - \frac{\boldsymbol{\omega}^n}{h}\right) + \frac{\Delta\boldsymbol{\theta} \times (\mathbf{I}_\text{body}\,\Delta\boldsymbol{\theta})}{h^2} - \boldsymbol{\tau}^{n+1}(\Delta\mathbf{x}, \Delta\boldsymbol{\theta}) \;=\; \mathbf{0}$$

$\Delta\boldsymbol{\theta} \times (\mathbf{I}\_\text{body}\,\Delta\boldsymbol{\theta})/h^2$ is called the gyroscopic term. It is a quadratic force term. 

### Step 4: Combined Nonlinear System

Stack both residuals into a single 6-equation system:

$$F(\Delta\mathbf{x},\,\Delta\boldsymbol{\theta}) = \begin{bmatrix} \mathbf{r}_\text{lin}(\Delta\mathbf{x}, \Delta\boldsymbol{\theta}) \\\\ \mathbf{r}_\text{rot}(\Delta\mathbf{x}, \Delta\boldsymbol{\theta}) \end{bmatrix} = \mathbf{0}$$

This is solved with Newton's method. Initialize $\Delta\mathbf{x} = h\mathbf{v}^n$, $\Delta\boldsymbol{\theta} = h\boldsymbol{\omega}^n$ (explicit Euler guess), then iterate:

1. Evaluate residual $F$
2. Build Jacobian $\mathbf{J} = \partial F / \partial (\Delta\mathbf{x},\, \Delta\boldsymbol{\theta})$
3. Solve $\mathbf{J}\,\delta = -F$
4. Update $\Delta\mathbf{x} \mathrel{+}= \delta\_x$, $\Delta\boldsymbol{\theta} \mathrel{+}= \delta\_\theta$

Once converged, recover the new state and velocities:

$$\mathbf{x}^{n+1} = \mathbf{x}^n + \Delta\mathbf{x}, \qquad \mathbf{R}^{n+1} = \exp(\widehat{\Delta\boldsymbol{\theta}})\,\mathbf{R}^n$$

$$\mathbf{v}^{n+1} = \frac{\Delta\mathbf{x}}{h}, \qquad \boldsymbol{\omega}^{n+1} = \frac{\Delta\boldsymbol{\theta}}{h}$$

---

## From Residual to the 6&times;6 Newton System

Rather than solving the full implicit-Euler residual---which includes the nonlinear gyroscopic term $\boldsymbol{\omega}\times\mathbf{I}\_\text{body}\boldsymbol{\omega}$ and requires the Newton-Euler equation to stay in the body frame---we split the problem into explicit and implicit parts:

- **Explicit:** free-body dynamics (inertia, gravity, gyroscopic torque) are forward-integrated once into inertial targets $\mathbf{x}^{\ast}$ and $\mathbf{R}^{\ast}$, then frozen for the rest of the step.
- **Implicit:** contact and constraint forces are resolved iteratively through VBD's Gauss-Seidel sweeps.

This is a compromise from fully-implicit backward Euler for the rigid-body dynamics. In exchange, it buys three things: the gyroscopic nonlinearity is absorbed into $\mathbf{R}^{\ast}$ rather than carried in the residual, the angular Hessian stays symmetric positive-definite, and the entire Newton system can be assembled and solved in world frame (since the body-frame gyroscopic term---the reason Newton-Euler is traditionally formulated in the body frame---is no longer present in the iterative solve).

With this split, the rotational residual reduces from

$$\mathbf{r}_\text{rot} = \mathbf{I}_\text{body}\!\left(\frac{\Delta\boldsymbol{\theta}}{h^2} - \frac{\boldsymbol{\omega}^n}{h}\right) + \frac{\Delta\boldsymbol{\theta} \times (\mathbf{I}_\text{body}\,\Delta\boldsymbol{\theta})}{h^2} - \boldsymbol{\tau}^{n+1}$$

to a simple **spring pulling toward the explicit target**:

$$\mathbf{r}_\text{rot} = \frac{1}{h^2}\,\mathbf{I}_\text{world}\,(\Delta\boldsymbol{\theta} - h\boldsymbol{\omega}^{\ast}) - \boldsymbol{\tau}_\text{constraint}$$

where $\boldsymbol{\omega}^{\ast}$ is the gyro-corrected angular velocity from the forward step and $\Delta\boldsymbol{\theta} - h\boldsymbol{\omega}^{\ast}$ is just $-\boldsymbol{\theta}$, the rotation vector from $\mathbf{R}\_\text{cur}$ to $\mathbf{R}^{\ast}$. This has the same structure as the translational residual $\tfrac{m}{h^2}(\mathbf{x}\_\text{com} - \mathbf{x}^{\ast}\_\text{com}) - \mathbf{f}\_\text{constraint}$: a quadratic spring to an explicit inertial target, plus implicit constraint forces. The 6&times;6 system then has a natural 2&times;2 block structure, all in world frame:

$$\begin{bmatrix} H_{ll} & H_{al}^T \\\\ H_{al} & H_{aa} \end{bmatrix} \begin{bmatrix} \Delta\mathbf{x} \\\\ \Delta\boldsymbol{\omega} \end{bmatrix} = \begin{bmatrix} \mathbf{f}_{lin} \\\\ \mathbf{f}_{ang} \end{bmatrix}$$

where $\Delta\mathbf{x}$ and $\Delta\boldsymbol{\omega}$ are the Newton step corrections and the right-hand side is $-\mathbf{r}$ at the current iterate. In VBD we run this as a **single Newton step per body per VBD iteration**, giving us a fast inner solve with guaranteed descent.

### Inertial Blocks

The inertial blocks are simple springs to the forward-integrated targets:

$$H_{ll}^\text{inertia} = \frac{m}{h^2}\mathbf{I}_3, \qquad \mathbf{f}_{lin}^\text{inertia} = \frac{m}{h^2}(\mathbf{x}^{\ast}_\text{com} - \mathbf{x}_\text{com})$$

$$H_{aa}^\text{inertia} = \frac{1}{h^2}\mathbf{I}_\text{world}, \qquad \mathbf{f}_{ang}^\text{inertia} = \frac{1}{h^2}\mathbf{I}_\text{world}\,\boldsymbol{\theta}$$

$$H_{al}^\text{inertia} = \mathbf{0}$$

Here $\mathbf{x}^{\ast}\_\text{com} = \mathbf{x}^n\_\text{com} + h\mathbf{v}^n + h^2 m^{-1}\mathbf{f}\_\text{ext}$ is the **translational inertial target**, $\boldsymbol{\theta}$ is the rotation vector from the current orientation to $\mathbf{R}^{\ast}$, and $\mathbf{I}\_\text{world} = \mathbf{R}\,\mathbf{I}\_\text{body}\,\mathbf{R}^T$. The linear and angular inertial blocks have identical structure: a mass-weighted pull toward an explicit prediction, with the constraint solve handling everything else.

At convergence the angular residual gives $\mathbf{I}\_\text{world}(\boldsymbol{\omega}^{n+1} - \boldsymbol{\omega}^{\ast})/h = \boldsymbol{\tau}\_\text{constraint}$, i.e. the only thing that changes $\boldsymbol{\omega}$ from the gyro-corrected prediction is the implicit constraint response.

#### Computing $\mathbf{R}^{\ast}$: baking the gyroscopic term into the target

The angular target $\mathbf{R}^{\ast}$ is produced by one semi-implicit Newton-Euler step that includes the gyroscopic torque. Inside `integrate_rigid_body` the body-frame torque used to step angular velocity is

```python
# body-frame angular velocity and torque, with gyroscopic correction
wb = wp.quat_rotate_inv(r0, w0)
tb = wp.quat_rotate_inv(r0, t0) - wp.cross(wb, inertia * wb)   # subtract ω × Iω
w1 = wp.quat_rotate(r0, wb + inv_inertia * tb * dt)            # semi-implicit ω*
r1 = wp.normalize(r0 + wp.quat(w1, 0.0) * r0 * 0.5 * dt)        # → R*
```

Line by line, with $\mathbf{R}^n$ the current orientation, $\boldsymbol{\omega}^n$ the world-frame angular velocity, and $\boldsymbol{\tau}^n$ the world-frame torque:

**Rotate $\boldsymbol{\omega}^n$ into the body frame:**

$$\boldsymbol{\omega}_b \;=\; (\mathbf{R}^n)^{T}\,\boldsymbol{\omega}^n$$

**Rotate the torque into the body frame and subtract the gyroscopic term** (Newton-Euler RHS, $\mathbf{I}\_\text{body}\dot{\boldsymbol{\omega}}\_b = \boldsymbol{\tau}\_b - \boldsymbol{\omega}\_b\times\mathbf{I}\_\text{body}\boldsymbol{\omega}\_b$):

$$\boldsymbol{\tau}_b^\text{eff} \;=\; (\mathbf{R}^n)^{T}\,\boldsymbol{\tau}^n \;-\; \boldsymbol{\omega}_b \times (\mathbf{I}_\text{body}\,\boldsymbol{\omega}_b)$$

**Semi-implicit Euler step on body-frame $\boldsymbol{\omega}$, then rotate back to world:**

$$\boldsymbol{\omega}^{\ast} \;=\; \mathbf{R}^n\!\left(\boldsymbol{\omega}_b + h\,\mathbf{I}_\text{body}^{-1}\,\boldsymbol{\tau}_b^\text{eff}\right)$$

Compactly:

$$\boxed{\;\boldsymbol{\omega}^{\ast} = \mathbf{R}^n\!\left[\boldsymbol{\omega}_b + h\,\mathbf{I}_\text{body}^{-1}\!\big((\mathbf{R}^n)^{T}\boldsymbol{\tau}^n - \boldsymbol{\omega}_b\times\mathbf{I}_\text{body}\boldsymbol{\omega}_b\big)\right], \qquad \mathbf{R}^{\ast} = \exp\!\big(h\,[\boldsymbol{\omega}^{\ast}]_\times\big)\,\mathbf{R}^n\;}$$

Because $\boldsymbol{\omega}^{\ast}$ includes the gyroscopic correction $-\mathbf{I}\_\text{body}^{-1}(\boldsymbol{\omega}^n \times \mathbf{I}\_\text{body}\boldsymbol{\omega}^n)\,h$, the free-body residual vanishes to leading order at $\mathbf{R}^{\ast}$. The simple inertial spring $\mathbf{f}\_\text{ang}^\text{inertia} = h^{-2}\mathbf{I}\_\text{world}\,\boldsymbol{\theta}$ therefore agrees with the full nonlinear residual at the initial iterate $\mathbf{R}\_\text{cur} = \mathbf{R}^{\ast}$, with the gyroscopic torque's value rerouted through $\mathbf{R}^{\ast}$ rather than evaluated directly each iteration.

#### What this approximation drops

The full rotational residual re-centered at $\mathbf{R}^{\ast}$ (writing $\boldsymbol{\delta}$ for the rotation vector from $\mathbf{R}^{\ast}$ to $\mathbf{R}\_\text{cur}$, i.e. how far constraints have pushed the iterate off the target) is

$$\mathbf{r}_\text{rot} = \underbrace{\frac{1}{h^2}\,\mathbf{I}_\text{body}\,\boldsymbol{\delta}}_{\text{kept: inertial spring}} \;+\; \underbrace{\frac{\boldsymbol{\omega}\times\mathbf{I}_\text{body}\boldsymbol{\delta} + \boldsymbol{\delta}\times\mathbf{I}_\text{body}\boldsymbol{\omega}}{h}}_{\text{dropped: gyro coupling}} \;+\; \underbrace{\frac{\boldsymbol{\delta}\times\mathbf{I}_\text{body}\boldsymbol{\delta}}{h^2}}_{\text{dropped: quadratic gyro}} \;-\; \boldsymbol{\tau}_\text{constraint}$$

The kept spring scales like $1/h^2$ in $\boldsymbol{\delta}$, the gyro coupling like $\|\boldsymbol{\omega}\|/h$, and the quadratic gyro like $\|\boldsymbol{\delta}\|/h^2$. The ratio of the gyro coupling to the inertial spring is $O(\|\boldsymbol{\omega}\|h)$---small for typical simulation timesteps. Three practical reasons AVBD drops these terms: the gyro coupling's Jacobian $[\boldsymbol{\omega}]\_\times\mathbf{I} - [\mathbf{I}\boldsymbol{\omega}]\_\times$ is **not symmetric**, which would break the Cholesky factorization used in the Schur complement solve; keeping the gyroscopic term in the residual would require working in the body frame (where $\mathbf{I}\_\text{body}$ is constant), giving up the world-frame formulation that contacts and joints naturally live in; and the gyroscopic term evaluated at $\boldsymbol{\omega}^n$ vs. $\boldsymbol{\omega}^{n+1}$ differs by $O(h)$ in torque units, the same order as backward Euler's intrinsic discretization error, so refining it further would not improve the integrator's accuracy.

In Newton, `forward_step_rigid_bodies` computes $\mathbf{x}^{\ast}$ and $\mathbf{R}^{\ast}$ by semi-implicit integration, storing them as `body_inertia_q`:

```python
# forward_step_rigid_bodies (simplified)
q_new, qd_new = integrate_rigid_body(
    q_current, qd_current, f_ext,
    com_local, I_body, inv_m, inv_I, gravity, dt
)
body_inertia_q[tid] = q_new   # frozen inertial target q*
body_q[tid]         = q_new   # initial guess for iterations
```

### Contact and Constraint Blocks

Any force element (contact, joint) acting at contact point $\mathbf{p}$ with moment arm $\mathbf{r} = \mathbf{p} - \mathbf{x}\_\text{com}$ contributes:

$$H_{ll}^c = \mathbf{K}_c, \qquad H_{al}^c = -[\mathbf{r}]_\times^T \mathbf{K}_c, \qquad H_{aa}^c = [\mathbf{r}]_\times^T \mathbf{K}_c\,[\mathbf{r}]_\times$$

where $\mathbf{K}\_c = \partial \mathbf{f}\_c / \partial \mathbf{x}$ is the contact stiffness and $[\mathbf{r}]\_\times$ is the skew-symmetric cross-product matrix. All blocks are summed over adjacent force elements before the solve.

---

## Assembling the 6&times;6 System in Code

### Contact Force and Hessian (`evaluate_rigid_contact_from_collision`)

For each contact between body $A$ and body $B$, the contact model in `evaluate_rigid_contact_from_collision` computes the full wrench and Hessian blocks for both bodies. The normal force and stiffness come from the penalty model:

```python
# Normal force and stiffness
n_outer  = wp.outer(contact_normal, contact_normal)
f_total  = contact_normal * (contact_ke * penetration_depth)
K_total  = contact_ke * n_outer
```

Damping is added when the contact is closing ($\mathbf{v}\_\text{rel} \cdot \hat{\mathbf{n}} < 0$):

```python
# Relative velocity via finite difference of contact points
dx_rel = (x_c_b_now - x_c_b_prev) - (x_c_a_now - x_c_a_prev)
v_rel  = dx_rel / dt
v_dot_n = wp.dot(contact_normal, v_rel)

if contact_kd > 0.0 and v_dot_n < 0.0:
    damping_coeff    = contact_kd * contact_ke
    f_total         += -damping_coeff * v_dot_n * contact_normal
    K_total         += (damping_coeff / dt) * n_outer
```

Then for each body the moment arm $\mathbf{r} = \mathbf{p}\_\text{contact} - \mathbf{x}\_\text{com}$ is used to build all three Hessian blocks:

```python
# Body B side (body A is symmetric with opposite sign on force)
force_b  =  f_total
torque_b = wp.cross(r_b, force_b)

r_b_skew       = wp.skew(r_b)               # [r]_x
r_b_skew_T_K   = wp.transpose(r_b_skew) * K_total

h_ll_b = K_total                             # ∂f/∂x
h_al_b = -r_b_skew_T_K                      # ∂τ/∂x  =  -[r]_x^T K
h_aa_b =  r_b_skew_T_K * r_b_skew           # ∂τ/∂ω  =  [r]_x^T K [r]_x
```

### Per-Body Accumulation (`accumulate_body_body_contacts_per_body`)

Rather than iterating over all contacts globally, the solver builds a **per-body contact list** once per step (a CSR-style buffer). During each Gauss-Seidel color sweep, each body iterates only over its own contacts using 16 strided threads, accumulating into local registers before a single atomic write:

```python
# Each body_id iterates its own contact list (strided over 16 threads)
force_acc = vec3(0);  torque_acc = vec3(0)
h_ll_acc  = mat33(0); h_al_acc  = mat33(0); h_aa_acc = mat33(0)

i = thread_id_within_body               # 0..15
while i < num_contacts_for_body:
    contact_idx = body_contact_indices[body_id * buffer_size + i]

    # Compute contact world points and penetration depth
    cp0_world = transform_point(body_q[b0], cp0_local)
    cp1_world = transform_point(body_q[b1], cp1_local)
    penetration = thickness - dot(contact_normal, cp1_world - cp0_world)

    if penetration > eps:
        force_0, torque_0, h_ll_0, h_al_0, h_aa_0,
        force_1, torque_1, h_ll_1, h_al_1, h_aa_1 = \
            evaluate_rigid_contact_from_collision(b0, b1, ...)

        # Pick the side that belongs to this body
        if body_id == b0:
            force_acc += force_0;  torque_acc += torque_0
            h_ll_acc  += h_ll_0;   h_al_acc  += h_al_0;  h_aa_acc += h_aa_0
        else:
            force_acc += force_1;  torque_acc += torque_1
            h_ll_acc  += h_ll_1;   h_al_acc  += h_al_1;  h_aa_acc += h_aa_1

    i += 16   # stride

# One atomic add per body at the end
atomic_add(body_forces,      body_id, force_acc)
atomic_add(body_torques,     body_id, torque_acc)
atomic_add(body_hessian_ll,  body_id, h_ll_acc)
atomic_add(body_hessian_al,  body_id, h_al_acc)
atomic_add(body_hessian_aa,  body_id, h_aa_acc)
```

### Final Assembly and Solve (`solve_rigid_body`)

After all contacts (and joints, via `evaluate_joint_force_hessian`) have been accumulated into `body_forces/torques/hessians`, `solve_rigid_body` reads those external contributions and adds the **inertial blocks** to form the complete system:

```python
# ── Inertial contributions ────────────────────────────────────────
inertial_coeff = m * dt_sqr_reciprocal          # m/h²

# Linear inertial force: pull COM toward inertial target
f_lin = (com_star - com_current) * inertial_coeff

# Angular inertial torque: pull orientation toward target
q_delta   = quat_inverse(rot_current) * rot_star
theta_body = axis_angle_to_vec(q_delta)         # rotation vector in body frame
tau_body   = I_body * (theta_body * dt_sqr_reciprocal)
tau_world  = quat_rotate(rot_current, tau_body)

# Angular Hessian in world frame
R_cur      = quat_to_matrix(rot_current)
I_world    = R_cur * I_body * R_cur.T
angular_hessian = dt_sqr_reciprocal * I_world

# ── Add external (contact + joint) contributions ──────────────────
f_force  = f_lin   + external_forces[body_id]
f_torque = tau_world + external_torques[body_id]

h_ll = diag(inertial_coeff) + external_hessian_ll[body_id]
h_al =                         external_hessian_al[body_id]
h_aa = angular_hessian       + external_hessian_aa[body_id]

# ── Joint contributions (CSR adjacency loop) ──────────────────────
for j in adjacent_joints(body_id):
    jf, jt, jH_ll, jH_al, jH_aa = evaluate_joint_force_hessian(body_id, j, ...)
    f_force  += jf;   f_torque += jt
    h_ll += jH_ll;    h_al += jH_al;   h_aa += jH_aa

# ── Schur complement solve (see next section) ─────────────────────
dw, dx = schur_solve(h_ll, h_al, h_aa, f_force, f_torque)
```

The key design point: contacts write into a shared `body_forces/hessians` buffer with atomic adds (one write per body per color), while joints are accumulated inline inside `solve_rigid_body` via a private loop. Both feed into the same 6&times;6 solve.

---

## Solving via Schur Complement

We reduce the 6&times;6 system to two successive 3&times;3 solves. Eliminate $\Delta\mathbf{x}$ from the top block row:

$$\Delta\mathbf{x} = H_{ll}^{-1}(\mathbf{f}_{lin} - H_{al}^T \Delta\boldsymbol{\omega})$$

Substitute into the bottom row:

$$\underbrace{(H_{aa} - H_{al}\,H_{ll}^{-1}\,H_{al}^T)}_{\mathbf{S}}\,\Delta\boldsymbol{\omega} = \mathbf{f}_{ang} - H_{al}\,H_{ll}^{-1}\,\mathbf{f}_{lin}$$

Factorize and solve in order:

**Step 1.** $\mathbf{L}\_M\mathbf{L}\_M^T = H\_{ll}$ &nbsp;&nbsp;(Cholesky)

**Step 2.** $\mathbf{S} = H\_{aa} - H\_{al}\,H\_{ll}^{-1}\,H\_{al}^T$ &nbsp;&nbsp;(Schur complement)

**Step 3.** $\mathbf{S}\,\Delta\boldsymbol{\omega} = \mathbf{f}\_{ang} - H\_{al}\,H\_{ll}^{-1}\,\mathbf{f}\_{lin}$ &nbsp;&nbsp;(solve for angular)

**Step 4.** $H\_{ll}\,\Delta\mathbf{x} = \mathbf{f}\_{lin} - H\_{al}^T\,\Delta\boldsymbol{\omega}$ &nbsp;&nbsp;(back-substitute for linear)

Both 3&times;3 solves use a packed Cholesky (6-float lower-triangular factor). $H\_{aa}$ is lightly regularized first:

$$H_{aa} \leftarrow H_{aa} + \varepsilon\mathbf{I}, \qquad \varepsilon = 10^{-9}\!\left(\tfrac{\mathrm{tr}(H_{aa})}{3} + 1\right)$$

---

## Pose Update and Velocity Recovery

Apply the Newton increments to the current pose:

$$\mathbf{x}_\text{com}^\text{new} = \mathbf{x}_\text{com} + \Delta\mathbf{x}$$

$$\mathbf{r}^\text{new} = \delta\mathbf{r} \otimes \mathbf{r}, \qquad \delta\mathbf{r} = \text{quat\_from\_axis\_angle}\!\left(\tfrac{\Delta\boldsymbol{\omega}}{|\Delta\boldsymbol{\omega}|},\; |\Delta\boldsymbol{\omega}|\right)$$

For small $\Delta\boldsymbol{\omega}$ the first-order approximation $\delta\mathbf{r} \approx \text{normalize}(\tfrac{1}{2}\Delta\boldsymbol{\omega},\, 1)$ is used for efficiency (controlled by `_USE_SMALL_ANGLE_APPROX` in Newton).

After all VBD iterations finish, velocities are recovered by finite difference (BDF1):

$$\mathbf{v}^{n+1} = \frac{\mathbf{x}_\text{com}^{n+1} - \mathbf{x}_\text{com}^n}{h}, \qquad \boldsymbol{\omega}^{n+1} = \frac{\log(\mathbf{r}^n{}^{-1} \otimes \mathbf{r}^{n+1})}{h}$$

---

## AVBD: Adaptive Penalty for Constraints and Contacts

For particles, force elements are elastic energies with analytic Hessians. For rigid bodies, the dominant force elements are **contacts** (non-penetration) and **joints** (relative pose targets). Both enter the same Newton system as soft penalty forces with **adaptive stiffness**---this is the "Augmented" in AVBD.

A contact with penetration depth $d > 0$ contributes $E\_c = \tfrac{1}{2}k\_c d^2$, giving $\mathbf{f}\_c = k\_c d\,\hat{\mathbf{n}}$ and stiffness $k\_c$. Rather than a fixed $k\_c$, the penalty grows each iteration to push the violation toward zero:

$$k\_c \leftarrow \min\!\left(k\_c + \beta\,|C|,\; k\_\text{max}\right)$$

where $C$ is the constraint violation, $\beta$ is a ramp rate, and $k\_\text{max}$ is the material stiffness cap. At the start of each timestep, $k\_c$ is warmstarted from the previous step with a small decay:

$$k\_c \leftarrow \gamma\,k\_c, \qquad k\_c \in [k\_\text{min},\; k\_\text{max}]$$

with $\gamma \approx 0.99$. This carries stiffness information across frames without indefinite growth.

---

## The Complete Per-Step Algorithm

```
# ── Initialization ────────────────────────────────────────────────
for each body b:
    q_star[b] = forward_integrate(q[b], qd[b], f_ext, dt)
    q[b]          = q_star[b]    # initial guess = inertial target
    body_inertia_q[b] = q_star[b]

warmstart_penalties(gamma)       # k <- clamp(gamma*k, k_min, k_max)
build_contact_lists(contacts)    # per-body CSR adjacency

# ── VBD Iterations ────────────────────────────────────────────────
for iter in range(N_iterations):
    for color in body_color_groups:      # Gauss-Seidel by coloring
        zero(body_forces, body_torques, body_hessians)

        for each contact adjacent to bodies in color:
            f, tau, H_ll, H_al, H_aa = contact_force_hessian(...)
            body_forces[b]  += f
            body_torques[b] += tau
            body_hessians[b] += (H_ll, H_al, H_aa)

        for each body b in color:
            # Inertial contributions (from r_lin, r_rot)
            f_lin = (m/h^2) * (x_com_star - x_com)
            theta  = log(r^-1 * r_star)     # rotation vector to target
            f_ang = (I_world/h^2) * theta
            H_ll  = (m/h^2)*I3 + H_ll_contacts
            H_al  =                H_al_contacts
            H_aa  = I_world/h^2  + H_aa_contacts

            for each joint adjacent to b:
                f_lin, f_ang, H_ll, H_al, H_aa += joint_force_hessian(b, j)

            # Schur complement solve
            L_M = chol(H_ll)
            S   = H_aa - H_al @ inv(L_M) @ H_al.T
            dw  = solve(chol(S), f_ang - H_al @ solve(L_M, f_lin))
            dx  = solve(L_M, f_lin - H_al.T @ dw)

            x_com += dx
            r = normalize(quat(dw) * r)

    # Dual update after each sweep
    for each contact c:
        k_c = min(k_c + beta * |penetration_c|, k_max_c)
    for each joint j:
        k_j = min(k_j + beta * |C_j|,           k_max_j)

# ── Finalization ──────────────────────────────────────────────────
for each body b:
    v[b]     = (x_com[b] - x_com_prev[b]) / dt
    omega[b] = quat_velocity(r[b], r_prev[b], dt)
    body_q_prev[b] = body_q[b]
```

---

## Reference Code

The implementation lives in [Newton's VBD solver](https://github.com/newton-physics/newton/blob/main/newton/_src/solvers/vbd/). Key files:

- `rigid_vbd_kernels.py` --- GPU kernels: `forward_step_rigid_bodies`, `solve_rigid_body`, `update_duals_body_body_contacts`, `update_duals_joint`, `update_body_velocity`
- `solver_vbd.py` --- orchestration in `SolverVBD.step()` and `_solve_rigid_body_iteration()`

The Schur complement solve from `solve_rigid_body`:

```python
# Regularize H_aa
trA = wp.trace(h_aa) / 3.0
eps = 1e-9 * (trA + 1.0)
h_aa[0,0] += eps;  h_aa[1,1] += eps;  h_aa[2,2] += eps

# Factorize H_ll
Lm = chol33(h_ll)

# Compute H_ll^{-1} * H_al^T, column by column
X0 = chol33_solve(Lm, h_al[0])
X1 = chol33_solve(Lm, h_al[1])
X2 = chol33_solve(Lm, h_al[2])
MinvCt = mat33_from_columns(X0, X1, X2)

# Schur complement and solve
S     = h_aa - h_al @ MinvCt
Ls    = chol33(S)
rhs_w = f_ang - h_al @ chol33_solve(Lm, f_lin)
dw    = chol33_solve(Ls, rhs_w)           # angular increment
dx    = chol33_solve(Lm, f_lin - wp.transpose(h_al) @ dw)  # linear

# Apply (small-angle approximation)
half_w = dw * 0.5
dq     = wp.normalize(wp.quat(half_w[0], half_w[1], half_w[2], 1.0))
r_new  = wp.normalize(dq * r_current)
x_com_new = x_com + dx
```

AVBD dual updates after each color sweep:

```python
# Contact penalty (update_duals_body_body_contacts)
penetration = max(0.0, thickness - dot(n, p1_world - p0_world))
k[contact]  = min(k[contact] + beta * penetration, k_max[contact])

# Joint penalty (update_duals_joint), e.g. BALL joint
C_lin    = length(x_child_frame - x_parent_frame)
k[joint] = min(k[joint] + beta * C_lin, k_max[joint])
```

---

## What's Next

This covers the full pipeline for free rigid bodies: continuous Newton-Euler dynamics, the pose-increment formulation of backward Euler, the resulting 6&times;6 Newton system and its Schur complement solve, and the AVBD adaptive penalty mechanism for contacts. Each body is updated as an independent local solve within its color group, matching the exact VBD pattern from the particle solver---just 6 DoF instead of 3.

Section II will cover **articulated bodies**: joint constraints, the rotation-vector curvature error for cable/fixed joints, and how the adjacency graph coloring extends to joint chains.

---

*Newton: [github.com/newton-physics/newton](https://github.com/newton-physics/newton)*

*VBD paper: Anka He Chen, Ziheng Liu, Yin Yang, Cem Yuksel. "Vertex Block Descent." ACM Trans. Graph. 43, 4, Article 116 (2024). [doi:10.1145/3658179](https://doi.org/10.1145/3658179)*

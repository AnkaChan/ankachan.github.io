---
permalink: /avbd-franka-rigid-robot-tuning-guide/
title: 'AVBD Franka Simulation Notes'
tags:
  - physics-simulation
  - VBD
  - rigid-body
  - robotics
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
---

## Rigid & Robot Tuning Guide

For this project, the key changes that made the AVBD Franka simulation work were:

1. **Fixed AVBD drive-vs-limit behavior**

   The main Newton-side fix was Donglai's AVBD `resolve_drive_limit_mode()` change.

   Before, if a joint was slightly outside or numerically on a limit, AVBD could switch to `LIMIT_UPPER` / `LIMIT_LOWER` and suppress the PD drive. For Franka, that meant the commanded target could not pull the joint back from the limit, so the arm looked stuck or failed to track.

   The fix is: if the joint is outside a limit but the drive target is back inside the valid range, use `DRIVE`, not `LIMIT`.

   Conceptually:

   ```python
   if q > lim_upper:
       if has_drive and drive_target < lim_upper:
           mode = DRIVE
       else:
           mode = LIMIT_UPPER
   ```

   Same for the lower limit.

2. **Used one unified VBD solve**

   The Franka, rigid contents, and deformable bag are all integrated by the same `SolverVBD`, so contact is two-way coupled instead of having the robot treated as an external or kinematic driver.

3. **Increased AVBD structural joint constraint stiffness**

   Once Franka was running in AVBD, the key parameter change was not more PD stiffness. It was making the AVBD structural joint constraints stiff enough:

   ```python
   rigid_joint_linear_ke = 1.0e6
   rigid_joint_angular_ke = 1.0e6
   rigid_joint_linear_kd = 1.0e3
   rigid_joint_angular_kd = 1.0e2
   ```

   These make the articulated rigid bodies stay coherent under contact load. PD gains only control target tracking; they do not make the AVBD joint constraints themselves rigid.

4. **Adjusted the PD control setup**

   I kept the arm PD gains high enough for tracking, but did not solve the softness by simply increasing PD stiffness. The working setup uses strong but stable joint drives:

   ```python
   arm_drive_ke = (1.0e6, 1.0e6, 8.0e5, 8.0e5, 6.0e5, 6.0e5, 6.0e5)
   arm_drive_kd = (1.0e5, 1.0e5, 8.0e4, 8.0e4, 6.0e4, 6.0e4, 6.0e4)
   gripper_drive_ke = 1.0e6
   gripper_drive_kd = 1.0e5
   ```

   The PD drives make the robot follow IK targets; the `rigid_joint_*` params make the robot physically rigid enough under contact.

5. **Made the gripper contact strong enough to pinch the bag**

   The stable pinch came from high-friction, dense finger pads and a small closed gap:

   ```python
   soft_contact_mu = 1.0
   rigid_contact_hard = True
   finger_pad_density = 1000.0
   finger_shape_mu = 1.0
   finger_shape_ke = 1.0e6
   gripper_closed_gap = 0.001
   ```

6. **Used the provided bag asset with self-contact off**

   For the OBJ-bag Franka example, the bag asset is loaded from the provided USDA mesh. Bag self-contact is disabled because it did not materially improve the result while adding cost.

7. **Kept rigid body contact stiffness moderate**

   For the rigid contents, overly stiff material/contact `ke` and `kd` can make the bodies unstable. If rigid bodies jitter or explode, reduce their `ke` and `kd` first rather than increasing solver stiffness everywhere.

8. **Tune contact distance per rigid body with separate margins**

   Different rigid contents may need different contact distances. Use separate margins per rigid body or shape so each object can be tuned independently instead of relying on one global margin for all contacts.

## Soft Tuning Guide

1. **Try to make it work with the KFC bag asset**

   Start from the target bag asset instead of overfitting the solver to a simplified proxy. The real asset exposes the mesh quality, scale, and contact details that the final demo needs to handle.

2. **Improve convergence instead of blindly increasing stiffness**

   To get a stiffer-looking bag, the first goal should be better convergence. Very stiff material settings, roughly `tri_ke` / `tri_ka > 1e5`, are already in the stiff regime. Increasing them further can hurt convergence, reduce diagonal dominance, and make the simulation look worse.

3. **Use smaller time steps**

   Smaller time steps improve conditioning and diagonal dominance, so VBD can converge to the intended stiff behavior more reliably.

4. **Use lower resolution when debugging**

   Lower cloth resolution reduces DoFs and makes convergence easier. Once the behavior is stable, increase resolution carefully.

5. **Improve mesh quality**

   Better triangle quality improves conditioning. Bad or highly irregular triangles make the local VBD solves harder and can amplify instability.

6. **Turn off self-contact when it does not matter**

   Bag self-contact is expensive and may not materially change the result for this pickup demo. Turn it off unless the visual or physical behavior clearly requires it.

7. **Keep edge bending modest for poor-quality geometry**

   Edge bending should not be set too high on bad-quality meshes. High bending stiffness on irregular geometry can fight the solver and introduce artifacts instead of improving shape preservation.

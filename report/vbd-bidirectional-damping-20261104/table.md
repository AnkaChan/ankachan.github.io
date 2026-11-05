| Arm | Completed runs | Retained through hold | Mean J* mm | Mean V* m/s | Mean overlap mm | Mean substep ms |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| One-sided (closing only; previous default) | 7/7 | 5/7 | 25.44 | 0.495 | 0.780 | 23.74 |
| Bidirectional (new default) | 7/7 | 5/7 | 26.64 | 0.497 | 1.093 | 18.78 |

Means use the 7 cases completed in both arms.

| Case | Robot | Rows | Arm | Hold | Release | J* mm | V* m/s | Windows | Overlap mm | Substep ms | Finite |
| --- | --- | ---: | --- | --- | --- | ---: | ---: | ---: | ---: | ---: | --- |
| `m2_franka_pouch` | Franka | 512 | One-sided (closing only; previous default) | separated at 4.5 s | hand clear at end (from 4.5 s) | 16.96 | 0.695 | 120 | 0.998 | 57.66 | yes |
| `m2_franka_pouch` | Franka | 512 | Bidirectional (new default) | separated at 42.0 s | hand clear at end (from 42.0 s) | 28.34 | 0.532 | 120 | 0.998 | 23.29 | yes |
| `m3_franka_tote_empty` | Franka | 128 | One-sided (closing only; previous default) | separated at 7.3 s | hand clear at end (from 7.3 s) | 50.08 | 1.072 | 120 | 1.838 | 10.47 | yes |
| `m3_franka_tote_empty` | Franka | 128 | Bidirectional (new default) | separated at 17.9 s | hand clear at end (from 17.9 s) | 53.65 | 1.233 | 120 | 1.006 | 9.97 | yes |
| `r5_allegro_towel` | Allegro | 512 | One-sided (closing only; previous default) | retained through hold | hand clear at end (from 68.6 s) | 22.88 | 0.371 | 120 | 0.901 | 11.11 | yes |
| `r5_allegro_towel` | Allegro | 512 | Bidirectional (new default) | retained through hold | hand clear at end (from 68.7 s) | 21.61 | 0.372 | 120 | 2.938 | 11.20 | yes |
| `r5_franka_dress` | Franka | 512 | One-sided (closing only; previous default) | retained through hold | hand clear at end (from 68.6 s) | 24.55 | 0.344 | 120 | 0.209 | 27.20 | yes |
| `r5_franka_dress` | Franka | 512 | Bidirectional (new default) | retained through hold | hand clear at end (from 69.4 s) | 23.34 | 0.348 | 120 | 0.000 | 30.73 | yes |
| `r5_franka_patch_contact_v2` | Franka | 512 | One-sided (closing only; previous default) | retained through hold | hand clear at end (from 68.4 s) | 20.67 | 0.324 | 120 | 0.712 | 15.44 | yes |
| `r5_franka_patch_contact_v2` | Franka | 512 | Bidirectional (new default) | retained through hold | hand clear at end (from 68.4 s) | 21.16 | 0.334 | 120 | 2.710 | 20.44 | yes |
| `r5_franka_towel_contact_v2` | Franka | 512 | One-sided (closing only; previous default) | retained through hold | hand clear at end (from 68.3 s) | 24.45 | 0.359 | 120 | 0.799 | 16.31 | yes |
| `r5_franka_towel_contact_v2` | Franka | 512 | Bidirectional (new default) | retained through hold | hand clear at end (from 69.0 s) | 20.50 | 0.355 | 120 | 0.000 | 12.46 | yes |
| `r5_franka_trousers` | Franka | 64 | One-sided (closing only; previous default) | retained through hold | hand clear at end (from 68.7 s) | 18.52 | 0.301 | 120 | 0.000 | 27.98 | yes |
| `r5_franka_trousers` | Franka | 64 | Bidirectional (new default) | retained through hold | hand clear at end (from 69.4 s) | 17.85 | 0.301 | 120 | 0.000 | 23.38 | yes |

### All attempts

| Case | Arm | Storage | Status | Stopped at s | Hold | J* mm | V* m/s | Windows |
| --- | --- | --- | --- | ---: | --- | ---: | ---: | ---: |
| `m2_franka_pouch` | One-sided (closing only; previous default) | as selected by the synced-main campaign | completed | completed | separated at 4.5 s | 16.96 | 0.695 | 120 |
| `m2_franka_pouch` | Bidirectional (new default) | as selected by the synced-main campaign | completed | completed | separated at 42.0 s | 28.34 | 0.532 | 120 |
| `m3_franka_tote_empty` | One-sided (closing only; previous default) | as selected by the synced-main campaign | completed | completed | separated at 7.3 s | 50.08 | 1.072 | 120 |
| `m3_franka_tote_empty` | Bidirectional (new default) | as selected by the synced-main campaign | completed | completed | separated at 17.9 s | 53.65 | 1.233 | 120 |
| `r5_allegro_towel` | One-sided (closing only; previous default) | as selected by the synced-main campaign | completed | completed | retained through hold | 22.91 | 0.374 | 120 |
| `r5_allegro_towel` | One-sided (closing only; previous default) | 512-row repeat | completed | completed | retained through hold | 22.88 | 0.371 | 120 |
| `r5_allegro_towel` | Bidirectional (new default) | as selected by the synced-main campaign | error | 67.90 | run error | 23.60 | 0.375 | 120 |
| `r5_allegro_towel` | Bidirectional (new default) | 512-row repeat | completed | completed | retained through hold | 21.61 | 0.372 | 120 |
| `r5_franka_dress` | One-sided (closing only; previous default) | as selected by the synced-main campaign | completed | completed | retained through hold | 24.55 | 0.344 | 120 |
| `r5_franka_dress` | Bidirectional (new default) | as selected by the synced-main campaign | completed | completed | retained through hold | 23.34 | 0.348 | 120 |
| `r5_franka_patch_contact_v2` | One-sided (closing only; previous default) | as selected by the synced-main campaign | completed | completed | retained through hold | 20.67 | 0.324 | 120 |
| `r5_franka_patch_contact_v2` | Bidirectional (new default) | as selected by the synced-main campaign | completed | completed | retained through hold | 21.16 | 0.334 | 120 |
| `r5_franka_towel_contact_v2` | One-sided (closing only; previous default) | as selected by the synced-main campaign | error | 68.67 | run error | 24.11 | 0.358 | 120 |
| `r5_franka_towel_contact_v2` | One-sided (closing only; previous default) | 512-row repeat | completed | completed | retained through hold | 24.45 | 0.359 | 120 |
| `r5_franka_towel_contact_v2` | Bidirectional (new default) | as selected by the synced-main campaign | completed | completed | retained through hold | 22.77 | 0.351 | 120 |
| `r5_franka_towel_contact_v2` | Bidirectional (new default) | 512-row repeat | completed | completed | retained through hold | 20.50 | 0.355 | 120 |
| `r5_franka_trousers` | One-sided (closing only; previous default) | as selected by the synced-main campaign | completed | completed | retained through hold | 18.52 | 0.301 | 120 |
| `r5_franka_trousers` | Bidirectional (new default) | as selected by the synced-main campaign | completed | completed | retained through hold | 17.85 | 0.301 | 120 |

### Context: synced main 351b9d9f, one-sided damping (2026-11-02 campaign)

| Case | Hold (synced main) | J* mm | V* m/s | Overlap mm | Substep ms |
| --- | --- | ---: | ---: | ---: | ---: |
| `m2_franka_pouch` | separated at 49.5 s | 27.72 | 0.458 | 0.998 | 14.18 |
| `m3_franka_tote_empty` | separated at 13.0 s | 53.47 | 1.330 | 2.683 | 11.68 |
| `r5_allegro_towel` | retained through hold | 23.05 | 0.369 | 0.892 | 11.00 |
| `r5_franka_dress` | retained through hold | 25.84 | 0.383 | 0.098 | 22.34 |
| `r5_franka_patch_contact_v2` | retained through hold | 20.47 | 0.328 | 0.822 | 12.30 |
| `r5_franka_towel_contact_v2` | retained through hold | 24.67 | 0.356 | 0.592 | 11.64 |
| `r5_franka_trousers` | retained through hold | 18.21 | 0.301 | 0.277 | 19.17 |

## Poker cards (added 2026-11-04 afternoon)

Demo `example_cloth_poker_cards`, ball disabled, issue-3494 harness (settle score = mean over frames 40 to 79 of the largest card-vertex speed):

| Mode | Run | Settle score m/s | Mean kinetic energy J | Verdict |
| --- | ---: | ---: | ---: | --- |
| One-sided (closing only; previous default) | 1 | 0.615 | 2.33e-3 | bouncy |
| One-sided (closing only; previous default) | 2 | 0.630 | 2.38e-3 | bouncy |
| Bidirectional (new default) | 1 | 0.0705 | 5.12e-5 | settled |
| Bidirectional (new default) | 2 | 0.0704 | 4.84e-5 | settled |

Benchmark passive scene `s4_poker_cards_no_ball` (65 s, no hold or release criteria):

| Arm | J* mm | V* m/s | Windows | Overlap mm | Substep ms | Finite |
| --- | ---: | ---: | ---: | ---: | ---: | --- |
| One-sided (closing only; previous default) | 16.36 | 0.421 | 120 | 0.782 | 4.53 | yes |
| Bidirectional (new default) | 15.61 | 0.422 | 120 | 0.864 | 6.19 | yes |
| Synced-main campaign 2026-11-04 (one-sided, other source revision) | 19.33 | 0.402 | n/a | 0.800 | n/a | n/a |

Arm labels: the one-sided runs used `contact_damping_bidirectional=False`, the bidirectional runs `True`; the constructor default became `True` on the branch on 2026-11-04 afternoon, after the runs.

Later the option was split by contact type (commit 2cd7d288f): `soft_contact_damping_bidirectional` (default `True`; self-contact and rigid-soft) and `rigid_contact_damping_bidirectional` (default `False`; rigid-rigid). The runs above used the earlier single flag.

## Slip-error demo (2026-11-05)

A 16 cm cloth patch slides at 1 m/s over a fully pinned 2 m cloth sheet (self-contact only, interaction distance 2 mm, friction 0.5, Coulomb stop at 0.10 m). The distance-based gap rate of 68c18328a reads tangential slip as a closing gap and removes friction; the projected rate does not. Videos: `demo-slip/compare_<preset>.mp4`. Commit 68c18328a was removed from the branch on 2026-11-05 (new head 93f4eb8fb).

| `soft_contact_ke, soft_contact_kd` | upstream main 7f0c13af6, one-sided, projected rate | 81fd7250a, two-sided, projected rate | HEAD 2cd7d288f, two-sided, distance rate |
| --- | --- | --- | --- |
| 5e4, 0.1 | stops at 0.34 m, 0.40 s | stops at 0.34 m, 0.40 s | stops at 0.35 m, 0.42 s |
| 1e4, 1 | stops at 0.98 m, 1.68 s | stops at 0.97 m, 1.68 s | 0.78 m/s at 1.5 s, leaves the sheet at 2.23 s |
| 1e3, 0.1 | 0.83 m/s at 1.5 s, leaves at 2.10 s | 0.83 m/s at 1.5 s, leaves at 2.10 s | 0.95 m/s at 1.5 s, leaves at 1.90 s |
| 1e3, 10 | 0.93 m/s at 1.5 s, leaves at 1.95 s | 0.51 m/s at 1.5 s, still on the sheet at 2.5 s | 0.99 m/s at 1.5 s, rides 3 mm above the contact distance, leaves at 1.87 s |

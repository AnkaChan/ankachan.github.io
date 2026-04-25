---
title: 'Stable Neo-Hookean for VBD: Deriving the Per-Vertex Hessian'
date: 2026-04-24
permalink: /posts/2026/04/neohookean-vbd-cofactor-cancellation/
tags:
  - physics-simulation
  - VBD
  - neo-hookean
  - computer-graphics
layout: single
author_profile: true
read_time: true
comments: true
share: true
related: true
---

This post derives the per-vertex 3&times;3 Hessian block for the [stable Neo-Hookean](https://graphics.pixar.com/library/StableElasticity/paper.pdf) tet material under [VBD](https://anka-chen.me/publication/vbd-paper)-style block Gauss-Seidel, and shows how it lands as an unconditionally PSD expression with no clamp or eigenvalue projection required. The derivation is short but the algebraic cancellation it relies on is easy to miss, so it is worth writing out in full. The post is meant as a reference for anyone wiring stable Neo-Hookean into a VBD solver.

## Stable Neo-Hookean Energy and Its Hessian

For a tet with deformation gradient $\mathbf{F} \in \mathbb{R}^{3\times3}$ and energy parameters $\mu, \lambda$, the stable Neo-Hookean energy density is

$$\psi(\mathbf{F}) \;=\; \tfrac{\mu}{2}(I_C - 3) \;+\; \tfrac{\lambda}{2}(J - \alpha)^2,
\qquad I_C = \|\mathbf{F}\|_F^2,\quad J = \det \mathbf{F},\quad \alpha = 1 + \tfrac{\mu}{\lambda}.$$

The shift $\alpha$ ensures $\partial\psi/\partial \mathbf{F} = \mathbf{0}$ at the rest configuration $\mathbf{F} = \mathbf{I}$; it does *not* prevent inversion.

A subtlety worth flagging: the symbols $\mu, \lambda$ in this energy are *not* directly the Lam&eacute; parameters. Matching the small-strain limit of stable Neo-Hookean to linear elasticity ([Smith et al. &sect;3.4](https://graphics.pixar.com/library/StableElasticity/paper.pdf), eq. 13) gives the relation

$$\mu_\text{NH} \;=\; \mu_\text{Lam\'e}, \qquad \lambda_\text{NH} \;=\; \lambda_\text{Lam\'e} \;+\; \mu_\text{Lam\'e}.$$

So if you are exposing material constants to users in textbook Lam&eacute; convention, convert with $\lambda_\text{NH} = \lambda_\text{Lam\'e} + \mu_\text{Lam\'e}$ before plugging into the energy. Throughout the rest of this post, $\mu, \lambda$ refer to the Neo-Hookean parameters $\mu_\text{NH}, \lambda_\text{NH}$ as they appear in the energy expression above.

The first Piola--Kirchhoff stress is

$$\mathbf{P}(\mathbf{F}) \;=\; \frac{\partial \psi}{\partial \mathbf{F}} \;=\; \mu \mathbf{F} \;+\; s\,\text{cof}(\mathbf{F}),
\qquad s \;\equiv\; \lambda(J - \alpha).$$

Vectorising $\mathbf{F}$ column-major as $\text{vec}(\mathbf{F}) \in \mathbb{R}^9$, the Hessian splits into three pieces:

$$\mathbf{H}\_\text{elastic} \;=\; \underbrace{\mu \mathbf{I}\_9}\_{\mathbf{A}\_\mu} \;+\; \underbrace{\lambda\,\text{vec}(\text{cof}\,\mathbf{F})\;\text{vec}(\text{cof}\,\mathbf{F})^T}\_{\mathbf{A}\_\lambda} \;+\; \underbrace{s\,\frac{\partial^2 J}{\partial \mathbf{F}^2}}\_{\mathbf{A}\_\sigma}.$$

$\mathbf{A}\_\mu$ is a positive multiple of identity. $\mathbf{A}\_\lambda$ is rank-1 PSD. $\mathbf{A}\_\sigma$ is the only piece that can be indefinite: $s$ is negative for compressed tets, and $\partial^2 J/\partial \mathbf{F}^2$ has both positive and negative eigenvalues. Standard Newton-style implementations therefore SPD-project $\mathbf{A}\_\sigma$ (e.g. via [Smith--Kim eigenanalysis](https://www.tkim.graphics/RESCALE/elastic_eigenanalysis.pdf), or by clamping $s$ into a precomputed safe interval).

For VBD this projection turns out to be unnecessary. Showing why is the rest of the post.

## What VBD Needs

VBD updates one vertex at a time by solving the local Newton system

$$\mathbf{H}_{aa}\,\Delta\mathbf{x}_a \;=\; \mathbf{f}_a,$$

so the only piece of the elastic Hessian that ever enters a solve is the $3\times3$ diagonal block $\mathbf{H}_{aa}$ corresponding to a single vertex $a$. Off-diagonal blocks $\mathbf{H}_{ab}$ ($a \neq b$) influence convergence rate through Gauss-Seidel coupling but never appear inside any matrix inverse.

For a linear tet, $\mathbf{F}$ is affine in the vertex positions, so each vertex contributes via a fixed rest-frame weight $\mathbf{m}^a \in \mathbb{R}^3$ (a row of $\mathbf{D}_m^{-1}$):

$$\frac{\partial F_{ij}}{\partial x_a^\alpha} \;=\; \delta_{i\alpha}\, m_j^a.$$

Throughout, $i,j,k,l$ are deformation-gradient indices ($1\ldots 3$) and $\alpha,\beta$ are spatial-coordinate indices ($1\ldots 3$). The per-vertex 3&times;3 block is

$$\mathbf{H}\_{aa}^{\alpha\beta} \;=\; \sum_{ijkl}\,\frac{\partial F\_{ij}}{\partial x\_a^\alpha}\;\frac{\partial^2 \psi}{\partial F\_{ij}\,\partial F\_{kl}}\;\frac{\partial F\_{kl}}{\partial x\_a^\beta}.$$

We will plug each of the three pieces $\mathbf{A}\_\mu$, $\mathbf{A}\_\lambda$, $\mathbf{A}\_\sigma$ into this and simplify.

## Contracting $\mathbf{A}_\mu$ and $\mathbf{A}_\lambda$

For $\mathbf{A}\_\mu = \mu\,\mathbf{I}\_9$:

$$\mathbf{H}\_{aa}^{\alpha\beta}\big[\mathbf{A}\_\mu\big] \;=\; \mu\,\sum\_{ij}\,\delta\_{i\alpha}\,m\_j^a\,\delta\_{i\beta}\,m\_j^a \;=\; \mu\,\delta\_{\alpha\beta}\,\|\mathbf{m}^a\|^2.$$

So the $\mathbf{A}\_\mu$ contribution is $\mu\,\|\mathbf{m}^a\|^2\,\mathbf{I}\_3$.

For $\mathbf{A}\_\lambda = \lambda\,\text{vec}(\text{cof}\mathbf{F})\,\text{vec}(\text{cof}\mathbf{F})^T$, define $\mathbf{w}^a = \text{cof}(\mathbf{F})\,\mathbf{m}^a \in \mathbb{R}^3$. Then

$$\sum\_{ij}\,\delta\_{i\alpha}\,m\_j^a\,(\text{cof}\,\mathbf{F})\_{ij} \;=\; \sum\_j (\text{cof}\,\mathbf{F})\_{\alpha j}\, m\_j^a \;=\; w\_\alpha^a,$$

so $\mathbf{H}\_{aa}^{\alpha\beta}[\mathbf{A}\_\lambda] = \lambda\,w\_\alpha^a\,w\_\beta^a$, i.e. the rank-1 dyad $\lambda\,\mathbf{w}^a(\mathbf{w}^a)^T$.

Both contributions are PSD by inspection.

## Contracting $\mathbf{A}_\sigma$

The Hessian of $J = \det \mathbf{F}$ is the Levi-Civita identity

$$\frac{\partial^2 J}{\partial F\_{ij}\,\partial F\_{kl}} \;=\; \varepsilon\_{ikp}\,\varepsilon\_{jlq}\,F\_{pq}.$$

This tensor is nonzero in general, but contract it with $\partial F/\partial x\_a$ on both legs:

$$\begin{aligned}
\mathbf{H}\_{aa}^{\alpha\beta}\big[\mathbf{A}\_\sigma\big]
&= s\,\sum\_{ijkl}\,\delta\_{i\alpha}\,m\_j^a \,\cdot\, \varepsilon\_{ikp}\,\varepsilon\_{jlq}\,F\_{pq}\,\cdot\,\delta\_{k\beta}\,m\_l^a \\\\
&= s\,\varepsilon\_{\alpha\beta p}\,F\_{pq}\,\sum\_{j,l}\, m\_j^a\,\varepsilon\_{jlq}\,m\_l^a \\\\
&= s\,\varepsilon\_{\alpha\beta p}\,F\_{pq}\,(\mathbf{m}^a \times \mathbf{m}^a)\_q \\\\
&= 0.
\end{aligned}$$

The inner sum is the cross product of $\mathbf{m}^a$ with itself, which vanishes for any vector. The cancellation goes through for any $\mathbf{F}$ (including $\det \mathbf{F} \leq 0$), any $\mathbf{m}^a$, and any scalar $s$.

The structural reason: $\partial F/\partial x_a^\alpha = \mathbf{e}\_\alpha \otimes \mathbf{m}^a$ is a rank-1 dyad. Sandwiching the antisymmetric tensor $\partial^2 J/\partial \mathbf{F}^2$ between two copies of the *same* rank-1 dyad pins the $j,l$ indices to the same vector $\mathbf{m}^a$, and antisymmetry collapses the contraction to zero. Off-diagonal blocks $\mathbf{H}_{ab}$ for $a \neq b$ replace $\mathbf{m}^a \times \mathbf{m}^a$ with $\mathbf{m}^a \times \mathbf{m}^b$, which is generically nonzero — they do see $\mathbf{A}\_\sigma$.

## The Per-Vertex Block

Combining the three contractions,

$$\boxed{\;\mathbf{H}\_{aa} \;=\; \mu\,\|\mathbf{m}^a\|^2\,\mathbf{I}\_3 \;+\; \lambda\,\mathbf{w}^a (\mathbf{w}^a)^T,\qquad \mathbf{w}^a = \text{cof}(\mathbf{F})\,\mathbf{m}^a.\;}$$

Both summands are PSD for any $\mathbf{F}$ and any $\mathbf{m}^a$:

- $\mu\,\|\mathbf{m}^a\|^2\,\mathbf{I}\_3$ is a positive multiple of identity.
- $\lambda\,\mathbf{w}^a(\mathbf{w}^a)^T$ is a rank-1 outer product with positive coefficient.

So the per-vertex block is unconditionally PSD with no projection step. The cofactor-derivative term that complicates Newton-style implementations does not contribute to it.

The corresponding per-vertex elastic force is the same expression evaluated against the *true* (unclamped) stress:

$$\mathbf{f}\_a \;=\; -\mathbf{P}(\mathbf{F})\,\mathbf{m}^a \;=\; -\mu\,\mathbf{F}\,\mathbf{m}^a \;-\; s\,\mathbf{w}^a.$$

Forces use the real $s = \lambda(J-\alpha)$ even when it is negative; this is what carries the inversion-recovery signal in stable Neo-Hookean.

## Implementation

The full evaluator multiplies the result by the rest volume and (optionally) adds a damping contribution. In Warp the elastic part is just:

```python
@wp.func
def evaluate_volumetric_neo_hookean_force_and_hessian(
    tet_id: int, v_order: int,
    pos: wp.array[wp.vec3],
    tet_indices: wp.array2d[wp.int32],
    Dm_inv: wp.mat33,
    mu: float, lmbd: float,
):
    v0 = pos[tet_indices[tet_id, 0]]
    v1 = pos[tet_indices[tet_id, 1]]
    v2 = pos[tet_indices[tet_id, 2]]
    v3 = pos[tet_indices[tet_id, 3]]
    rest_volume = 1.0 / (wp.determinant(Dm_inv) * 6.0)

    # F = D_s D_m^{-1}
    Ds = wp.matrix_from_cols(v1 - v0, v2 - v0, v3 - v0)
    F = Ds * Dm_inv

    # Per-vertex weight m^a (a row of D_m^{-1}; vertex 0 is the negative sum)
    if v_order == 0:
        m = wp.vec3(-(Dm_inv[0,0] + Dm_inv[1,0] + Dm_inv[2,0]),
                    -(Dm_inv[0,1] + Dm_inv[1,1] + Dm_inv[2,1]),
                    -(Dm_inv[0,2] + Dm_inv[1,2] + Dm_inv[2,2]))
    elif v_order == 1:
        m = wp.vec3(Dm_inv[0,0], Dm_inv[0,1], Dm_inv[0,2])
    elif v_order == 2:
        m = wp.vec3(Dm_inv[1,0], Dm_inv[1,1], Dm_inv[1,2])
    else:
        m = wp.vec3(Dm_inv[2,0], Dm_inv[2,1], Dm_inv[2,2])

    # Stress (uses the TRUE s, no clamp)
    J     = wp.determinant(F)
    alpha = 1.0 + mu / lmbd
    s     = lmbd * (J - alpha)
    cof   = compute_cofactor(F)             # adjugate via cross products

    # Per-vertex auxiliary vectors
    Fm = F * m                              # mu term
    w  = cof * m                            # lambda term: w^a = cof(F) m^a

    # Force: f_a = -P m^a
    force = -rest_volume * (mu * Fm + s * w)

    # Hessian: H_aa = mu ||m||^2 I + lambda w w^T
    I3      = wp.identity(n=3, dtype=float)
    hessian = rest_volume * (mu * wp.dot(m, m) * I3 + lmbd * wp.outer(w, w))
    return force, hessian
```

The 9&times;9 elastic Hessian never gets assembled; nothing is clamped. Compared to a textbook implementation that builds $\mathbf{A}\_\mu + \mathbf{A}\_\lambda + \mathbf{A}\_\sigma$ as a $9\times9$ matrix, projects it, then contracts with $\partial F/\partial x_a$, this is a small constant-factor saving per tet per VBD inner iteration.

## Why This Doesn't Extend to Triangle Membranes

It is tempting to apply the same logic to a stable Neo-Hookean *triangle* membrane and conclude that its per-vertex 3&times;3 block also drops the cofactor-derivative term. It does not. The cancellation hinges on a structural property of the volumetric case that the membrane does not share.

For a 3D triangle in 2D rest space, the deformation gradient is $\mathbf{F} \in \mathbb{R}^{3\times 2}$ with columns $\mathbf{f}\_0, \mathbf{f}\_1$. The natural area scalar that plays the role of $J$ is

$$J_s \;=\; \sqrt{\det(\mathbf{F}^T \mathbf{F})} \;=\; \|\mathbf{f}_0 \times \mathbf{f}_1\|.$$

Two things change relative to the volumetric case:

1. **$J_s$ is not a polynomial in $\mathbf{F}$** (it is a square root). Its second derivative does not have the clean Levi-Civita form $\partial^2 J/\partial F\_{ij}\partial F\_{kl} = \varepsilon\_{ikp}\varepsilon\_{jlq}F\_{pq}$. There is in fact an extra $-(1/J_s)\,\nabla J_s \otimes \nabla J_s$ piece coming from differentiating the $1/J_s$ factor in $\nabla J_s = (\mathbf{n}\cdot\nabla\mathbf{n})/J_s$.
2. **Rows and columns of $\mathbf{F}$ live in different spaces.** Row indices run over 3D world coordinates $(i \in \\{1,2,3\\})$, column indices run over 2D parameter coordinates $(j \in \\{1,2\\})$. The 3-index Levi-Civita $\varepsilon\_{jlq}$ that produced $\mathbf{m}\times\mathbf{m}$ in the volumetric proof has nowhere to live on the column-index leg --- there are only two column indices to antisymmetrise over, not three.

Concretely, the per-vertex contraction in the membrane case becomes

$$\mathbf{H}\_{aa}^{\alpha\beta}\big[\mathbf{A}\_\sigma^\text{2D}\big] \;=\; s\,\sum\_{j,l \in \\{0,1\\}}\,m\_j^a\,m\_l^a\,\frac{\partial^2 J\_s}{\partial F\_{\alpha j}\,\partial F\_{\beta l}},$$

with no antisymmetric-in-$(j,l)$ structure to exploit. Working through the algebra with $\mathbf{n} = \mathbf{f}_0\times\mathbf{f}_1$ gives a clean form for the contracted block:

$$\mathbf{H}\_{aa}\big[\mathbf{A}\_\sigma^\text{2D}\big] \;=\; \frac{s}{J_s}\,\Big(\|\mathbf{w}\|^2\,\mathbf{I}\_3 \;-\; \mathbf{w}\mathbf{w}^T \;-\; \boldsymbol{\nabla}J_s\,\boldsymbol{\nabla}J_s^T\Big),$$

with $\mathbf{w} = \mathbf{f}\_1\,m\_0^a - \mathbf{f}\_0\,m\_1^a$ and $\boldsymbol{\nabla}J_s = \mathbf{g}\_0\,m\_0^a + \mathbf{g}\_1\,m\_1^a$, $\mathbf{g}\_\alpha = \partial J_s/\partial \mathbf{f}\_\alpha$. None of these vanish in general; in fact the $\|\mathbf{w}\|^2 \mathbf{I}\_3 - \mathbf{w}\mathbf{w}^T$ piece projects onto the direction *normal* to the membrane and produces a genuine out-of-plane stiffness. The cofactor-derivative term carries real physics here.

**Geometric reading.** The volumetric tet has no "extra" direction --- both legs of $\mathbf{F}$ span the same 3D space, and the Levi-Civita pattern absorbs all three coordinate axes uniformly. The membrane has a normal direction that is *not* in the column space of $\mathbf{F}$; the second-derivative term contributes precisely along that normal. Stripping it would weaken out-of-plane resistance and change the physics, not just save flops.

## The Tight PSD Clamp for the Membrane

Although the cofactor-derivative term has to stay, the per-vertex 3&times;3 block still has a clean PSD characterisation. Combining the three contractions for the membrane case,

$$\mathbf{H}\_{aa} \;=\; \mu\,\|\mathbf{m}^a\|^2\,\mathbf{I}\_3 \;+\; (\lambda - r)\,\boldsymbol{\nabla}J\_s\,\boldsymbol{\nabla}J\_s^T \;+\; r\,\big(\|\mathbf{w}\|^2\,\mathbf{I}\_3 - \mathbf{w}\mathbf{w}^T\big), \qquad r \;\equiv\; \frac{s}{J_s}.$$

(The $\lambda\,\boldsymbol{\nabla}J\_s\,\boldsymbol{\nabla}J\_s^T$ piece comes from $\mathbf{A}\_\lambda$ in the membrane case --- the rank-1 cofactor outer product specialises to $\boldsymbol{\nabla}J\_s\,\boldsymbol{\nabla}J\_s^T$ here. The $-r\,\boldsymbol{\nabla}J\_s\,\boldsymbol{\nabla}J\_s^T$ piece comes from the $\mathbf{A}\_\sigma$ contraction, which is why the two combine.)

Two algebraic identities make this block diagonalisable.

**Lemma 1.** $\mathbf{w} \cdot \boldsymbol{\nabla}J_s = 0$.

*Proof.* Direct computation using $\mathbf{g}\_\alpha = \partial J_s/\partial \mathbf{f}\_\alpha$ and $J_s^2 = AB - C^2$ with $A = \|\mathbf{f}\_0\|^2,\ B = \|\mathbf{f}\_1\|^2,\ C = \mathbf{f}\_0\cdot \mathbf{f}\_1$ gives
$\mathbf{f}\_1\cdot\mathbf{g}\_0 = \mathbf{f}\_0\cdot\mathbf{g}\_1 = 0$ and $\mathbf{f}\_0\cdot\mathbf{g}\_0 = \mathbf{f}\_1\cdot\mathbf{g}\_1 = J_s$. Expanding $\mathbf{w}\cdot\boldsymbol{\nabla}J\_s$ in $(m\_0^a, m\_1^a)$ and substituting collapses the four terms to $J\_s\,m\_0^a m\_1^a - J\_s\,m\_0^a m\_1^a = 0$. $\square$

**Lemma 2.** $\|\mathbf{w}\| = \|\boldsymbol{\nabla}J_s\|$.

*Proof.* Compute $\mathbf{w}\times\boldsymbol{\nabla}J_s$ using $\mathbf{f}\_i\times\mathbf{g}\_j$ which all reduce to scalar multiples of $\mathbf{n} = \mathbf{f}\_0\times\mathbf{f}\_1$. The four cross products give

$$\mathbf{w}\times\boldsymbol{\nabla}J\_s \;=\; -\frac{\mathbf{n}}{J\_s}\,\big(A(m\_1^a)^2 - 2C\,m\_0^a m\_1^a + B(m\_0^a)^2\big) \;=\; -\|\mathbf{w}\|^2\,\hat{\mathbf{n}},$$

where the last equality uses $\|\mathbf{w}\|^2 = A(m\_1^a)^2 - 2C\,m\_0^a m\_1^a + B(m\_0^a)^2$ and $\hat{\mathbf{n}} = \mathbf{n}/J\_s$. By Lemma 1, $\mathbf{w}\perp\boldsymbol{\nabla}J\_s$, so $\|\mathbf{w}\times\boldsymbol{\nabla}J\_s\| = \|\mathbf{w}\|\,\|\boldsymbol{\nabla}J\_s\|$. Equating with the right-hand side gives $\|\mathbf{w}\|\,\|\boldsymbol{\nabla}J\_s\| = \|\mathbf{w}\|^2$. $\square$

**Diagonalisation.** Choose the orthonormal basis $\\{\hat{\mathbf{w}}, \widehat{\boldsymbol{\nabla}J\_s}, \hat{\mathbf{n}}\\}$ where $\hat{\mathbf{n}}$ is the unit triangle normal (orthogonal to both $\mathbf{w}$ and $\boldsymbol{\nabla}J\_s$ by Lemma 1 and the cross-product computation). Off-diagonal entries of $\mathbf{H}\_{aa}$ vanish in this basis (each of the three building blocks $\mathbf{I}\_3$, $\boldsymbol{\nabla}J\_s\,\boldsymbol{\nabla}J\_s^T$, $\|\mathbf{w}\|^2\mathbf{I}\_3 - \mathbf{w}\mathbf{w}^T$ is diagonal in it), and using $\|\mathbf{w}\| = \|\boldsymbol{\nabla}J\_s\|$ to combine the $r$-terms in the $\widehat{\boldsymbol{\nabla}J\_s}$ direction:

| Direction | Eigenvalue |
| --- | --- |
| $\hat{\mathbf{w}}$ | $\mu\,\|\mathbf{m}^a\|^2$ |
| $\widehat{\boldsymbol{\nabla}J\_s}$ | $\mu\,\|\mathbf{m}^a\|^2 + \lambda\,\|\boldsymbol{\nabla}J\_s\|^2$ |
| $\hat{\mathbf{n}}$ | $\mu\,\|\mathbf{m}^a\|^2 + r\,\|\mathbf{w}\|^2$ |

The first two eigenvalues are PSD for any $r$ --- the $r$-dependence in the $\widehat{\boldsymbol{\nabla}J\_s}$ direction cancels exactly because $\|\mathbf{w}\| = \|\boldsymbol{\nabla}J\_s\|$. Only the normal direction sees $r$, and the PSD condition there is

$$r \;\geq\; -\frac{\mu\,\|\mathbf{m}^a\|^2}{\|\mathbf{w}\|^2}.$$

The right-hand side is geometry-dependent. For a *uniform* clamp that works for every triangle and every vertex, the only safe choice is $r \geq 0$, i.e.

$$\boxed{\;s\_\text{clamp} \;=\; \max(0, s).\;}$$

This is tight in the uniform sense: any larger lower bound on $s$ would change the physics for at least some configurations where the unclamped block is already PSD; any smaller (more permissive) bound risks an indefinite block in some configuration.

A geometry-aware solver could instead use the per-element lower bound $s \geq -\mu\,\|\mathbf{m}^a\|^2 J\_s/\|\mathbf{w}\|^2$ and recover a slightly looser projection, but the bookkeeping cost rarely justifies it. Force always uses the unclamped $s = \lambda(J\_s - \alpha)$, exactly as in the volumetric case.

A stable Neo-Hookean triangle evaluator therefore keeps the second-derivative contribution and applies the simple uniform clamp $s\_\text{clamp} = \max(0, s)$. The simple result for the volumetric tet is genuinely a special property of square deformation gradients.

## Sanity Checks Before Shipping

A few things worth verifying when wiring this up:

1. The shift $\alpha = 1 + \mu/\lambda$ depends on $\lambda \neq 0$. Guard against $\lambda$ near zero (e.g. $\lambda \mapsto \text{sign}(\lambda)\,\max(|\lambda|, \epsilon)$).
2. Use the explicit cofactor / adjugate $\text{cof}(\mathbf{F})$ rather than $J\,\mathbf{F}^{-T}$. The adjugate is a polynomial in the entries of $\mathbf{F}$ and remains well-defined as $J \to 0$, while $\mathbf{F}^{-T}$ blows up.
3. The force expression carries the *signed* $s = \lambda(J - \alpha)$, including when $J < 0$ (inverted tet) or $J < \alpha$ (compressed). This is what pulls inverted tets back through $J = 0$.
4. The cancellation breaks for higher-order elements (quadratic tets, hexes, isogeometric basis), where $\partial F/\partial x_a$ is no longer a constant rank-1 dyad. If you adapt this evaluator to a non-linear element, the $\mathbf{A}\_\sigma$ term reappears and needs SPD projection.
5. The cancellation also breaks for off-diagonal blocks, so a global Newton solver assembling $\mathbf{H}_{ab}$ for $a \neq b$ does need a clamp. VBD's per-vertex block does not.

## Summary

For a linear tet with a stable Neo-Hookean energy, the VBD per-vertex block reduces to

$$\mathbf{H}\_{aa} \;=\; \mu\,\|\mathbf{m}^a\|^2\,\mathbf{I}\_3 \;+\; \lambda\,(\text{cof}\,\mathbf{F}\,\mathbf{m}^a)(\text{cof}\,\mathbf{F}\,\mathbf{m}^a)^T,$$

unconditionally PSD without any projection of the cofactor-derivative term. The cancellation comes from $\partial F/\partial x_a^\alpha = \delta_{i\alpha}\,m_j^a$ being a rank-1 dyad and the Hessian of $\det\mathbf{F}$ being antisymmetric in matching index pairs, so the contraction collapses through $\mathbf{m}^a \times \mathbf{m}^a = 0$. Force uses the unclamped stress and inversion recovery is carried by the gradient, not the Hessian.

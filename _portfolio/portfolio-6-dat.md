---
title: "Divide and Truncate (DAT)"
excerpt: "A penetration-free and inversion-free contact framework for coupled multi-physics systems, integrated into the VBD solver of NVIDIA Newton. SIGGRAPH 2026."
collection: portfolio
header:
  teaser: publications/dat-teaser.jpg
---

[View Project Page](/Projects/DAT/) | [Paper](https://arxiv.org/pdf/2604.15513) | [Code (Newton)](https://github.com/newton-physics/newton/tree/main/newton/_src/solvers/vbd)

DAT is a unified framework for penetration-free and inversion-free contact resolution across rigid bodies, soft volumetric objects, thin shells, rods, and animated elements. It divides space into zones and truncates motion at their boundaries, works independently of material properties, and can be applied as a post-processing step with any iterative optimizer. Planar-DAT restricts only normal motion toward contact surfaces, removing artificial damping and deadlock. DAT ships in the VBD solver of NVIDIA's open-source Newton physics engine.

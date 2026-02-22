---
permalink: /
title: "About"
excerpt: "About me"
author_profile: true
redirect_from:
  - /about/
  - /about.html
---

## About Me

I'm a Research Scientist at [NVIDIA](https://www.nvidia.com/), where I work on physics-based simulation and computer graphics. I received my PhD in Computing from the [University of Utah](https://www.utah.edu/) in 2024, where my research focused on **capturing the deformation of non-rigid objects and studying the physics problems that arise from it**. I was advised by Prof. Cem Yuksel and Prof. Ladislav Kavan.

I have interdisciplinary education backgrounds, which covers electric engineering, mathematics and computer science. I obtained by master's degree of computational mathematics with first class honours from the School of Mathematics at [DLUT](https://en.dlut.edu.cn/). I also won the title of Outstanding Graduate and Outstanding Master Graduation Thesis. Prior to that I have a bachelor's degree of Eletric Engineering at Hunan University.

**Explanation/pronunciation of My Name**: Please don't be confused by my first name, "He." Although it looks like a pronoun, it is pronounced as "hə" (similar to "her" without the "r"), which means "prominent" in Chinese. However, I would greatly appreciate it if you could call me by my preferred name, Anka.

## Education

**PhD of Computing** | [University of Utah](https://www.utah.edu/) | 2019 - 2024
- Advisor: Prof. Cem Yuksel, Prof. Ladislav Kavan
- Focus: Computer Graphics, Physics-Based Animation
- **Award:** ACM SIGGRAPH / Eurographics Symposium on Computer Animation (SCA) 2025 Best Doctoral Dissertation Award

**Master of Computational Mathematics** | [Dalian University of Technology](https://en.dlut.edu.cn/) | 2016 - 2019
- First Class Honours
- Outstanding Graduate, Outstanding Master Graduation Thesis

**Bachelor of Electrical Engineering** — [Hunan University](https://en.wikipedia.org/wiki/Hunan_University) — 2012 - 2016

## Work Experience

**Research Scientist** | [NVIDIA](https://www.nvidia.com/) | July 2024 - Present
- Focus: Physics-based simulation, Computer Graphics
- Location: Redmond, Washington (Remote)

## Research

My current research topics include:

- **Next Gen Motion Capture**: A system to capture thousand of unique points on the surface of a deformable object, including human body and clothes. Directly obtain 4D (spatial temporal) data from the motion.
- **Physics Based Animation**: My works covers time integrator, collision detection and resolution.
- **Point Cloud**: Point cloud normal estimation, point cloud denoising, point cloud alignment.
- **3D Scanning**: Data algning, texture synthesis, mesh reconstruction. Currently focus on registration of depth and feature preserving reconstruction.
- [**MeshFrame**](https://github.com/MeshFrame/MeshFrame/): A lightweight, efficient, header-only mesh processing framework with better efficiency superior to other state-of-the-art libraries. It supports dynamic mesh structure editing, supports runtime dynamic properties, supports triangle/tetrahedral mesh. It also includes a very fast mesh simplification application.
- **Volumetric parameterization**: Given an abitrary volumetric object in form of tetrahedral mesh, and make a bijective PL(Piecewise Linear) map from object to a canonical region. I have a parameterization method preserving as much original mesh subdivision structure as possible.
- [**3D Face Reconstruction**](/projects/face-recon/): I have been researching on full face reconstruction from multi-angle RGB-D data and face sequence reconstruction. I have developed some cutting-edge application on both PC and mobile platform.

## Selected Publications

{% assign featured_pubs = site.publications | where: "featured", true | sort: "date" | reverse %}
{% for post in featured_pubs %}
<div style="display: flex; gap: 1.5em; margin-bottom: 2.5em; align-items: flex-start;">
  <div style="flex-shrink: 0; width: 250px;">
    <img src="/images/{{ post.header.teaser }}" alt="{{ post.title }}" style="width: 100%; border-radius: 4px;">
  </div>
  <div>
    <h3 style="margin-top: 0;"><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <p style="margin: 0.5em 0; font-style: italic;">{{ post.venue }}, {{ post.date | date: "%Y" }}</p>
    <p>{{ post.excerpt | markdownify }}</p>
  </div>
</div>
{% endfor %}

[See all publications →](/publications/)

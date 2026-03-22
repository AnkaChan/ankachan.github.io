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
I am currently a research scientist at NVIDIA specializing in physics-based simulation and physical AI. I got my PhD in
Graphics from the University of Utah. My recent research concentrates on creating parallel solutions for implicit time integration,
aiming to deliver robust and efficient strategies for handling elasticity, collisions, and the dynamics of rigid bodies. Additionally,
I am investigating the use of deep learning methods to speed up the convergence of physics solvers. A significant portion of my
research also includes developing high-resolution deformation capture systems. Those systems are designed to collect real-world
data on non-rigid objects, facilitating the study of inverse physics problems. In summary, my work serves the goal of building infrastructure for Physical AI.

I am a member of the team that developed the [Warp](https://github.com/NVIDIA/warp) differentiable programming language and the [Newton](https://github.com/newton-physics/newton) solver. Newton is a next-generation world simulator for physical AI that NVIDIA is actively developing. I am the creator and one of the principal developers of one of Newton's core solvers: the [VBD (Vertex Block Descent) solver](/publication/2024-vbd). The VBD solver represents one of my principal contributions during my doctoral studies. It is a robust and parallel physics solver optimized for GPU architectures, capable of simulating coupled multiphysics systems including robots, soft bodies, cloth, cables, and rigid bodies. VBD achieves unprecedented precision and computational efficiency, establishing it as an ideal algorithm for constructing world simulators. 

## Explanation/pronunciation of My Name

Please don't be confused by my first name, "He." Although it looks like a pronoun, it is pronounced as "hə" (similar to "her" without the "r"), which means "prominent" in Chinese. However, I would greatly appreciate it if you could call me by my preferred name, Anka.

## Education

**PhD of Computing** | [University of Utah](https://www.utah.edu/) | 2019 - 2024
- Advisor: Prof. Cem Yuksel
- Focus: Computer Graphics, Physics-Based Animation
- **Award:** ACM SIGGRAPH / Eurographics Symposium on Computer Animation (SCA) 2025 Best Doctoral Dissertation Award

**Master of Computational Mathematics** | [Dalian University of Technology](https://en.dlut.edu.cn/) | 2016 - 2019
- Graduated with Outstanding Master Graduation Thesis Award

**Bachelor of Electrical Engineering** — [Hunan University](https://en.wikipedia.org/wiki/Hunan_University) — 2012 - 2016

## Dissertation

**[Towards Realistic Real-Time Physics-Based Simulation](/dissertation/)**
*PhD Dissertation, Kahlert School of Computing, The University of Utah, December 2024*
- **Award:** ACM SIGGRAPH / Eurographics Symposium on Computer Animation (SCA) 2025 Best Doctoral Dissertation Award
- <a href="/files/Anka_Chen_Disseration_Final_LowRes.pdf" target="_blank">PDF (Low-Res, 7.9 MB)</a>
- <a href="https://www.dropbox.com/scl/fi/7bbhqccsuzoc4kf3ja1w5/Anka_Chen_Disseration_Final.pdf?rlkey=p1rkhcvoxd9rgyr0mlfvyg6io&st=xn3f59tn&dl=0" target="_blank">PDF (Full-Res, 65 MB)</a>

## Work Experience

**Research Scientist** | [NVIDIA](https://www.nvidia.com/) | July 2024 - Present
    - Focus: Physics-based simulation, Physical AI

## Selected Publications

{% assign featured_pubs = site.publications | where: "featured", true | sort: "date" | reverse %}
{% for post in featured_pubs %}
<div style="margin-bottom: 2.5em;">
  <div style="max-width: 500px; margin-bottom: 0.75em;">
    <img src="/images/{{ post.header.teaser }}" alt="{{ post.title }}" style="max-width: 100%; max-height: 250px; border-radius: 4px; object-fit: contain;">
  </div>
  <h3 style="margin-top: 0;"><a href="{{ post.url }}">{{ post.title }}</a></h3>
  <p style="margin: 0.5em 0; font-style: italic;">{{ post.venue }}, {{ post.date | date: "%Y" }}</p>
  <p>{{ post.excerpt | markdownify }}</p>
</div>
{% endfor %}

[See all publications →](/publications/)

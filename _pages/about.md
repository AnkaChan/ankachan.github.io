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
data on non-rigid objects, facilitating the study of inverse physics problems. In summary, my works serves the goal of building infrastructure for Physical AI.

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

## Work Experience

**Research Scientist** | [NVIDIA](https://www.nvidia.com/) | July 2025 - Present
*Kirkland, Washington*

I am part of the team that created the [Warp](https://github.com/NVIDIA/warp) differentiable language and Newton solver. Newton solver is the next-generation world simulator for physical AI that NVIDIA is actively promoting. I created and am leading the development of one of its core solvers: the VBD (Vertex Block Descent) solver. VBD is a robust and parallel physics solver optimized for GPU architectures, capable of simulating coupled multiphysics systems including robots, soft bodies, cloth, cables, and rigid bodies. VBD provides unprecedented precision and speed, making it the ideal algorithm for building world simulators. This algorithm is one of my representative works during my PhD and contributed to winning the ACM SIGGRAPH / Eurographics Symposium on Computer Animation (SCA) 2025 Best Doctoral Dissertation Award.

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

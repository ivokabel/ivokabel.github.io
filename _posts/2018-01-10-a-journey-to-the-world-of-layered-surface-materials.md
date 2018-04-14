---
layout: post
title: A Journey to the World of Layered Surface Materials
comments: true
---

I decided to implement some kind of multi-layered material model (BSDF) in my pet renderer since they offer much more expressivity than basic models (e.g. smooth Fresnel, glossy micro-facet models, or diffuse Lambert) and I also wanted to learn some more advanced rendering-related tech.

In the beginning I wanted to just directly implement the paper [Arbitrarily Layered Micro-Facet Surfaces](https://www.cg.tuwien.ac.at/research/publications/2007/weidlich_2007_almfs/) by Andrea Weidlich and Alexander Wilkie. However, it turned out that the paper is too ad-hoc to my standards and their justifications too vague or even non-existent, so I decided to come up with my own approach, which is heavily inspired by the ideas from the paper, but improves some of it's problems.

I decided to restrict the model to just two layers (1 -- outer and 2 -- inner). This decision simplifies both theory and implementation a bit while still having the rich expressivity of a layered model. In my following explanation I will ignore the fact that there can be more than two layers even in the original Weidlich-Wilkie paper.

Since this post is fairly large, I decided to split it into two chapters:

- [Chapter 1: Weidlich-Wilkie Layered Model](chapter-1-weidlich-wilkie-layered-model.html), where I explain the original Weidlich-Wilkie model, which I sometimes refer to as the WWL model.
- [Chapter 2: Smooth Refraction Approximation Layered Model](chapter-2-smooth-refraction-approximation-layered-model.html), where I explain and analyse my approach to layering.
  - [Chapter 2.1: Energy conserving BSDF under a smooth refractive interface](chapter-2.1-energy-conserving-BSDF-under-smooth-refractive-interface.html), where I derive the energy-conserving form of a BSDF seen and illuminated through a smooth refractive interface needed in the chapter 2.
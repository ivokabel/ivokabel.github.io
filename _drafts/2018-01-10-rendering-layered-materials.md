---
layout: post
title: "Rendering Layered Materials"
comments: true
---

I decided to implement some kind of multi-layered material model (BSDF) in my pet renderer since they offer much more expressivity than basic models (e.g. smooth Fresnel, glossy micro-facet models, or diffuse Lambert) and because I wanted to learn some more advanced rendering-related tech.

My original plan was to directly implement the paper [Arbitrarily Layered Micro-Facet Surfaces](https://www.cg.tuwien.ac.at/research/publications/2007/weidlich_2007_almfs/) by Andrea Weidlich and Alexander Wilkie, which I sometimes refer to as the WWL model. However, it turned out that their model is too ad-hoc and their justifications too vague or even non-existent, so I decided to come up with my own solution, which is heavily inspired by the ideas from the paper, but improves some of it's problems.

I decided to restrict the model to just two layers (1 -- outer and 2 -- inner). This decision simplifies both theory and implementation a bit while still having the rich expressivity of a layered model. In my following explanation I will ignore the fact that there can be more than two layers even in the original Weidlich-Wilkie paper.

Since the amount of resulting text is fairly large, I decided to split it into several parts:

- [Weidlich-Wilkie Layered Model](rendering-layered-materials-weidlich-wilkie-layered-model.html), where I explain the original Weidlich-Wilkie model.
- [Smooth Refraction Approximation Layered Model](rendering-layered-materials-smooth-refraction-approximation-layered-model.html), where I explain my own approach to layering.
- [Energy Conserving BSDF Under a Smooth Refractive Interface](rendering-layered-materials-energy-conserving-BSDF-under-smooth-refractive-interface.html), where I derive the energy-conserving form of a BSDF seen and illuminated through a smooth refractive interface needed in the previous part.
<!-- - A More Detailed SRAL Model Analysis, ...-->
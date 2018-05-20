---
layout: post
title: "Rendering Layered Materials"
comments: true
---

<p style="text-align: center">
<img src="../../../images/SRAL/BlogTeaser_512s.jpg" alt="" width="700" />
</p>

I decided to implement a multi-layered material model (BSDF) in my pet renderer since layering offers much more expressivity than basic models (e.g. Fresnel, micro-facets, or Lambert) and because it is one of the rendering-related technologies I wanted to learn.

My original plan was to directly implement the paper [Arbitrarily Layered Micro-Facet Surfaces](https://www.cg.tuwien.ac.at/research/publications/2007/weidlich_2007_almfs/) by Andrea Weidlich and Alexander Wilkie, which I sometimes refer to as the WWL model, or just the original model. However, it turned out that their model is more ad-hoc and practically oriented. So I decided to come up with my own solution which would be heavily inspired by the ideas from the original paper, but built on a more theoretical foundation so that some weaknesses of a practically oriented approach could be overcome.

I decided to restrict the model to just two layers, instead of the original paper's general approach with arbitrary amount of layers. This decision simplifies both theory and implementation while still having the rich expressivity of a layered model. In my explanation I will ignore the fact that there can be more than two layers in the original model.

Since the amount of resulting text grew fairly large, I decided to split it into several parts:

- [Weidlich-Wilkie Layered Model](rendering-layered-materials-weidlich-wilkie-layered-model.html), where I explain the original Weidlich-Wilkie model.
- [Smooth Refraction Approximation Layered Model](rendering-layered-materials-smooth-refraction-approximation-layered-model.html), where I explain my own approach to layering.
- [Energy Conserving BSDF Under a Smooth Refractive Interface](rendering-layered-materials-energy-conserving-BSDF-under-smooth-refractive-interface.html), where I derive the energy-conserving form of a BSDF seen and illuminated through a smooth refractive interface needed in the previous part.

<!-- - A More Detailed SRAL Model Analysis, ...-->


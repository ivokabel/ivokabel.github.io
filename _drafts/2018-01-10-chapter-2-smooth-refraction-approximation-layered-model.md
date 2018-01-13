---
layout: post
title: "Chapter 2: Smooth Refraction Approximation Layered Model (SRAL)"
comments: true
---

This is the second chapter of the the blog post [A Journey to the World of Layered Surface Materials](a-journey-to-the-world-of-layered-surface-materials.html). Here I present my approach to the layered surface material as an alternative to the [layered model](https://www.cg.tuwien.ac.at/research/publications/2007/weidlich_2007_almfs/) by Andrea Weidlich and Alexander Wilkie from 2007 (which I refer to as the original model or the WWL model).

*This chapter contains: an overall explanation of my model (philosophy, differences from WWL, ...), its evaluation and sampling and, at the end, an analysis of the model behaviour...*

## Overall Model Description

At first sight, my layered model doesn't differ very much from the WWL model, but it contains *incremental/important* improvements, which solve *some/critical?/various/several* problems of the previous model and offers a rigorous formulation of the model consistent with radiometry*/(BSDF)* framework and Monte Carlo theory. Namely, the changes are:

- For refraction it uses **geometric normal** instead of micro-facet's one for better approximation.
- Adds the missing compensation of **solid angle (de-)compression** effects in both evaluation and sampling -- this solves the energy conservation problem and incorrect sampling PDF leading to a biased Monte-Carlo estimator.
- **Optimizes the sampling routine**.

Formally, I denote the layers [BSDFs](https://en.wikipedia.org/wiki/Bidirectional_scattering_distribution_function) $f_{s1}$ and $f_{s2}$ (1 -- outer, 2 -- inner) consistently with the original paper. The resulting BSDF $f_{s}$ is then a sum of two components representing the contributions of the respective layers $f_{s1}^{\ast}$ and $f_{s2}^{\ast}$:

$$
f_{s} = f_{s1}^{\ast} + f_{s2}^{\ast}
$$

## Geometrical normal (?)

The new model still assumes single-point simplifications of both refraction and sampling from the original model, as well as non-scattering medium between layers. However, unlike the WWL model, it doesn't make any assumption about the relative size of micro-facets and layers, which justifies using a single micro-facet during the whole evaluation and sampling process. The difference of my model is that for computing refraction directions it uses the *(main)* **geometrical normal** rather than the reflection-defined micro-facet's one.

It is *worth noting/important to understand* that the new model still estimates the whole sub-surface light transport with just one light path and a single scattering event, but the used refraction directions define a path, which is more likely the one through which it flows *the peak amount of energy*. This makes it a better representative/estimate of the actual total (single-scattered) energy transferred via all refracted paths. *(Comparisons needed!)*

It is also important to keep in mind that both models neglect the energy which is reflected from the outer layer back into the medium (multiple scattering). *WWL model uses some kind of compensation!...*

[?Image? refraction direction, peak energy path, neglected energy]

## Evaluation

...

Outer layer

- Almost trivial...

Inner layer

- Needs Fresnel attenuations and solid angle (de)compression.
- Formula derivation: Oh yeah! :-)

### Solid angle (de)compression (?) compensation

...

## Sampling

Idea...

Outer layer:

- Almost trivial...

Inner layer:

- Sampling almost trivial (just refraction).
- PDF: "Too dark" problem -- missing solid angle (de)compression

Both layers together:

Optimizations: Weighting -- contribution estimation: 

- Fresnel split approximation
- Medium attenuation approximation
- ...

## Model Analysis

Certain configuration...

...comparison of the behaviour with path-traced reference renderings...

### Reference images

Story...

### No medium attenuation

- energy conservation, fitting the reference, ...

### Later: With medium attenuation

...

### Conclusion

...

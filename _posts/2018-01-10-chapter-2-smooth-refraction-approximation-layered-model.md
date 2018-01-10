---
layout: post
title: "Chapter 2: Smooth Refraction Approximation Layered Model"
comments: true
---

This is the second chapter of the the blog post [A Journey to the World of Layered Surface Materials](a-journey-to-the-world-of-layered-surface-materials.html). Here I explain my approach to the layered surface material as an alternative to the [layered model](https://www.cg.tuwien.ac.at/research/publications/2007/weidlich_2007_almfs/) by Andrea Weidlich and Alexander Wilkie from 2007.

At first sight, my layered model doesn't differ very much from the WW model, but it contains important improvements, which solve some (critical) problems of the previous model. The main *(philosophical)* differences are using the (main) geometrical normal rather than the reflection micro-facet's one for computing refraction directions and a more rigorous formulation of the model. Besides that:

- (Fresnel-based geometric-normal refraction: Proper smooth surface refraction rather than some ad-hoc single-path rough surface refraction. Better approximation? Needs comparison!)
- Adds the missing solid angle (de-)compression compensation in both evaluation and sampling -- solves energy conservation problem and incorrect sampling PDF leading to a biased Monte-Carlo estimator.
- Improves/optimizes the sampling routine
- ...

...comparison of the behaviour with path-traced reference renderings...

## Overall model description

If it's brief, merge with the introduction...or, if it makes sense, move to evaluation.

Explain the motivation behind it? (still single-ray/path approximation, but smooth surface refraction -- better justification --> more physically based approximation?).

- Formula -- a sum of two components: $f_{s} = f_{s1} + f_{s2}$
  -  Name the basic (micro-facet) models (parameters) somehow?

## Evaluation

...

### Outer layer

Almost trivial...

### Inner layer

Needs Fresnel attenuations and solid angle (de)compression.

Formula derivation: Oh yeah! :-)

#### Solid angle (de)compression (?) compensation

...

## Sampling

- Idea...

### Outer layer

- Almost trivial...

### Inner layer

- Sampling almost trivial (just refraction).
- PDF: "Too dark" problem -- missing solid angle (de)compression

### Both layers together

Optimizations: Weighting -- contribution estimation: 

- Fresnel split approximation
- Medium attenuation approximation
- ...

## Model analysis

Certain configuration...

### Reference images

...

### No medium attenuation

- energy conservation, fitting the reference, ...

### Later: With medium attenuation

...

## Conclusion

...

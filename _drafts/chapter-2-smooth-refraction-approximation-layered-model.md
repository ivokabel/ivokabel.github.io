---
layout: post
title: Chapter 2: Smooth Refraction Approximation Layered Model
comments: true
---

This is the second chapter of the the blog post [A Journey to the World of Layered Surface Materials](a-journey-to-the-world-of-layered-surface-materials.html).

My Approach...

At first sight, my layered model doesn't differ very much from the WW model, but it contains important improvements, which solve some (critical) problems of the previous model. The main (philosophical) differences lie in using the (main) geometrical normal rather than the reflection micro-facet's one for computing refraction directions and in a more rigorous formulation of the model. Besides that:

- (Fresnel-based geometric-normal refraction: Proper smooth surface refraction rather than some ad-hoc single-path rough surface refraction. Better approximation? Needs comparison!)
- Adds the missing solid angle (de-)compression compensation in both evaluation and sampling,
- Improves/optimizes the sampling routine
- ...

Comparison of the behaviour with path-traced reference renderings...

### Overall model description

If it's brief, put to the chapter introduction...

### Evaluation

...

#### Outer layer

Almost trivial...

#### Inner layer

Needs Fresnel attenuations and solid angle (de)compression.

Formula derivation: Oh yeah! :-)

### Sampling

- Idea...

#### Outer layer

- Almost trivial...

#### Inner layer

- Sampling almost trivial (just refraction).
- PDF: "Too dark" problem -- missing solid angle (de)compression

### Both layers together

Optimizations: Weighting -- contribution estimation: 

- Fresnel split approximation
- Medium attenuation approximation
- ...

### Model analysis

Certain configuration...

#### Reference images

...

#### No medium attenuation

- energy conservation, fitting the reference, ...

#### Later: With medium attenuation

...

### Conclusion

...
---
layout: post
title: "Chapter 2: Smooth Refraction Approximation Layered Model (SRAL)"
comments: true
---

This is the second chapter of the the blog post [A Journey to the World of Layered Surface Materials](a-journey-to-the-world-of-layered-surface-materials.html). Here I present my approach to the layered surface material as an alternative to the [layered model](https://www.cg.tuwien.ac.at/research/publications/2007/weidlich_2007_almfs/) by Andrea Weidlich and Alexander Wilkie from 2007 (which I refer to as the original model or the WWL model).

*This chapter contains: an overall explanation of my model (philosophy, differences from WWL, ...), its evaluation and sampling and, at the end, an analysis of the model behaviour...*

## Overall Model Description

At first sight, my layered model doesn't differ very much from the WWL model, but it contains *incremental/important* improvements, which solve *some/critical?/various/several* problems of the previous model and offers a rigorous formulation of the model consistent with radiometry*/(BSDF)* framework and Monte Carlo theory. Namely, the improvements are:

- Using **geometric normal** for refraction instead of micro-facet's one.
- Adding the missing compensation of **solid angle (de-)compression** effects in both evaluation and sampling -- solves the energy conservation problem and incorrect sampling PDF leading to a biased Monte-Carlo estimator.
- The **sampling routine optimization**.

Formally, I denote the outer and inner layer [BSDFs](https://en.wikipedia.org/wiki/Bidirectional_scattering_distribution_function) (parameters of the whole model) $f_{s1}^{\ast}\left(x,\omega_{i}\rightarrow\omega_{o}\right)$ and $f_{s2}^{\ast}\left(x,\omega_{i}\rightarrow\omega_{o}\right)$ respectively, where $x$ is the surface point at which the model is evaluated, $\omega_{i}$ is the incident light direction, and $\omega_{o}$ is the outgoing light direction. In the following text I will omit the surface point $x$ from the notation and sometimes also the direction parameters for clarity. Note that in the original paper those BSDFs were denoted without asterisk and with $r$ subscript to *signal* that they are just reflective [BRDFs](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function) rather than BSDFs: $f_{r1}$ and $f_{r2}$.

The resulting model BSDF $f_{s}$ can be understood as a sum of two components representing the *contributions* of the respective layers $f_{s1}$ and $f_{s2}$:
$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) + f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

## Refraction Through the Geometrical Normal

The new model still assumes single-point simplifications of both refraction and sampling from the original model, as well as non-scattering medium between layers. However, unlike the WWL model, it doesn't make any assumption about the relative size of micro-facets and layers, which justified using a single micro-facet during the whole evaluation and sampling process. The difference of my model is that for computing refraction directions it uses the *(main)* geometrical normal rather than the reflection-defined micro-facet's one.

*[Image?]*

It is *worth noting/important to understand* that the new model still estimates the whole sub-surface light transport with just one light path and a single scattering event, but the used refraction directions define a path, which is more likely the one through which *the peak amount of energy flows*. This makes it a better representative/estimate of the actual total (single-scattered) energy transferred via all refracted paths. *(Comparisons needed!)*

It is also important to keep in mind that both models neglect the energy which is reflected from the outer layer back into the medium (*multiple scattering*). *WWL model uses some kind of compensation!...*

*[?Image #RefrGeomNorm? refraction direction, peak energy path, neglected energy]*

*Why should my approximation work?... The higher the specularity of the outer surface is the better the approximation behaves. For rougher surfaces the light is spreads over a wider interval of directions --> the Fresnel behaves differently (especially at grazing angles), the inner layer is lit from wider set of angles --> approximation starts to fail... But, at least the amount of transmitted energy should roughly approximate the actual transmission.*

## Evaluation

As mentioned earlier, the model can be understood as a sum of contributions of both layers:

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) + f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

### Outer layer

Since the (direct) contribution of the outer layer is just its reflection component $f_{s1}^{\ast}$, which is not affected by other parts of the model, it can be evaluated directly without any modification, therefore:

$$
f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}^{\ast}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

### Inner layer

For the inner layer, the thing are considerably more complicated because the light which reaches it undergoes refraction when passing the outer layer, gets attenuated by the medium between the layers and gets refracted again when passing the model through the outer layer for the second time. Let's have a look at how do these mechanisms affect the inner layer contribution one after another.

#### Refraction

Let's start with an example inner layer (orange Lambert) without any modifications (light: point, area, diffuse), which will undergo the aforementioned modifications.

*[Images: Orange Lambert layer without any modifications (light: point, area, diffuse)]*

First of all, the inner layer deals with refracted directions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$ instead of the directions at the outer layer $\omega_{i}$ and $\omega_{o}$ as can be seen in the *image [#RefrGeomNorm]*. After applying just these modified directions, the example inner layer contribution will look like this:

*[Images: Orange Lambert layer with refracted directions (light: point, area, diffuse)]*

Second, the light passing through the smooth interface gets attenuated according to the Fresnel equations, which attenuates the light at grazing angles.

*[Images: Orange Lambert layer with Fresnel attenuation (light: point, area, diffuse)]*

#### Medium attenuation

After refraction, we will add the effect of medium attenuation. In a non-scattering medium, the attenuation $a$ can be well modeled with the [Beer-Lambert-Bouguer law](https://en.wikipedia.org/wiki/Beer%E2%80%93Lambert_law), exactly as it was done in the original paper:

$$
a=e^{-\alpha\left(d\cdot\left(\frac{1}{\cos\theta_{i}^{\prime}}+\frac{1}{\cos\theta_{o}^{\prime}}\right)\right)}
$$

where $\alpha$ is the medium attenuation coefficient, $d$ is the thickness of the layer, and $\theta_{i}^{\prime}$ and $\theta_{o}^{\prime}$ are inclinations of the respective refraction directions.

To demonstrate the effect of medium attenuation I used a purely white diffuse Lambert surface model for the inner layer and a bluish medium with different inter-layer thicknesses. You can nicely see the effect of darkening and colour saturation when the light passed through different amounts of medium:

*[Images: Ideally white (albedo 100%) Lambert layer under blue medium (thicknesses: large to none; light: point, area, diffuse)]*

You can, as well, see that there is something wrong with the model when the medium attenuation is week. Although we used a physically-plausible energy-conserving Lambert model for the inner layer, the model is sometimes much lighter than one would expect it to be. There obviously is an energy conservation problem, which took me some time to decipher. Long story short: the problem is caused by the compression and decompression of light when crossing an interface between media with different indices of refraction.

#### Solid angle (de)compression (?) compensation

In this sub-section I will explain how to compensate the light (de-)compression to obtain an energy-conserving layered model. To do that, I will have to dig deeper into the theory of BDSFs. If you are not nerdy enough for that, just skip to the final sub-section, which summarizes the whole inner model formula.

...

TODO: Energy conservation & Solid angle (de)compression: formula derivation, oh yeah! :-)

#### The complete formula

...

## Sampling

Main idea: sum BSDF --> weighted average of two sampling strategies ($p^{\ast}_1$ and $p^{\ast}_2$)

**Outer layer:**

- Trivial: $p^{\ast}_1\left(\omega_{i}, \omega_{o}\right) = p_1\left(\omega_{i}, \omega_{o}\right)$

**Inner layer:**

- Sampling almost trivial (just refraction).
- PDF: "Too dark" problem -- missing solid angle (de)compression

**Both layers together:**

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

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

Formally, I denote the outer and inner layer [BSDFs](https://en.wikipedia.org/wiki/Bidirectional_scattering_distribution_function) (parameters of the whole model) $f_{s1}^{\ast}\left(x,\omega_{i}\rightarrow\omega_{o}\right)$ and $f_{s2}^{\ast}\left(x,\omega_{i}\rightarrow\omega_{o}\right)$ respectively, where $x$ is the surface point at which the model is evaluated, $\omega_{i}$ is the incident light direction, and $\omega_{o}$ is the outgoing light direction. For the sake of clarity, I will, from time to time, omit some of the parameters. Note that in the original paper those BSDFs were denoted without asterisk and with $r$ subscript to *signal* that they are just reflective [BRDFs](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function) rather than BSDFs: $f_{r1}$ and $f_{r2}$.

The resulting model BSDF $f_{s}$ can be vaguely understood as a sum of two *components/sub-BSDFs* representing the *contributions* of the respective layers $f_{s1}$ and $f_{s2}$:

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) + f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

I sometimes call the functions $f_{s1}$ and $f_{s2}$ the **contribution BSDFs** or just **contributions**.

## Refraction Through the Geometrical Normal

The new model still assumes single-point simplifications of both refraction and sampling from the original model, as well as non-scattering medium between layers. However, unlike the WWL model, it doesn't make any assumption about the relative size of micro-facets and layers, which justified using a single micro-facet during the whole evaluation and sampling process. The difference of my model is that for computing refraction directions it uses the *(main)* geometrical normal rather than the reflection-defined micro-facet's one.

*[Image?]*

It is *worth noting/important to understand* that the new model still estimates the whole sub-surface light transport with just one light path and a single scattering event, but the used refraction directions define a path, which is more likely the one through which *the peak amount of energy flows*. This makes it a better representative/estimate of the actual total (single-scattered) energy transferred via all refracted paths. *(Comparisons needed!)*

It is also important to keep in mind that both models neglect the energy which is reflected from the outer layer back into the medium (*multiple scattering*). *WWL model uses some kind of compensation!...*

*[?Image #RefrGeomNorm? refraction direction, peak energy path, neglected energy]*

*Why should my approximation work?... The higher the specularity of the outer surface is the better the approximation behaves. For rougher surfaces the light is spreads over a wider interval of directions --> the Fresnel behaves differently (especially at grazing angles), the inner layer is lit from wider set of angles --> approximation starts to fail... But, at least the amount of transmitted energy should roughly approximate the actual transmission.*

## Evaluation

As mentioned earlier, the model can be understood as a sum of two contribution BSDFs:

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) + f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

### Outer layer

The contribution BSDF of the outer layer is trivial. It can be split into the reflection and refraction part. The light which gets refracted through the first layer into the model is accounted for in the inner layer component so we ignore it here. Since the reflected light is unaffected by the rest of the model and the whole model takes only reflection directions into account, it can be evaluated directly by evaluating the outer layer BSDF $f_{s1}^{\ast}$ *without any modification*. Therefore:

$$
f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}^{\ast}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

### Inner layer

For the inner layer, the things are considerably more complicated because the light which reaches it undergoes refraction when passing the outer layer, gets attenuated by the medium between the layers and gets refracted again when passing the model through the outer layer for the second time. Let's have a look at how do these mechanisms affect the inner layer contribution one after another.

#### Refraction

Let's start with an inner layer (orange Lambert) without any modifications, which will undergo the aforementioned modifications later on. BSDF is the original one:

$$
f_{s2}^{\ast}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

*[Images: Orange Lambert layer without any modifications (light: point, area, diffuse)]*

The inner layer, in fact, deals with refracted directions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$ instead of the directions at the outer layer $\omega_{i}$ and $\omega_{o}$ as can be seen in the *image [#RefrGeomNorm]*. After applying  the modified directions, the inner layer contribution will look like this:

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right)
$$

*[Images: Orange Lambert layer with refracted directions (light: point, area, diffuse)]*

The light passing through the smooth interface gets attenuated according to the Fresnel equations, which attenuates the light at grazing angles:

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) \cdot T_{12} \cdot T_{21}
$$

Where $T_{12} = T\left(\theta_{i}^{\prime}\right)$ and $T_{21} = T\left(\theta_{o}^{\prime}\right)$ are the Fresnel transmission coefficients.

*[Images: Orange Lambert layer with Fresnel attenuation (light: point, area, diffuse)]*

#### Medium attenuation

After refraction, we will add the effect of medium attenuation. In a non-scattering medium, the attenuation $a$ can be well modeled with the [Beer-Lambert-Bouguer law](https://en.wikipedia.org/wiki/Beer%E2%80%93Lambert_law), exactly as it was done in the original paper:

$$
a=e^{-\alpha\left(d\cdot\left(\frac{1}{\cos\theta_{i}^{\prime}}+\frac{1}{\cos\theta_{o}^{\prime}}\right)\right)}
$$

where $\alpha$ is the medium attenuation coefficient, $d$ is the thickness of the layer, and $\theta_{i}^{\prime}$ and $\theta_{o}^{\prime}$ are inclinations of the respective refraction directions. The BSDF is now:

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) \cdot T_{12} \cdot T_{21} \cdot a
$$

To demonstrate the effect of medium attenuation I used a purely white diffuse Lambert surface model for the inner layer and a blue medium with decreasing inter-layer thicknesses. You can nicely see the effect of darkening and colour saturation when the light passed through different amounts of medium:

*[Images: Ideally white (albedo 100%) Lambert layer under blue medium (thicknesses: large to none; light: point, area, diffuse)]*

You can, as well, see that there is something wrong with the model when the medium attenuation is week. Although we used a physically-plausible energy-conserving Lambert model for the inner layer, the model is sometimes much lighter than one would expect it to be. In the furnace test (the constant white light configuration) we can clearly see that it reflects more energy than it receives from the environment which is a sign of an energy conservation problem. I spent a non-trivial amount of time to crack this problem, but I won in the end and I gained some important computer graphics knowledge on this way. Long story short: the problem is caused by the compression and decompression of light when crossing an interface between media with different indices of refraction.

#### Solid angle (de-)compression compensation ?

Alternative title: "BSDF under smooth refractive interface"

In this section I will explain how to *(compensate the light (de-)compression to)* obtain a correct energy-conserving form of a BSDF under a smooth refractive interface with a single scattering event. To do that, I will have to dig deeper into the theory of BDSFs. If you are not feeling nerdy enough, just skip to the final sub-section, which summarizes the whole inner model formula including the derived compensation.

**TODO:**

- We want the correct form of BSDF, we need to:
  - Re-formulate *(from definition)* the inner layer BSDF as a function of (non-refracted) incoming and outgoing directions $\omega_{i}$ and $\omega_{o}$ of the whole model rather than *as a function* of refracted directions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$
    - [image: geometry]
  - We want to see how the layer behaves in the eyes of the path-tracer which doesn't care about the inside mechanisms of the model (refractions and attenuations) -- *it just wants a properly defined BSDF which it evaluates*.
- To understand the problem properly we need to go down to the fundamental concepts/theory upon which the model is built:
  - Assume knowledge of underlying frameworks: solid angle, ...
  - Radiometric quantities (assume knowledge or do a very light introduction -- using Radon-Nykodym derivation; measure-theoretic radiometry?):
    - Radiant energy: $Q \quad \left[J\right]$
    - Radiant flux/power: $\Phi = \frac{\mathrm{d}Q}{\mathrm{d}t} \quad \left[W=Js^{-1}\right]$
    - Irradiance: $E=\frac{\mathrm{d}\Phi}{\mathrm{d}S} \quad \left[Wm^{-2}\right]$
    - Radiance: $L = \frac{\mathrm{d}^{2}\Phi}{\cos\theta\cdot \mathrm{d}S\cdot \mathrm{d}\omega} \quad \left[Wm^{-2}sr^{-1}\right]$
      - Projected solid angle measure: $\mathrm{d}\sigma^{\bot}$
  - Bi-directional scattering distribution function (BSDF):
    - Geometry explanation with image (from Veach for example)
    - $f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = \frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{\mathrm{d}E\left(\omega_{i}\right)} = \frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{L_{i}\left(\omega_{i}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}\right)} = \frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{L_{i}\left(\omega_{i}\right)\cos\theta_{i}\mathrm{d}\omega_{i}} \quad \left[sr^{-1}\right]$
    - How to understand it...
  - Prerequisite formulae:
    - [geometry image]
    - 1, 2 -- Radiances: $L_{o}\left(\omega_{o}\right)$, $L_{i}\left(\omega_{i}\right)$
    - 3 -- Projected solid angle measure: $\mathrm{d}\sigma^{\bot} \left({\omega}_t\right)$
      - Use Veach, eq. 5.4, p. 143: relationship between $i$ and $o$ measures
    - 4 -- Equality: $\mathrm{d}\sigma^{\bot} \left({\omega}_i^\prime\right) = \mathrm{d}\sigma^{\bot} \left({\omega}_t\right)$
  - Refracted formula (using the prerequisites)
    - $f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = \frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{L_{i}\left(\omega_{i}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}\right)}$
    - ...single-point adjustment

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

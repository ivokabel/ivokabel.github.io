---
layout: post
title: "Chapter 2: Smooth Refraction Approximation Layered Model (SRAL)"
comments: true
---

This is the second chapter of the the blog post [A Journey to the World of Layered Surface Materials](a-journey-to-the-world-of-layered-surface-materials.html). Here I present my approach to the layered surface material as an alternative to the [layered model](https://www.cg.tuwien.ac.at/research/publications/2007/weidlich_2007_almfs/) by Andrea Weidlich and Alexander Wilkie from 2007 (which I refer to as the original model or the WWL model).

*This chapter contains: an overall explanation of my model (philosophy, differences from WWL, ...), its evaluation and sampling and an analysis of the model behaviour...*

## Overall Model Description

At first sight, my layered model doesn't differ very much from the WWL model, but it contains *incremental/important* improvements, which solve *some/critical?/various/several* problems of the previous model and offers a rigorous formulation of the model consistent with radiometry*/(BSDF)* framework and Monte Carlo theory. Namely, the improvements are:

- Using **geometric normal** for refraction instead of micro-facet's one.
- Adding the missing compensation of **solid angle (de-)compression** effects in both evaluation and sampling -- solves the energy conservation problem and incorrect sampling PDF leading to a biased Monte-Carlo estimator.
- The **sampling routine optimization**.

Formally, I denote the stand-alone outer and inner layer [BSDFs](https://en.wikipedia.org/wiki/Bidirectional_scattering_distribution_function) (parameters of the whole model) $f_{s1}^{\ast}\left(x,\omega_{i}\rightarrow\omega_{o}\right)$ and $f_{s2}^{\ast}\left(x,\omega_{i}\rightarrow\omega_{o}\right)$ respectively, where $x$ is the surface point at which the model is evaluated, $\omega_{i}$ is the incident light direction, and $\omega_{o}$ is the outgoing light direction. Any direction $\omega$ can be expressed as a pair of angles $\left(\phi,\theta\right)$, where $\phi$ is azimuth and $\theta$ is inclination, i.e. angle between the direction and the surface normal. For the sake of clarity, I will, from time to time, omit some of the parameters. Note that in the original paper those BSDFs were denoted without asterisk and with $r$ subscript to *signal* that they are just reflective [BRDFs](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function) rather than BSDFs: $f_{r1}$ and $f_{r2}$.

The resulting model BSDF $f_{s}$ can be vaguely understood as a sum of two *components/sub-BSDFs* representing the *contributions* of the respective layers $f_{s1}$ and $f_{s2}$:

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) + f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

I sometimes call the functions $f_{s1}$ and $f_{s2}$ the **contribution BSDFs** or just **contributions**.

The outer layer is expected to be an interface between two media with refractive indices: $\eta_0$ above the outer layer and $\eta_1$ between the outer and inner layer. There is no assumption on the medium below the inner layer.

## Refraction Through the Geometrical Normal

The new model still assumes single-point simplifications of both refraction and sampling from the original model, as well as non-scattering medium between layers. However, unlike the WWL model, it doesn't make any assumption about the relative size of micro-facets and layers, which justified using a single micro-facet during the whole evaluation and sampling process. The difference of my model is that for computing refraction directions it uses the *(main)* geometrical normal rather than the reflection-defined micro-facet's one.

*[Image?]*

It is *worth noting/important to understand* that the new model still estimates the whole sub-surface light transport with just one light path and a single scattering event, but the used refraction directions define a path, which is more likely the one through which *the peak amount of energy flows*. This makes it a better representative/estimate of the actual total (single-scattered) energy transferred via all refracted paths. *(Comparisons needed!)*

It is also important to keep in mind that both models neglect the energy which is reflected from the outer layer back into the medium (*multiple scattering*). *WWL model uses some kind of compensation!...*

*[?Image #RefrGeomNorm refraction direction, peak energy path, neglected energy]*

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

#### Naïve refraction

Let's start with an inner layer (orange Lambert) without any modifications, which will undergo the aforementioned modifications later on. BSDF is the original one:

$$
f_{s2}^{\ast}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

*[Images: Orange Lambert layer without any modifications (light: point, area, diffuse)]*

The inner layer, in fact, *deals* with refracted directions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$ instead of the directions at the outer layer $\omega_{i}$ and $\omega_{o}$ as can be seen in the *image [#RefrGeomNorm]*. After applying  the modified directions, the inner layer contribution will look like this:

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right)
$$

*[Images: Orange Lambert layer with refracted directions (light: point, area, diffuse)]*

The light passing through the smooth interface gets attenuated according to the Fresnel equations, which attenuates the light at grazing angles:

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{i}\right) T\left(\theta_{o}\right)
$$

Where $T\left(\theta_{i}\right)$ and $T\left(\theta_{o}\right)$ are the Fresnel transmission coefficients.

*[Images: Orange Lambert layer with Fresnel attenuation (light: point, area, diffuse)]*

#### Medium attenuation

After refraction, we will add the effect of medium attenuation. In a non-scattering medium, the attenuation $a$ can be well modeled with the [Beer-Lambert-Bouguer law](https://en.wikipedia.org/wiki/Beer%E2%80%93Lambert_law), exactly as it was done in the original paper:

$$
a=e^{-\alpha\left(d\cdot\left(\frac{1}{\cos\theta_{i}^{\prime}}+\frac{1}{\cos\theta_{o}^{\prime}}\right)\right)}
$$

where $\alpha$ is the medium attenuation coefficient, $d$ is the thickness of the layer, and $\theta_{i}^{\prime}$ and $\theta_{o}^{\prime}$ are inclinations of the respective refraction directions. The BSDF is now

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{i}\right) T\left(\theta_{o}\right) a
$$

To demonstrate the effect of medium attenuation I used a purely white diffuse Lambert surface model for the inner layer and a blue medium with decreasing inter-layer thicknesses. You can nicely see the effect of darkening and colour saturation when the light passed through different amounts of medium:

*[Images: Ideally white (albedo 100%) Lambert layer under blue medium (thicknesses: large to none; light: point, area, diffuse)]*

*?MOVE TO THE BEGINNING OF THE NEXT SUB-SECTION?:* You can, as well, see that there is something wrong with the model when the medium attenuation is week. Although we used a physically-plausible energy-conserving Lambert model for the inner layer, the model is sometimes much lighter than one would expect it to be. In the furnace test (the constant white light configuration) we can clearly see that it reflects more energy than it receives from the environment which is a sign of an energy conservation problem. I spent a non-trivial amount of time to crack this problem, but I won in the end and I gained some important computer graphics knowledge on this way. Long story short: the problem is caused by the compression and decompression of light when crossing an interface between media with different indices of refraction.

#### Proper refraction

*Alternative titles:*

- *"Solid angle (de-)compression compensation"*
- *"BSDF under smooth refractive interface"*

In this section I will explain how to obtain a correct energy-conserving BSDF under a smooth refractive interface with a single scattering event. To do that, I will have to dig deeper into the theory of BDSFs, so if you are not feeling nerdy enough, just skip to the "The complete inner layer formula" sub-section, which summarizes the whole inner model formula including the compensation derived in this chapter.

One way of obtaining the correct form of a BSDF under a smooth refractive interface its to re-formulate the inner layer BSDF as a function of (non-refracted) incoming and outgoing directions of the whole model $\omega_{i}$ and $\omega_{o}$ rather than *as a function* of refracted directions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$:

*[image: #RefrBsdfGeom?]*


In other words: we want to see how the layer behaves to the rendered (e.g path-tracer) which doesn't know about the internal mechanics of the model (refractions, reflections, attenuations, etc.) and regards the evaluated BSDF as a black box.

##### Radiometric quantities

To handle the problem properly we need to *go down/grasp it through* to the fundamental *concepts/theory* upon which the model is built. *My explanation assumes knowledge of mathematical concepts like derivative, measure, solid angle, etc.*

First, we need to define a few radiometric quantities, which the BSDF theory builds upon. There are several ways of doing that, but I will follow the measure-theoretic radiometry way, which defines its quantities as ratios of measures and is explained in more detail for example in the third chapter of the [Eric Veach's dissertation thesis](http://graphics.stanford.edu/papers/veach_thesis/). This approach uses the so-called Radon-Nikodym derivative, which, expresses density of one measure with respect to another one. For measures $Q$ and $\rho$ the corresponding Radon-Nikodym derivative is denoted $\frac{\mathrm{d}Q}{\mathrm{d}\rho}$ (similarly to the "normal" derivative). In the measure-theoretic radiometry case, we usually express the density of electro-magnetic energy measure (amount of photons, loosely speaking) with respect to some space and/or time measure.

*...cohesion: energy measure == energy...*

The most basic/fundamental radiometric quantity is *radiant energy*:

$$
Q \quad \left[J\right]
$$

measured in Joules, which can be understood as the amount of energy of photons in a given time set and space. For the density of energy per unit of time, there is *radiant flux* or *radiant power*:

$$
\Phi\left(t\right) = \frac{\mathrm{d}Q\left(t\right)}{\mathrm{d}t} \quad \left[Js^{-1}=W\right]
$$

where $t$ is the time parameter and also the time measure. In computer graphics we are very often interested in "amount of light" which arrives at, gets scattered or is emitted from a certain surface point $x$. This is *expressed* with density of radiant power per unit of surface area and is called *irradiance* for incident radiation and *radiant exitance* or *radiosity* for scattered and emitted radiation:

$$
E\left(x\right) = \frac{\mathrm{d}\Phi\left(x\right)}{\mathrm{d}A\left(x\right)} \quad \left[Wm^{-2}\right]
$$

where $A$ is the surface area measure. While irradiance measures energy density from all directions, we often need to measure the amount of light entering or leaving at a point of surface in a single direction. This is what quantity called *radiance* is for. Formally, it is expressed as the density of radiant power per unit. There are several equivalent formulations for radiance, but the most practical one for our purposes is

$$
L\left(x,\omega\right) = \frac{\mathrm{d}^{2}\Phi\left(x,\omega\right)}{\mathrm{d}A\left(x\right) \mathrm{d}\sigma^{\bot}\left(\omega\right)} \quad \left[Wm^{-2}sr^{-1}\right]
$$

Note that this definition uses *projected solid angle measure* $\sigma^{\bot}$ rather than "normal" solid angle measure $\sigma$ in order to make the quantity independent from relative position of the light direction and the the surface normal. Basically, it just adds a simple cosine factor $\cos\left(\theta\right)$ to the solid angle measure $\sigma$, which, in this case, compensates the loss of area density when direction $\omega$ *gets further* from the normal.

##### BSDF definition

Now that we have defined the needed radiometry quantities, we can finally define the *bidirectional scattering density function* (BSDF), a formal description of the light-scattering properties of a surface point. It expresses how the light outgoing from a point $x$ on a surface in a particular direction $\omega_o$ is dependent on the light incoming to the point from a particular direction $\omega_i$:

*[image: BSDF geometry from Veach, fig. 3.1]*

*The surface point $x$ will be omitted from the notation in the following text for clarity.* We denote $L_o\left(\omega_o\right)$ the radiance leaving the point $x$ in the direction $\omega_o$. In general, it is dependent on the radiance incoming to $x$ from all possible directions (from both above or below the surface). Let's now consider the radiance $L_i\left(\omega_i\right)$ incoming to $x$ through an infinitesimal cone $\mathrm{d}\omega_{i}$ around the direction $\omega_i$ with solid angle size $\mathrm{d}\sigma\left(\omega_{i}\right)$. This generates irradiance denoted $\mathrm{d}E\left(\omega_{i}\right)$ on the surface, which can be computed as

$$
\mathrm{d}E\left(\omega{i}\right) = L{i}\left(\omega{i}\right)\mathrm{d}\sigma^{\bot}\left(\omega{i}\right)
$$

The incident light can be partially absorbed by the surface and partially scattered in all directions. Let's denote $\mathrm{d}L_{o}\left(\omega_{o}\right)$ the out-scattered contribution of $\mathrm{d}E\left(\omega_{i}\right)$ to $L_{o}\left(\omega_{o}\right)$. It can be shown experimentally that amount of outgoing radiance $\mathrm{d}L_{o}\left(\omega_{o}\right)$ is proportional to the incoming irradiance $\mathrm{d}E\left(\omega_{i}\right)$ and the BSDF is then defined simply as the ratio between the two:

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = \frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{\mathrm{d}E\left(\omega_{i}\right)} = \frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{L_{i}\left(\omega_{i}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}\right)} \quad \left[sr^{-1}\right]
$$

The radiometry and BSDF explanation is heavily inspired by the excellent [Eric Veach's PhD thesis](http://graphics.stanford.edu/papers/veach_thesis/) (it's more of a book than a PhD dissertation ;-)), which I recommend to read for more detailed explanation and also to everyone who is seriously interested in computer graphics, especially in realistic 3D rendering.

##### BSDF under a smooth refractive interface

*[image: #RefrBsdfGeom again?]*

Using the general definition of BSDF, we'll now derive the proper formula for a layer under a smooth Fresnel surface. For that we will need to prepare 4 prerequisite formulae, which we will then use to build the final BSDF formula

First, let's see what happens to the incoming radiance when it passes a smooth interface between two media with different indices of refraction

$$
L_{i}\left(\omega_{i}^{\prime}\right) = L_{i}\left(\omega_{i}\right) \frac{\eta_1^2}{\eta_0^2} T\left(\theta_i\right)
$$

where $\frac{\eta_1^2}{\eta_0^2}$ is the radiance scaling due to solid angle compression (see the Veach's thesis, formula 5.2, p. 141) and $T\left(\theta_i\right)$ is the Fresnel transmission coefficient (how much light is refracted rather than reflected). For our purposes, it will be handy to rewrite the equation a little bit

$$
L_{i}\left(\omega_{i}\right) = L_{i}\left(\omega_{i}^{\prime}\right)  \frac{\eta_0^2}{\eta_1^2} T\left(\theta_i\right)^{-1}
$$

Similarly, we can express what happens to the outgoing radiance

$$
L_{o}\left(\omega_{o}\right) = L_{o}\left(\omega_{o}^{\prime}\right) \frac{\eta_0^2}{\eta_1^2} T\left(\theta_o\right)
$$

Now, the factor $\frac{\eta_0^2}{\eta_1^2}$ represents the amount of solid angle de-compression. Note that we simplified the notation by using  $T\left(\theta_o\right)$ instead of $T\left(-\theta_o^{\prime}\right)$ because the two are equal.

*[image: light cones geometry?] ...probably not necessary*

The third piece of our puzzle is how the projected solid angle $\mathrm{d}\sigma^{\bot}$ of a light cone changes when refracted through smooth interface. The relation between the incident and refracted light cone can be found for example in the Veach's thesis in the equation 5.4 at page 143 and, after a minor rewrite, we will get

$$
\mathrm{d}\sigma^{\bot} \left(\omega_i\right) = \frac{\eta_1^2}{\eta_0^2} \mathrm{d}\sigma^{\bot} \left(\omega_t\right)
$$

where $\omega_t$ is the refracted direction of $\omega_i$. SPOILER ALERT: this relationship contains the missing component of our new BSDF formula.

The fourth and the last prerequisite is a little technical and I will not go into detail of this one. The point is, but it can be shown that the projected solid angle measure of the refracted cone equals the cone of light reaching the inner layer given that the both layers are parallel *to each other*:

$$
\mathrm{d}\sigma^{\bot} \left({\omega}_i^\prime\right) = \mathrm{d}\sigma^{\bot} \left({\omega}_t\right)
$$

Now we have all the necessary pieces to build the BSDF for the a layer under a smooth Fresnel surface. Let's *put/substitute* them into the plain definition of BSDF, simplify it and separate the BSDF of the stand-alone inner layer BSDF to see what compensation factors have to be applied to get the correct formula:

$$
\begin{eqnarray*}
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right)&=&\frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{L_{i}\left(\omega_{i}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}\right)}\\&=&\frac{\mathrm{d}L_{o}\left(\omega_{o}^{\prime}\right)\frac{\eta_{0}^{2}}{\eta_{1}^{2}}T\left(\theta_{o}\right)}{L_{i}\left(\omega_{i}^{\prime}\right)\frac{\eta_{0}^{2}}{\eta_{1}^{2}}T\left(\theta_{i}\right)^{-1}\frac{\eta_{1}^{2}}{\eta_{0}^{2}}\mathrm{d}\sigma^{\bot}\left(\omega_{i}^{\prime}\right)}\\&=&\frac{\mathrm{d}L_{o}\left(\omega_{o}^{\prime}\right)}{L_{i}\left(\omega_{i}^{\prime}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}^{\prime}\right)}T\left(\theta_{o}\right)T\left(\theta_{i}\right)\frac{\eta_{0}^{2}}{\eta_{1}^{2}}\\&=&f_{s}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{o}\right) T\left(\theta_{i}\right) \frac{\eta_{0}^{2}}{\eta_{1}^{2}}
\end{eqnarray*}
$$

And that's it, folks! You can see that our original approach, in which we just used the refracted direction to evaluate the inner layer model along with attenuating the result with Fresnel transmission coefficients, was almost *correct*. What we were missing was the (relatively trivial) compensation factor, which, however, makes the difference as you can see in the following images:

*[Images: Solid angle compression applied. Ideally white (albedo 100%) Lambert layer under blue medium (thicknesses: large to none; light: point, area, diffuse)]*

It's important to note, that we use single-point simplification here as was used in the original WWL paper -- we assume the incoming and outgoing light to pass through the same point on the outer layer.

*Mention Mitsuba's version? Even they struggled with it and still don't use the correct BSDF. Maybe after I try its approach practically and show the results...*

#### The complete inner layer formula

If we put together all the components which affect the inner layer contribution, we'll get the final formula

$$
f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{i}\right) T\left(\theta_{o}\right) \frac{\eta_{0}^{2}}{\eta_{1}^{2}} a
$$

where $f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right)$ is the stand-alone BSDF of the inner layer, $T\left(\theta_{i}\right)$ and $T\left(\theta_{o}\right)$ are the Fresnel transmission coefficients, $\frac{\eta_{0}^{2}}{\eta_{1}^{2}}$ is the radiance compression coefficient and $a$ is the medium attenuation coefficient.

Just a side note: Since we are implicitly dealing with a rendering system which is not polarization aware we don't have to worry about the order in which the coefficients are multiplied together. In a polarization-aware system, however, multiplication operations (representing light-matter interaction) would have to be dealt with much more care.

## Sampling

Main idea: sum BSDF --> weighted average of two sampling strategies ($p^{\ast}_1$ and $p^{\ast}_2$).

First, have a look at the separate layers sampling procedures, then combine them together.

**Outer layer sampling**

- Almost trivial: $p^{\ast}_1\left(\omega_{i}, \omega_{o}\right) = p_1\left(\omega_{i}, \omega_{o}\right)$
  - Just the upper hemisphere -- reflection contribution

**Inner layer sampling**

- Easy:
  - Use refracted direction $\omega_o^{\prime}$ (obviously)
  - Refract the sampled direction; with brief justification?
- PDF: "Too dark" problem -- missing solid angle (de)compression

**Sampling both layers together**

Optimizations: Weighting = contribution estimation: 

- Fresnel split approximation
- Medium attenuation approximation
- ...

## Model Analysis

- Certain configuration...

- ...comparison of the behaviour with path-traced reference renderings...


### Reference images

Story...

### No medium attenuation

- energy conservation, fitting the reference, ...

### Later: With medium attenuation

...

### Conclusion

...

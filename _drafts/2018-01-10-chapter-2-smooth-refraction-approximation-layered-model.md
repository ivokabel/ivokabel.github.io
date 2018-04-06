---
layout: post
title: "Chapter 2: Smooth Refraction Approximation Layered Model (SRAL)"
comments: true
---

This is the second chapter of the the blog post [A Journey to the World of Layered Surface Materials](a-journey-to-the-world-of-layered-surface-materials.html). Here I present my approach to the layered surface material as an alternative to the [layered model](https://www.cg.tuwien.ac.at/research/publications/2007/weidlich_2007_almfs/) by Andrea Weidlich and Alexander Wilkie from 2007 (which I refer to as *the original model or* the WWL model).

*This chapter contains: an overall explanation of my model (philosophy, differences from WWL, ...), its evaluation and sampling and an analysis of the model behaviour...*

## Overall Model Description

At first sight, my layered model doesn't differ very much from the WWL model, but it contains *important incremental* improvements, which solve *some/critical?/various/several* problems of the previous model and offers a rigorous formulation of the model consistent with radiometry*/(BSDF)* framework and Monte Carlo theory. Namely, the improvements are:

- Using **geometric normal** for refraction instead of micro-facet's one.
- Adding the missing compensation of **solid angle (de-)compression** effects in both evaluation and sampling -- solves the energy conservation problem and incorrect sampling PDF leading to a biased Monte-Carlo estimator.
- *(reworked and more efficient?, ...)* **sampling strategy optimization**.

The model has two main parameters -- two stand-alone layer [BSDFs](https://en.wikipedia.org/wiki/Bidirectional_scattering_distribution_function) (outer and inner), which I denote

$$
\begin{eqnarray*}
&f_{s1}^{\ast}\left(x,\omega_{i}\rightarrow\omega_{o}\right)& \\
&f_{s2}^{\ast}\left(x,\omega_{i}\rightarrow\omega_{o}\right)&
\end{eqnarray*}
$$

where $x$ is the surface point at which the model is evaluated, $\omega_{i}$ is the incident light direction, and $\omega_{o}$ is the outgoing light direction. A direction $\omega$ can be expressed as a pair of angles $\left(\phi,\theta\right)$, where $\phi$ is the azimuth and $\theta$ is the inclination, i.e. angle between the direction and the surface normal. For the sake of clarity, I will, from time to time, omit some of the parameters. Note that in the original paper those BSDFs were denoted without asterisk and with $r$ subscript to *signal* that they are just reflective [BRDFs](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function) rather than BSDFs: $f_{r1}$ and $f_{r2}$.

The resulting model BSDF $f_{s}$ can be vaguely understood as a sum of two *components/sub-BSDFs* representing the *contributions* of the respective layers $f_{s1}$ and $f_{s2}$:

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) + f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

I sometimes call the functions $f_{s1}$ and $f_{s2}$ the **contribution BSDFs** or just **contributions**.

The outer layer is expected to be a microfacet-based interface between two media with refractive indices: $\eta_0$ above the outer layer and $\eta_1$ between the outer and inner layer. There is no assumption on the medium below the inner layer.

### Refraction through the geometrical normal

The new model still assumes single-point simplifications of both evaluation and sampling used in the original model, as well as non-scattering behaviour of the medium between layers. However, unlike the WWL model, it doesn't make any assumption about the relative size of micro-facets and layers, which served as a justification for using the reflection-defined micro-facet normal during the whole evaluation and sampling process. The difference of my model is that for computing refraction directions it uses the *(main)* geometrical normal rather than the micro-facet's one.

*It behaves as if we temporarily shank the distribution of micro-facets into a Dirac pulse -- super highly specular material. The same amount of energy gets refracted, but in modified (degenerated) set of directions...*

*[image #RefrGeom: Refraction through the geometrical normal]*

It is *important to keep in mind* that the new model still estimates the whole sub-surface light transport with just one light path and a single scattering event, but the used refraction directions define a path, which is more likely the one through which *the peak amount of energy flows*. This makes it a better representative/estimate of the actual total (single-scattered) energy transferred via all refracted paths. *(Comparisons needed!)*

It is also important to keep in mind that both models neglect the energy which is reflected from the outer layer back into the medium (*multiple scattering*). *WWL model uses some kind of compensation!...*

*Why should my approximation work?... The higher the specularity of the outer surface is the better the approximation behaves. For rougher surfaces the light is spreads over a wider interval of directions --> the Fresnel behaves differently (especially at grazing angles), the inner layer is lit from wider set of angles --> approximation starts to fail... But, at least the amount of transmitted energy should roughly approximate the actual transmission.*

## Evaluation

As mentioned earlier, the model can be understood as a sum of two contribution BSDFs:

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) + f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

### Outer layer

The contribution BSDF of the outer layer is trivial. It can be split into the reflection and refraction part. The light which gets refracted through the first layer into the model is accounted for in the inner layer component so we ignore it here and the renderer must make sure that it is not evaluated for directions below surface. Since the reflected light is unaffected by the rest of the model and the whole model takes only reflection directions into account, it can be evaluated directly by evaluating the outer layer BSDF $f_{s1}^{\ast}$ *without any modification*. Therefore:

$$
f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}^{\ast}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

### Inner layer

For the inner layer, the things are considerably more complicated because the light which reaches it undergoes refraction when passing the outer layer, gets attenuated by the medium between the layers and gets refracted again when passing the model through the outer layer for the second time. Let's have a look at how do these mechanisms affect the inner layer contribution one after another.

#### Naïve refraction

Let's start with an inner layer (white Lambert) without any modifications, which will undergo the aforementioned modifications later on. BSDF is the original one:

$$
f_{s2}^{\ast}\left(\omega_{i}\rightarrow\omega_{o}\right)
$$

*[images: White Lambert layer without any modifications (3 lights)]*

The inner layer, in fact, *deals* with refracted directions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$ instead of the directions at the outer layer $\omega_{i}$ and $\omega_{o}$ as can be seen in the *image [#RefrGeom]*. After applying  the modified directions, the inner layer contribution will look like this:

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right)
$$

*[images: White Lambert layer with refracted directions (3 lights)]*

*This seemingly doesn't change anything... lambert is constant -- no change in BSDF shape in the positive hemisphere...*

The light passing through the smooth interface gets attenuated according to the Fresnel equations, which attenuates the light at grazing angles:

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{i}\right) T\left(\theta_{o}\right)
$$

Where $T\left(\theta_{i}\right)$ and $T\left(\theta_{o}\right)$ are the Fresnel transmission coefficients.

*[images: White Lambert layer with Fresnel attenuation (3 lights)]*

#### Medium attenuation

After refraction, we will add the effect of medium attenuation. In a non-scattering medium, the attenuation $a$ can be well modeled with the [Beer-Lambert-Bouguer law](https://en.wikipedia.org/wiki/Beer%E2%80%93Lambert_law), exactly as it was done in the original paper:

$$
a\left(\theta_{i}, \theta_{o}\right)  = e^{-\alpha\left(d\cdot\left(\frac{1}{\cos\theta_{i}^{\prime}}+\frac{1}{\cos\theta_{o}^{\prime}}\right)\right)}
$$

where $\alpha$ is the medium attenuation coefficient, $d$ is the thickness of the layer, and $\theta_{i}^{\prime}$ and $\theta_{o}^{\prime}$ are inclinations of the respective refraction directions. The BSDF is now

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{i}\right) T\left(\theta_{o}\right) a\left(\theta_{i}, \theta_{o}\right)
$$

To demonstrate the effect of medium attenuation I used a purely white diffuse Lambert surface model for the inner layer and a brown medium with decreasing inter-layer thicknesses. You can nicely see the effect of darkening and colour saturation when the light passed through different amounts of medium:

*[images: Ideally white (albedo 100%) Lambert layer under brown medium (thicknesses: large to none; 3 lights)]*

#### Proper refraction

*Alternative titles:*

- *"Proper/correct refraction formula/computation/implementation"*
- *"Solid angle (de-)compression compensation"*
- *"BSDF under smooth refractive interface"*

*TODO: Although everything looks nice for the previously shown images... problems with BSDFs other than constant (Lambert) BSDF*

*[images: Missing solid angle problem. Glossy layer under brown medium (thicknesses: large to none; 3 lights)]*

*You can, as well, see that there is something wrong with the model when the medium attenuation is week. Although we used a physically-plausible energy-conserving ~~Lambert~~ model for the inner layer, the model is much lighter than one would expect it to be for some settings. In the furnace test (the constant white light configuration) we can clearly see that it reflects more energy than it receives from the environment which is a sign of an energy conservation problem. I spent a non-trivial amount of time to crack this problem, but I won in the end and I gained some important computer graphics knowledge on this way. Long story short: the problem is caused by the compression and decompression of light when crossing an interface between media with different indices of refraction.*

In this section I will explain how to obtain a correct energy-conserving BSDF under a smooth refractive interface with a single scattering event. To do that, I will have to dig deeper into the theory of BDSFs, so if you are not feeling nerdy enough, just skip to the "Inner layer formula" sub-section, which summarizes the whole inner model formula including the compensation derived in this chapter.

One way of obtaining the correct form of a BSDF under a smooth refractive interface its to re-formulate the inner layer BSDF as a function of (non-refracted) incoming and outgoing directions of the whole model $\omega_{i}$ and $\omega_{o}$ rather than *as a function* of refracted directions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$:

*[image #RefrBsdfGeomPlain: Basic geometry of a BSDF under a smooth refractive interface.]*


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

<p style="text-align: center">
   <img src="../images/BRDF definition geometry.svg" alt="BSDF evaluation process" width="500" /><br/>
   BSDF geometry.
</p>

*The surface point $x$ will be omitted from the notation in the following text for clarity.* We denote $L_o\left(\omega_o\right)$ the radiance leaving the point $x$ in the direction $\omega_o$. In general, it is dependent on the radiance incoming to $x$ from all possible directions (from both above or below the surface). Let's now consider the radiance $L_i\left(\omega_i\right)$ incoming to $x$ through an infinitesimal cone $\mathrm{d}\omega_{i}$ around the direction $\omega_i$ with solid angle size $\mathrm{d}\sigma\left(\omega_{i}\right)$. This generates irradiance denoted $\mathrm{d}E\left(\omega_{i}\right)$ on the surface, which can be computed as

$$
\mathrm{d}E\left(\omega_{i}\right) = L_{i}\left(\omega_{i}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}\right)
$$

The incident light can be partially absorbed by the surface and partially scattered in all directions. Let's denote $\mathrm{d}L_{o}\left(\omega_{o}\right)$ the out-scattered contribution of $\mathrm{d}E\left(\omega_{i}\right)$ to $L_{o}\left(\omega_{o}\right)$. It can be shown experimentally that amount of outgoing radiance $\mathrm{d}L_{o}\left(\omega_{o}\right)$ is proportional to the incoming irradiance $\mathrm{d}E\left(\omega_{i}\right)$ and the BSDF is then defined simply as the ratio between the two:

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = \frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{\mathrm{d}E\left(\omega_{i}\right)} = \frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{L_{i}\left(\omega_{i}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}\right)} \quad \left[sr^{-1}\right]
$$

The radiometry and BSDF explanation is heavily inspired by the excellent [Eric Veach's PhD thesis](http://graphics.stanford.edu/papers/veach_thesis/) (it's more of a book than a PhD dissertation ;-)), which I recommend to read for more detailed explanation and also to everyone who is seriously interested in computer graphics, especially in realistic 3D rendering.

##### BSDF under a smooth refractive interface

Using the general definition of BSDF, we'll now derive the proper formula for a layer under a smooth Fresnel surface. For that we will need to prepare 4 prerequisite formulae, which we will then use to build the final BSDF formula

*[image #RefrBsdfGeomFull: Full geometry of a BSDF under a smooth refractive interface.]*

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

The third piece of our puzzle is how the projected solid angle $\mathrm{d}\sigma^{\bot}$ of a light cone changes when refracted through smooth interface. The relation between the incident and refracted light cone can be found for example in the Veach's thesis in the equation 5.4 at page 143 and, after a minor rewrite, we will get

$$
\mathrm{d}\sigma^{\bot} \left(\omega_i\right) = \frac{\eta_1^2}{\eta_0^2} \mathrm{d}\sigma^{\bot} \left(\omega_t\right)
$$

where $\omega_t$ is the refracted direction of $\omega_i$. SPOILER ALERT: this relationship contains the missing component of our new BSDF formula.

The fourth and the last prerequisite is a little technical and I will not go into detail of this one. The point is, but it can be shown that the projected solid angle measure of the refracted cone equals the cone of light reaching the inner layer given that the both layers are parallel *to each other*:

$$
\mathrm{d}\sigma^{\bot} \left({\omega}_i^\prime\right) = \mathrm{d}\sigma^{\bot} \left({\omega}_t\right)
$$

Now we finally have all the necessary pieces to build the BSDF for the a layer under a smooth Fresnel surface. Let's *put/substitute* them into the plain definition of BSDF, simplify it and separate the BSDF of the stand-alone inner layer BSDF to see what compensation factors have to be applied to get the correct formula:

$$
\begin{eqnarray*}
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right)&=&\frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{L_{i}\left(\omega_{i}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}\right)}\\
&=&\frac{\mathrm{d}L_{o}\left(\omega_{o}^{\prime}\right)\frac{\eta_{0}^{2}}{\eta_{1}^{2}}T\left(\theta_{o}\right)}{L_{i}\left(\omega_{i}^{\prime}\right)\frac{\eta_{0}^{2}}{\eta_{1}^{2}}T\left(\theta_{i}\right)^{-1}\frac{\eta_{1}^{2}}{\eta_{0}^{2}}\mathrm{d}\sigma^{\bot}\left(\omega_{i}^{\prime}\right)}\\
&=&\frac{\mathrm{d}L_{o}\left(\omega_{o}^{\prime}\right)}{L_{i}\left(\omega_{i}^{\prime}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}^{\prime}\right)}T\left(\theta_{o}\right)T\left(\theta_{i}\right)\frac{\eta_{0}^{2}}{\eta_{1}^{2}}\\
&=&f_{s}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{o}\right) T\left(\theta_{i}\right) \frac{\eta_{0}^{2}}{\eta_{1}^{2}}
\end{eqnarray*}
$$

And that's it, folks! You can see that our original approach, *in which we just used the refracted direction to evaluate the inner layer model along with attenuating the result with Fresnel transmission coefficients*, was almost *correct*. What we were missing was the (relatively trivial) compensation factor $\frac{\eta_{0}^{2}}{\eta_{1}^{2}}$, which, however, makes the difference as you can see in the following images:

*[images: Solid angle compression applied. Ideally white (albedo 100%) Lambert layer under brown medium (thicknesses: large to none; 3 lights)]*

It's important to note, that we use single-point simplification here as *was used* in the original WWL paper -- i.e. we assume the incoming and outgoing light to pass through the same point on the outer layer.

*Mention Mitsuba's version? Even they struggled with it and still don't use the correct BSDF. Maybe after I try its approach practically and show the results...*

#### Inner layer formula

If we put together all the components which affect the inner layer contribution, we'll get the final formula for the inner layer:

$$
f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{i}\right) T\left(\theta_{o}\right) \frac{\eta_{0}^{2}}{\eta_{1}^{2}} a\left(\theta_{i}, \theta_{o}\right)
$$

where $f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right)$ is the stand-alone BSDF of the inner layer, $T\left(\theta_{i}\right)$ and $T\left(\theta_{o}\right)$ are the Fresnel transmission coefficients, $\frac{\eta_{0}^{2}}{\eta_{1}^{2}}$ is the projected solid angle compression compensation and $a$ is the medium attenuation coefficient.

Just a side note: Since we are implicitly dealing with a rendering system which is not polarization aware we don't have to worry about the order in which the coefficients are multiplied together. In a polarization-aware system, however, multiplication operations (representing light-matter interaction) would have to be dealt with much more care.

### Whole formula

Just to summarize, the complete BSDF formula containing contributions of both layers is

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right) = f_{s1}^{\ast}\left(\omega_{i}\rightarrow\omega_{o}\right) + f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{i}\right) T\left(\theta_{o}\right) a\left(\theta_{i}, \theta_{o}\right) \frac{\eta_{0}^{2}}{\eta_{1}^{2}}
$$

where $f_{s1}^{\ast}$ and $f_{s2}^{\ast}$ are the stand-alone outer and inner layer [BSDFs](https://en.wikipedia.org/wiki/Bidirectional_scattering_distribution_function), $T \left(\theta_{i}\right)$ and $T\left(\theta_{o}\right)$ are the Fresnel transmission coefficients, $a\left(\theta_{i}, \theta_{o}\right)$ is the medium attenuation (see section "Medium attenuation") and $\frac{\eta_{0}^{2}}{\eta_{1}^{2}}$ is the projected solid angle compression compensation. It's important to notice that the formula is, in fact, just a [BRDF](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function) rather than BSDF, because it is properly defined only for directions from the upper hemisphere.

The complete model for one type of configuration may look like this:

*[images: Highly glossy outer layer, ideally white (albedo 100%) Lambert inner layer and brown medium between them (thicknesses: large to none; 3 lights)]*	

## Sampling

In a [Monte Carlo](https://en.wikipedia.org/wiki/Monte_Carlo_integration)-based (MC) rendering system (e.g. path tracing) a BSDF model needs to provide not only an evaluation formula, but also a direction *sampling strategy* with evaluation of its [PDF](https://en.wikipedia.org/wiki/Probability_density_function). Since the sampling strategy can be partially described by its PDF, it is often sufficient to use the symbol of the PDF to denote the whole strategy (e.g. $p$).

In our layered model we assume that the stand-alone *outer and inner layer* BSDFs $f_{s1}^{\ast}$ and $f_{s2}^{\ast}$ provide their own sampling strategies $p^{\ast}_1$ and $p^{\ast}_2$ respectively, which we will use to build a sampling strategy $p$ for the whole model. Since our BSDF is a simple sum of two slightly modified BSDFs, we can build a reasonably efficient overall sampling strategy $p$ by blending two slightly modified strategies $p^{\ast}_1$ and $p^{\ast}_2$, given that they are reasonably efficient on their own:

$$
p\left(\omega_{i},\omega_{o}\right) = w_{1}p_{1}\left(\omega_{i},\omega_{o}\right) + w_{2} p_{2}\left(\omega_{i},\omega_{o}\right)
$$

where $p_1$ and $p_2$ are the sampling strategies of the respective BSDF layers contributions (modified strategies $p^{\ast}_1$ and $p^{\ast}_2$), $\omega_{i}$ is the (sampled) direction of incident light, $\omega_{o}$ is the (fixed) outgoing light direction and $w_1$ and $w_2$ are the blending coefficients.

First, we'll have a look at separate sampling strategies $p_1$ and $p_2$, then we'll combine them together to obtain $p$.

### Outer layer sampling

Because the outer layer BSDF $f_{s1}^{\ast}$ is evaluated directly without any modification, we can use the original sampling strategy of the outer layer stand-alone BSDF

$$
p_1\left(\omega_{i}, \omega_{o}\right) = p_1^{\ast}\left(\omega_{i}, \omega_{o}\right)
$$

The renderer just has to instruct the sampling routine to draw samples only from the upper hemisphere because we are interested only in the reflected contribution of the layer, not refracted.

### Inner layer sampling

*[image #InnerLayerSampling: Sampling the inner layer contribution]*

Sampling the inner layer component is not hard either, we just have to feed the original strategy of the inner stand-alone BSDF with $\omega_{o}^{\prime}$ -- the refracted version of the outgoing direction and refract the generated incoming direction $\omega_{i}^{\prime}$ back into the outside world. It is possible that the generated direction $\omega_{i}^{\prime}$ is refracted back into the model due to [total internal reflection](https://en.wikipedia.org/wiki/Total_internal_reflection) (TIR). The resulting sample in such case is still valid, it must not be discarded and its contribution is zero because our model neglects energy which undergoes multiple scattering events. This approach will modify the shape of the original sampling PDF in the same way we modified the shape of the original stand-alone BSDF.

Now that we have constructed the sampling routine, we also need to evaluate its PDF. At first sight it seems that we just have to evaluate the original PDF using the refracted directions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$, analogically to the way we modified the original sampling strategy

$$
p_2\left(\omega_{i}, \omega_{o}\right) = p_2^{\ast}\left(\omega_{i}^{\prime}, \omega_{o}^{\prime}\right)
$$

If we feed a MC renderer with such PDF, the result will look like this:

*[images: "Too dark inner layer problem" Just the inner layer. Lambert. Reference vs. model. 3 lights]*

As you can see, the result is consistently darker than it should be according to the reference images. It's because our PDF is computed incorrectly! The reason is that the non-uniform change of directions due to refraction changes the *solid* angular density of directional samples. Since the sampling PDF expresses the density of generated samples with respect to the solid angle measure, we need to compensate the original PDF accordingly to get the correct value.

$$
p_2\left(\omega_{i}, \omega_{o}\right) = p_2^{\ast}\left(\omega_{i}^{\prime}, \omega_{o}^{\prime}\right) \frac{\eta_{0}^2 \cos\theta_{i}}{\eta_{1}^2 \cos\theta^{\prime}_{i}}
$$

This compensation factor is closely related to what happens to radiance when it gets refracted through a smooth interface between two media with different refractive indices. Its application will finally yield the correct PDF values resulting in an unbiased Monte Carlo estimator leading to the correct rendering output:

*[images: "Too dark inner layer problem fixed." Just the inner layer. Lambert. Reference vs. model. 3 lights]*

### Sampling both layers

*Naïve approach? One could possibly use some ad-hoc/fixed weighting approach, but to obtain better result, we need to be more sophisticated about it....*

To obtain an efficient MC estimator of the *scattering/rendering* integral, the sampling PDF has to be as proportional to the integrand as possible. Ideally, the sampling PDF should be the normalized version of the integrand. i.e. scaled by a factor to make its integral over the whole sphere equal 1. The integrand in the practically used form is

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right)L_{i}\left(\omega_{i}\right)\cos\theta_{i}
$$

However, because we usually don't know the distribution of the incident radiance, the sampling routine is usually trying to mimic at least the product of the BSDF and the cosine factor.

$$
f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right)\cos\theta_{i}
$$

In the case of our layered model, the BSDF is a sum of two components so the integrand looks like

$$
f_{s1}\left(\omega_{i}\rightarrow\omega_{o}\right) \cos\theta_{i} + f_{s2}\left(\omega_{i}\rightarrow\omega_{o}\right) \cos\theta_{i}
$$

It would not be *very* reasonable to try to derive a sampling strategy directly for the sum formula because the components are general functions which can be hard to sample even as stand-alone functions and which we have little information about anyway. We will instead build the overall sampling strategy on top of the already existing strategies $p_1$ and $p_2$ just by blending them together. If we can sample and evaluate both $p_1$ and $p_2$, it is easy to sample and evaluate a weighted average of the two

$$
p = w_{1}p_{1} * w_{2}p_{2}
$$

The main question *then* is how to choose the weights to make the resulting PDF as proportional to $f_{s}$ as possible. Ideally, the weights should be proportional to the integrals of respective components and the overall strategy would then be as good as the partial ones. The problem is that the integrals are in general hard to evaluate precisely so we have to resort to an approximation.

#### Outer layer integral

On the outer layer, the light is either reflected from or refracted through the layers micro-facets and we will approximate the amount of reflected light just by the Fresnel reflection coefficient of a smooth interface:

$$
w_{1}^{\ast} = F\left(\theta_{o}\right)
$$

#### Inner layer integral

Let's now recall the inner layer contribution

$$
f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{i}\right) T\left(\theta_{o}\right) \frac{\eta_{0}^{2}}{\eta_{1}^{2}} T\left(\theta_{i}\right) a\left(\theta_{i}, \theta_{o}\right)
$$

Let's now approximate its integral

$$
\frac{\eta_{0}^{2}}{\eta_{1}^{2}} T\left(\theta_{o}\right) \int_{\mathcal{H}_{+}^{2}} {T\left(\theta_{i}\right) a\left(\theta_{i}, \theta_{o}\right) f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) \cos\theta_{i} \mathrm{d}\omega_{i}}
$$

Note that the components independent from $\omega_{i}$ were moved outside the integral and we integrate only over the upper hemisphere $\mathcal{H}_{+}^{2}$ rather than over the whole sphere $\mathcal{S}^{2}$ because there's no transmission through the inner layer allowed.

While the components outside the integral are easy to evaluate analytically, the integral part is more complicated and has to be approximated. We will start with the partial integral

$$
\int_{\mathcal{H}_{+}^{2}} f_{s2}^{\ast}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) \cos\theta_{i} \mathrm{d}\omega_{i}
$$

We assume that the stand-alone BSDFs can compute their reflectance over the upper hemisphere

$$
\rho_{s2}^{\ast} = \int_{\mathcal{H}_{+}^{2}} f_{s}\left(\omega_{i}\rightarrow\omega_{o}\right)\cos\theta_{i}\mathrm{d}\omega_{i}
$$

and we will use it as an approximation of the partial integral. This is rather inaccurate because $\rho_{s2}^{\ast}$ uses the original directions $\omega_{i}$ and $\omega_{o}$ instead of their refracted versions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$; therefore, the shape of the integrand is not stretched out and also contains the parts which are cut away by the [total internal reflection](https://en.wikipedia.org/wiki/Total_internal_reflection) in our model, which can lead to substantial error especially at grazing angles. However, this approximation contains -- at least partially -- the characteristics of the inner material and is better than nothing.

Both missing components $T\left(\theta_{i}\right)$ and $a\left(\theta_{i}, \theta_{o}\right)$ change the shape of the integrand and have to be taken into account, but their effect is hard to solve analytically. One way to approximate their overall effect is to evaluate them over the mirror reflection path ($\theta_{i} = \theta_{o}$) -- analogically to what we did in evaluating the sub-surface scattering contribution of the whole model -- and pretend they are constant over the whole hemisphere.

After putting everything together, the resulting approximation for the inner layer contribution integral is then

$$
w_{2}^{\ast} = 
\frac{\eta_{0}^{2}}{\eta_{1}^{2}} T^2\left(\theta_{o}\right) a\left(\theta_{o}, \theta_{o}\right) \rho_{s2}^{\ast}
$$

Where $\eta_{1}^{2}$ and $\eta_{0}^{2}$ are refractive indices of the respective media, $T\left(\theta_{o}\right)$ is the Fresnel transmission coefficient, $a\left(\theta_{i}, \theta_{o}\right)$ is the medium attenuation (see the section "Medium attenuation"), and $\rho_{s2}^{\ast}$ is the reflectance of the stand-alone inner layer BSDF.

#### Complete sampling routine

Now that we have the integrals approximations $w_{1}^{\ast}$ and $w_{2}^{\ast}$, we obtain the PDF weights by normalizing the approximations

$$
\begin{eqnarray}
w_{1}&=&\frac{w_{1}^{\ast}}{w_{1}^{\ast} + w_{2}^{\ast}} \\
w_{2}&=&\frac{w_{2}^{\ast}}{w_{1}^{\ast} + w_{2}^{\ast}}
\end{eqnarray}
$$

which define our mighty overall sampling routine

$$
p = w_{1}p_{1} * w_{2}p_{2}
$$

To sample from blended PDF, first randomly pick one of the sampling sub-routines $p_1$ and $p_2$ with probabilities equal to $w_1$ and $w_2$ respectively and then draw sample from the selected sub-routine. To evaluate the probability density of the sample, evaluate the whole PDF $p$.

*[image: Sampling performance: Low sample rate. Just furnace test to isolate the BSDF sampling performance. Strategies: fixed ratio?, Fresnel ratio, my approach.]*

## Model Analysis

- Certain configuration...
- ...comparison of the behaviour with path-traced reference renderings...
- Images, lots of images :-)


### Reference images

Story?...

### No medium attenuation

- energy conservation, fitting the reference, ...

### *Later: With medium attenuation*

...

## Conclusion

...

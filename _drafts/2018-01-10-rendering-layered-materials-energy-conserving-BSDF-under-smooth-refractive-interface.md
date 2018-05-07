---
layout: post
title: "Rendering Layered Materials: Energy Conserving BSDF Under a Smooth Refractive Interface"
comments: true
---

This is a part of the blog post series [Rendering Layered Materials](rendering-layered-materials.html).

Here I explain how to obtain a correct energy-conserving BSDF for a given BSDF seen and illuminated through a smooth refractive interface with a single scattering event simplification. To do that, I will first define a few basic radiometric quantities and BSDF and then I will derive the BSDF for a model under a smooth refractive interface. The model uses the single-point simplification which assumes that the point through which the light ray enters the outer material interface is the same as the one through which the light ray exists. For the sake of clarity, this explanation neglects the reflection contribution of the smooth interface as it is trivial to add to the resulting model.

## Main idea

One way of obtaining the correct form of a BSDF under a smooth refractive interface its to re-formulate the inner layer BSDF as a function of (non-refracted) incoming and outgoing directions of the whole model $\omega_{i}$ and $\omega_{o}$ rather than *as a function* of refracted directions $\omega_{i}^{\prime}$ and $\omega_{o}^{\prime}$:

<p style="text-align: center">
<img src="../images/SRAL/RefrBsdfGeomAngles.svg" alt="" width="500" />
</p>

where  $f_{s1}$ is the smooth interface, $f_{s2}$ is the given BSDF, $\eta_0$, and $\eta_1$ are the refractive indices of media separated by the outer smooth layer, $N_g$ is the geometrical normal of the surface.

In other words: we want to see how the layer behaves to the rendered (e.g path-tracer) which doesn't know about the internal mechanics of the model (refractions, reflections, attenuations, etc.) and regards the evaluated BSDF as a black box.

## Radiometric quantities

To handle the problem properly we need to *go down/grasp it through* to the fundamental *concepts/theory* upon which the model is built. *My explanation assumes knowledge of mathematical concepts like derivative, measure, solid angle, etc.*

First, we need to define a few radiometric quantities, which the BSDF theory builds upon. There are several ways of doing that, but I will follow the measure-theoretic radiometry way, which defines its quantities as ratios of measures and is explained in more detail for example in the third chapter of the [Eric Veach's dissertation thesis](http://graphics.stanford.edu/papers/veach_thesis/). This approach uses the so-called Radon-Nikodym derivative, which expresses density of one measure with respect to another one. For measures $Q$ and $\rho$ the corresponding Radon-Nikodym derivative is denoted $\frac{\mathrm{d}Q}{\mathrm{d}\rho}$ (similarly to the "normal" derivative). In the measure-theoretic radiometry case, we usually express the density of electro-magnetic energy measure (amount of photons, loosely speaking) with respect to some space and/or time measure.

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

## BSDF definition

Now that we have defined the needed radiometry quantities, we can finally define the *bidirectional scattering density function* (BSDF), a formal description of the light-scattering properties of a surface point. It expresses how the light outgoing from a point $x$ on a surface in a particular direction $\omega_o$ is dependent on the light incoming to the point from a particular direction $\omega_i$:

<p style="text-align: center">
   <img src="../images/BRDF definition geometry.svg" alt="" width="400" /><br/>
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

## BSDF under a smooth refractive interface

Using the general definition of BSDF, we'll now derive the proper formula for a layer under a smooth Fresnel surface. For that we will need to prepare 4 prerequisite formulae, which we will then use to build the final BSDF formula.

<p style="text-align: center">
<img src="../images/SRAL/RefrBsdfGeomSolidAngles.svg" alt="" width="500" />
</p>

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

where $\omega_t$ is the refracted direction of $\omega_i$.

SPOILER ALERT: this relationship contains the missing component of our new BSDF formula ;-)

The fourth and the last prerequisite is a little technical and I will not go into detail of this one. The point is that the projected solid angle measure of the refracted cone equals the cone of light reaching the inner layer given that the both layers are parallel *to each other*:

$$
\mathrm{d}\sigma^{\bot} \left({\omega}_i^\prime\right) = \mathrm{d}\sigma^{\bot} \left({\omega}_t\right)
$$

Now we finally have all the necessary pieces to build the BSDF for the a layer under a smooth Fresnel surface. Let's *put/substitute* them into the plain definition of BSDF, simplify it and separate the BSDF of the stand-alone inner layer BSDF to see what compensation factors have to be applied to get the correct formula:

$$
\begin{eqnarray*}
f_{s}^{\ast}\left(\omega_{i}\rightarrow\omega_{o}\right)&=&\frac{\mathrm{d}L_{o}\left(\omega_{o}\right)}{L_{i}\left(\omega_{i}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}\right)}\\
&=&\frac{\mathrm{d}L_{o}\left(\omega_{o}^{\prime}\right)\frac{\eta_{0}^{2}}{\eta_{1}^{2}}T\left(\theta_{o}\right)}{L_{i}\left(\omega_{i}^{\prime}\right)\frac{\eta_{0}^{2}}{\eta_{1}^{2}}T\left(\theta_{i}\right)^{-1}\frac{\eta_{1}^{2}}{\eta_{0}^{2}}\mathrm{d}\sigma^{\bot}\left(\omega_{i}^{\prime}\right)}\\
&=&\frac{\mathrm{d}L_{o}\left(\omega_{o}^{\prime}\right)}{L_{i}\left(\omega_{i}^{\prime}\right)\mathrm{d}\sigma^{\bot}\left(\omega_{i}^{\prime}\right)}T\left(\theta_{o}\right)T\left(\theta_{i}\right)\frac{\eta_{0}^{2}}{\eta_{1}^{2}}\\
&=&f_{s}\left(\omega_{i}^{\prime}\rightarrow\omega_{o}^{\prime}\right) T\left(\theta_{o}\right) T\left(\theta_{i}\right) \frac{\eta_{0}^{2}}{\eta_{1}^{2}}
\end{eqnarray*}
$$

And that's it, folks! You can see that our original approach, *in which we just used the refracted direction to evaluate the inner layer model along with attenuating the result with Fresnel transmission coefficients*, was almost *correct*. What we were missing was the (relatively trivial) compensation factor $\frac{\eta_{0}^{2}}{\eta_{1}^{2}}$, which, however, makes the difference as you can see in the following images:

<p style="text-align: center">
<img src="../images/SRAL/Blog_InnerGlossyMedium_SolAngProblem_EM1_512s.jpg" alt="" width="700" /><br/>
<img src="../images/SRAL/Blog_InnerGLossyMedium_SolAngComp_EM1_512s.jpg" alt="" width="700" /><br/>
Without and with solid angle compression compensation applied: Glossy inner layer (GGX roughness 0.1) with refracted directions, Fresnel attenuation under orange medium with varying medium thickness under constant lighting.

*TODO: ?more general settings (various roughnesses, without medium)?*

</p>

It's important to note, that we use single-point simplification here as *was used* in the original WWL paper -- i.e. we assume the incoming and outgoing light to pass through the same point on the outer layer.

*Mention Mitsuba's version? Even they struggled with it and still don't use the correct BSDF. Maybe after I try its approach practically and show the results...*
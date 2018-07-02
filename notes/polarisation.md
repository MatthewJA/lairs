---
title: "Notes on POSSUM and polarisation"
author: "Matthew J. Alger"
geometry: margin=3cm
latex-engine: xelatex
---

# Data sources

We have a few potential sources of polarisation data:

- GOODS-N
- ATLAS CDFS/ELAIS-S1 --- 1.4 GHz, 10$^\circ{}$, $10''$, 10 µJy, 4000 components
- VLASS --- ongoing, 2--4 GHz, NVSS-area, $2.5''$, 70 µJy, 200000 components
- ASKAP cosmology fields
- SKADS, an SKA observations simulation

# Polarisation

Light is measured in four *Stokes channels*: I, Q, U, and V. I is the total intensity of the light, Q and U are linear polarisation, and V is circular polarisation (which we expect to be 0 for almost all extragalactic sources so we ignore it). The *total polarised intensity* is

$$
    P^2 = Q^2 + U^2
$$

For a fully-polarised monochromatic wave, $P = I$, but because waves aren't really monochromatic and radio sources are rarely fully-polarised $P \leq I$.

The *fractional polarisation* is

$$
    \Pi = \sqrt{\frac{P}{I}}.
$$

Q and U together give us the polarisation angle $PA$ or $\chi$.

$$
    PA = \chi = \frac{1}{2} \mathrm{atan}\left(\frac{U}{Q}\right).
$$

$\chi$ varies as a function of wavelength $\lambda$. We report this in terms of $\lambda^2$ since this results in a linear function for a polarised observation not affected by magnetic fields. The slope of the variation is called the *rotation measure* $RM$:

$$
    RM = \frac{\Delta \chi}{\Delta \lambda^2}.
$$

$RM$ is measured in $\mathrm{rad}\ \mathrm{m}^{-2}$. It is related to the magnetic field $B$ along the line of sight:

$$
    RM = 0.8 \int n_e(\ell) B_\parallel(\ell)\ d\ell.
$$

If $RM$ is constant then the source is called *Faraday simple*. Otherwise it is *Faraday complex*. Due to observational effects and noise it can be hard to distinguish between Faraday simple and Faraday complex sources.

$RM$ is only affected by *changes* in $B_\parallel$. An object that only changes the $RM$ once (perhaps it has a constant magnetic field) is called *Faraday thin*. Otherwise it is called *Faraday thick*.

POSSUM will give us many observations of $\chi(\lambda^2)$ for any given source. This means we can observe this function (called the *Faraday spectrum*) at a much higher resolution than before. For comparison NVSS provides several observations.

Fourier transforming $\chi(\lambda^2)$ gives $\chi(\phi)$ where $\phi$ is the conjugate axis of $\lambda^2$ called the *Faraday depth*. Constant $RM$ shows up as a delta function here. Multiple magnetic fields will show up as multiple deltas. Additionally we will get an analogue to the point spread function called the *rotation measure transfer function* (RMTF) which spreads out the deltas in practice.
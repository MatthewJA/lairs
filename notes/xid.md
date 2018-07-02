---
title: Radio Host Cross-identification
author: Matthew J. Alger
latex-engine: xelatex
geometry: margin=1in
amsmath: ''
amssymb: ''
---

*Radio source host galaxy cross-identification* is the task of associating radio emission with its infrared or optical host galaxy. This can be a complicated task: Radio objects may be extended far from the host galaxy and can become complex, messy structures due to interactions with the environment. In general, radio emission may bear no obvious relationship to the location of its host galaxy.

In upcoming radio surveys like the Evolutionary Map of the Universe (EMU; Norris+11), we will need to cross-identify large numbers of radio objects, but there will be far too many objects to cross-identify manually. Astronomers hope that machine learning will provide a solution for automating cross-identification.

This document is a loose collection of thoughts, facts, and definitions about cross-identification.

Throughout, we refer to *candidate host galaxies* as the target of our cross-identification --- that is, we seek associations between radio emission and candidate host galaxies. Candidate host galaxies can be drawn from an infrared or optical survey and ideally are *only* galaxies, but these surveys generally also contain stars and it is not always possible to definitively distinguish between stars and galaxies. For this reason, "candidates" or "candidate hosts" is the preferred abbreviation rather than "galaxies", though ideally the set of candidate host galaxies only contains galaxies.

A *radio component* is a Gaussian fit to some observed radio emission. A *radio island* is a set of connected (at $\geq 5\sigma$) radio components; Radio Galaxy Zoo refers to these as "components". A *radio source* is a set of radio islands that are physically associated --- in the context of extragalactic radio sources, this means that the islands have the same host galaxy. A *compact island* is an island composed of just one component the size of the beam. A *compact source* is a source comprised of just one component the size of the beam.

# Approaches to cross-identification
The most common approach to cross-identification is a combination of manual, expert examination of sources and positional matching. Manual examination is robust and relatively reliable, while positional matching is efficient but less reliable. Note that manual examination is not *entirely* reliable as there are intrinsic difficulties in this task: Banfield+15 estimated that 10% of extended radio sources are impossible to reliable cross-identify from wide-area surveys and found that in a first pass of manual cross-identification experts disagreed on 47% of selected sources.

Norris+06 and Middelberg+08 used this combination of methods to cross-identify objects in the Australia Telescope Large Area Survey (ATLAS) with infrared objects from the *Spitzer* Wide-area Infrared Extragalactic Survey (SWIRE; Lonsdale+03, Surace+05). They generated a catalogue of around 2000 cross-identifications (the *Norris/Middelberg cross-identifications* or *ATLAS expert cross-identifications*) and suggested that at least 91% of *Chandra* Deep Field South (CDFS) cross-identifications were correct using this approach (Norris+06). No research currently exists on how reliable these cross-identifications actually are, though this is likely to happen in future with Jesse Swan's manual cross-identification of the entirety of ATLAS and Ivy Wong's publishing of the Radio Galaxy Zoo ATLAS cross-identifications (Wong+??). Ray Norris has claimed an expert accuracy of ~99% on ATLAS in various talks, though this has not been published anywhere I can find.

Kimball+08 cross-identified radio objects detected in Faint Images of the Radio Sky at Twenty-centimeters (FIRST; Becker+95, Helfand+15) with optical sources from the Sloan Digital Sky Survey (SDSS; Adelman-McCarthy+07) by both a) selecting the nearest neighbour within $60''$ of each radio component, and b) selecting the brightest nearby galaxy for each radio component (within a series of fixed radii --- $3'', 10'', 30'',$ and $60''$). Estimates for misidentification rates are between 2%--8.1% (Kimball+08) but no concrete accuracy exists to my knowledge. I refer to these as the *Kimball cross-identifications*.

Radio Galaxy Zoo (RGZ; Banfield+15, Wong+??) turned to the internet to crowdsource the cross-identification task on the Zooniverse citizen science platform. RGZ presents an interface for non-expert volunteers to manually cross-identify radio components from FIRST and ATLAS-CDFS. Around 75,000 cross-identifications have now been completed. Banfield+15 estimated the volunteers' accuracy to be around 74% on a selection of radio sources with extended morphologies and Wong+?? breaks down the accuracy as a function of consensus level (defined shortly). Additionally, there is redundancy in the cross-identifications: Each radio object is shown to up to 20 volunteers. This allows us to find a kind of uncertainty by comparing the agreement of cross-identifications by different volunteers. RGZ refers to this as the *consensus level* (or simply *consensus*) for an object. This is similar to the disagreement fraction in query-by-committee active learning (Seung+92), suggesting that the consensus level of RGZ cross-identifications may be used to seek interesting or high-information content radio objects. Wong+?? will compare the RGZ cross-identifications to a sample of RGZ cross-identifications. They are yet to be compared to the Kimball cross-identifications.

Fan+15 presented a Bayesian model-fitting approach to the cross-identification problem. Lobe-core-lobe models are fit to sets of radio objects and the host galaxy is coincident with the core. This approach, applied to ATLAS-CDFS, gave cross-identification results almost entirely corresponding to those found by Norris+06. It does however assume a very specific model that will not work as well on the more complicated structures in higher resolution surveys like FIRST (though it might be perfect for the compact-dominated EMU!).

I developed an method applying the computer vision "sliding window" technique for object localisation (Alger+18) and applied it to ATLAS and FIRST. I will describe and discuss this later in this document.

# Formalism

Note that there are many ways to formalise cross-identification. Here, we describe some of them, but there is no right or wrong way to do this. Each formalism has different advantages and disadvantages.

We can formalise the cross-identification problem by defining a space of radio objects $\mathcal R$ and a space of candidate host galaxies $\mathcal G$. We want to find all associations between radio objects in $\mathcal R$ and candidate hosts in $\mathcal G$. This is a bipartite graph with nodes $\mathcal R \cup \mathcal G$ and edges $(r, g) \in \mathcal R \times \mathcal G$; cross-identification is then the problem of finding the edges given the nodes. This is impractical to solve directly, but it is a useful way to think about cross-identification and helps show the many different ways that cross-identification can be approached. This formalism may also help characterise label noise in cross-identification catalogues, which would be very useful for crowdsourced catalogues like RGZ.

It is worth noting at this point that our formalism isn't all that well-defined: What is a "radio object"? Observing the radio sky at different frequencies and resolutions can give dramatically different results, and a radio source that seems to be a single, complete object in one survey may appear as five disconnected objects in another survey. We defer this problem for now and define a *radio object* to be a connected patch of radio emission above a fixed signal-to-noise ratio (i.e. an island) for a fixed disjoint set of radio observations. We will need to address this problem eventually, but this requires an investigation of radio-radio cross-identification (e.g. Kimball+08).

## Finding components: a causal approach

Our observations of the radio sky are generated by underlying physics. Radio objects are generated by their host galaxies (or for active galactic nuclei, by some central engine contained in the host galaxy). We can therefore look at cross-identification from a causal perspective: Host galaxies cause radio objects, so given a host galaxy, find the radio objects it generated. We could also refer to this as an *infrared-selected* (or *optically-selected*) approach. This can be formalised as a function

$$
    f : \mathcal G \to 2^{\mathcal R}
$$

where $2^\mathcal X$ denotes the powerset of $\mathcal X$. Note that $\emptyset$ is a valid result of this function, so $f$ is a total function that can be applied to all candidate hosts. Note also that there is no inherent constraint that radio objects are uniquely associated to a host.

We could, equivalently, have some function

$$
    f : \mathcal G \times \mathcal R \to \mathbb B
$$

where $\mathbb B = \{0, 1\}$. This might be more conducive to automated methods as we can think of it as a comparison of pairs of radio objects and candidate hosts. This function predicts directly the edges of the bipartite graph described above.

This form of $f$ can be used to describe manual verification of automated cross-identifications such as those presented by Norris+06. When manually checking some method, an astronomer might examine each radio-candidate pairing and determine whether it seems plausible.

We could even generalise this function to return real numbers:

$$
    S : \mathcal G \times \mathcal R \to \mathbb R.
$$

In this context, $S$ is a scoring function that evaluates how likely an association is. Perhaps we could interpret this as the joint probability of observing a galaxy with an associated radio object.

## Finding hosts: an anticausal approach

We can flip this formalism around and instead talk about an anticausal approach. Given a radio object, where did it come from? We could refer to this as a *radio-selected* approach. This can be represented by functions of the form

$$
    f : \mathcal R \to \mathcal G
$$

and it is immediately obvious that this is in some sense nicer than our first attempt at a causal function. There is an implicit assumption that there is exactly one host galaxy for every radio object. Physically this is true (though we might not be able to observe the host galaxy in practice as radio is visible at much higher redshifts than infrared or optical). We can rewrite this as

$$
    f : \mathcal R \times \mathcal G \to \mathbb B
$$

though we have now expanded the set of possible functions and lost the assumption from before. This form, however, is equivalent to our causal approach, so it is clear that the anticausal approach is a subset of the causal approach.

# Integer linear programming

We can, of course, try and solve the graph-finding problem directly. Define a matrix $Z \in \mathbb B^{|\mathcal G| \times |\mathcal R|}$. $Z$ encodes our solution: $Z_{ij} = 1$ iff the $i$th radio object is associated with the $j$th candidate host galaxy. We refer to these objects as $r_i \in \mathcal R$ and $g_j \in \mathcal G$ respectively. In this framework, we can encode our assumptions as constraints. The main assumption we make is that a radio object is associated with exactly one host galaxy (the *unique host* assumption). This is *physically* true, but from an observational point of view it does not hold as radio emission from different host galaxies can overlap to form one radio object. Such radio objects are rare, so it is a reasonable assumption. The unique host assumption is:

$$
\forall i\ldotp \sum_{j = 1}^{|\mathcal G|} Z_{ij} \leq 1.
$$

This can be read as "each radio object has at most one host galaxy". Another assumption we make is that every radio object has a host galaxy (the *visible host* assumption) --- again, this is physically true, but as we can see much higher redshift objects in radio than we can in infrared or optical there are plenty of radio objects that do not have a visible host galaxy. Unfortunately this assumption is broken quite often, with an estimated 44% of RGZ sources having no identified host galaxy. Nevertheless, we make the assumption because otherwise the cross-identification task is somewhat ill-defined as not all radio objects can be cross-identified --- this is a problem we will have to return to later. The visible host assumption is:

$$
\forall i\ldotp \sum_{j = 1}^{|\mathcal G|} Z_{ij} \geq 0
$$

and we can combine this with the unique host assumption to neatly summarise all our assumptions:

$$
\forall i\ldotp \sum_{j = 1}^{|\mathcal G|} Z_{ij} = 1.
$$

This is a linear constraint, so we might be able to cast cross-identification as an integer linear program. For this, we need some objective function $L(Z)$, and for a linear program this must be linear:

$$
L(Z) = -\langle S, Z \rangle = -\mbox{tr}(S^T Z).
$$

We want to minimise $L(z)$ subject to our constraint. Of course, we have not defined $S$, so this is simply the functional form of our objective function. We can define $S$ based on some kind of scoring function which scores how good each association is, i.e.

$$
S_{ij} = S(r_i, g_j),
$$

and this is essentially a classifier. This has the unfortunate constraint that there are no cross-terms, so the associations of all radio components and candidate hosts are evaluated independently. Perhaps the simplest solution is to simply add cross-terms and get some kind of quadratic program:

$$
    L(Z) = -\langle S, Z \rangle - \sum_{ijkl} S(r_i, g_j, r_k, g_l) Z_{ij} Z_{kl}.
$$

This is, however, a lot more complicated and there is no clear approach to it. It may also be the case that a linear program is plenty powerful enough to solve the problem.

The form of our constraint begs the question: What can we do with $S$? We can sum over all radio objects to obtain some kind of measure of how host-like a candidate is:

$$
\mbox{hostiness}(g_j; S) = \sum_{i = 1}^{|\mathcal R|} S_{ij}.
$$

We can sum over all candidate hosts to get some kind of host density near a radio object:

$$
\mbox{density}(r_j; S) = \sum_{j = 1}^{|\mathcal G|} S_{ij}.
$$

Perhaps we can also find an entropy over candidate hosts as some kind of confusion score:

$$
\mbox{confusion}(r_j; S) = -\sum_{j = 1}^{|\mathcal G|} S_{ij} \log S_{ij}.
$$

It would be interesting to compare the cross-identification confusion to the RGZ consensus level.

# Positional matching

Surprisingly, there is no dedicated analysis of a positional matching approach to cross-identification in radio, though the approach is widely used (e.g. Norris+06, Kimball+08). In a positional matching approach, radio components are cross-identified with the galaxy that is closest on the sky to the centre of the Gaussian fitting the component. For unresolved single-component sources, this works well as the host galaxy tends to directly coincide with the centre of the object. For extended sources, positional matching tends to work poorly as this no longer holds (and for extended sources with multiple, disconnected components, multiple host galaxies will be identified for the one source).

Our results (Alger+18) show that positional matching works very well on radio components in ATLAS, with approximately 95% accuracy. Splitting each field into compact and resolved objects shows that positional matching achieves almost 100% accuracy on compact objects in both fields, but deviates between the fields for resolved objects. The accuracies on resolved components in CDFS and ELAIS-S1 are around 75% and 95% respectively. It is worth noting that these accuracies were assessed against the Norris/Middelberg cross-identifications, which were partially generated by positional matching --- so it isn't all that surprising that positional matching does so well, but it remains unclear how accurate the testing cross-identifications are. This may be cleared up when Jesse Swan's cross-identifications are finalised.

We can estimate positional matching performance on FIRST by taking a subset of RGZ-FIRST and comparing positional matching cross-identifications to the RGZ cross-identifications. There are a few subsets we can consider based on how we choose to filter RGZ-FIRST. We can choose to include or exclude subjects with consensus level $\geq 0.65$, and we can choose to include or exclude subjects with no visible infrared host. We can also choose to not filter for either criteria. We tabulate the accuracies alongside the mean separation between the nearest neighbour and radio component for each subset:

| Consensus?       | Visible?         | #     | Accuracy (%) | Mean ($''$) |
|------------------|------------------|-------|--------------|-------------|
| N                | N                | 10765 | 7.6          | 6.98        |
| Y                | N                | 31712 | 7.8          | 6.67        |
|                  |                  | 93600 | 47.3         | 5.33        |
| N                | Y                | 12636 | 47.9         | 5.80        |
| Y                |                  | 73358 | 52.0         | 5.02        |
|                  | Y                | 54817 | 76.4         | 4.25        |
| Y                | Y                | 42181 | 84.9         | 3.79        |

Note that RGZ is biased against compact objects, so it's hard to say how these results might generalise to the whole of FIRST. Filtering on consensus level increases the accuracy, which makes sense because positional matching should work well in easier cases, and consensus should be higher for easier cases. Including subjects with no visible host massively decreases the accuracy. Note that positional matching will only select no host when there is no chance alignment, so there is a strong dependency on the separation cutoff radius for these cases. There is a strong linear correlation between accuracy and mean separation.

RGZ is radio-selected so let's assume that the rate at which infrared hosts are not visible generalises to the whole of FIRST. Of 73358 sources, 31963 have no host, or around 43.6% (which is very close to Ivy's numbers). What's the chance of a coincidental alignment? If we take a cutoff of $\theta$ for positional matching, then the chance of a coincidental alignment in an $a \times b$ field containing $K$ galaxies is

$$
    p(\text{chance alignment} \mid \theta) = 1 - \left(1 - \frac{\pi (1 - \cos \theta)}{2\ \mbox{arcsin}\left(\sin\left(a\right)\sin\left(b\right)\right)}\right)^K
$$

It would be useful to compare this formula to the estimates chance alignment by Norris+06:

| Angular separation | Chance of false cross-identification |
|--------------------|--------------------------------------|
| $1''$              | 3.9%                                 |
| $1''-2''$          | 11.5%                                |
| $2''-3''$          | 37.2%                                |

We will assume that SWIRE-CDFS is rectangular with side lengths $2.2^{\circ} \times 1.7^{\circ}$ (I can't find the actual size anywhere, so these are the dimensions of ATLAS-CDFS measured from the catalogue). Taking $K = 105000$, the approximate number of objects in SWIRE overlapping with ATLAS-CDFS, we obtain

\begin{align*}
    p(\text{chance alignment} \mid \theta = 1'') &= 0.002\\
    p(\text{chance alignment} \mid \theta = 3'') &= 0.02\\
    p(\text{chance alignment} \mid \theta = 5'') &= 0.04\\
    p(\text{chance alignment} \mid \theta = 10'') &= 0.16.
\end{align*}

These numbers are a lot lower than Ray's estimates. Simulating the problem by choosing 5000 random points in a subset of SWIRE-CDFS and cross-identifying them with positional matching gives similar results:

```
0.92% coincidental alignments <1''
3.13% coincidental alignments <2''
6.59% coincidental alignments <3''
16.49% coincidental alignments <5''
```

# Sliding window

In some sense cross-identification is an image problem --- we get a multiwavelength image of the sky and need to locate the host galaxy in that image. This is the computer vision task of *object localisation* (or *object detection*, which is a more common but less precise name). Arguably the most common approach to object localisation is the *sliding window*: Sweep a window across an image and classify each pixel as containing or not containing the object we want to find. This works well for e.g. finding cats on roads, and it ought to work just as well on the radio sky provided a sufficiently good classifier exists. This claim is borne out by our results (Alger+18), where we demonstrate that a sliding window method achieves good results at cross-identifying ATLAS when given a perfect classifier.

Assuming that every host galaxy is visible --- this is without loss of generality if you don't use physical galaxy properties in your classifiers --- then the sliding window approach can be interpreted as binary class probability estimation: Given a set of candidate host galaxies and a radio object, classify each candidate as the host of the radio object or not. This is problematic because for a given radio object there is exactly one positive example, a task for which training a classifier would be impossible. Instead, we split the class probability into a radio object-specific term, representing the probability that the candidate is associated with the specific radio object; and a general term, representing the probability that the candidate is *any* host galaxy. We can train the general term with a set of known host galaxies in a positive-unlabelled classification task (which I refer to as the *galaxy classification task*). The specific term might be cleverly chosen, or based on models or known distributions. We will write the specific term as $f_r$ and the general term as $f_g$. The host-finding function we described in the formalism section is then either

\begin{align*}
    &f : \mathcal R \to \mathcal G\\
    &f(r) = \underset{g \in \mathcal G}{\mbox{argmax}}\ f_r(r, g) f_g(g)
\end{align*}

or

\begin{align*}
    &f : \mathcal R \times \mathcal G \to \mathbb{R}\\
    &f(r, g) = f_r(r, g) f_g(g)
\end{align*}

depending on how tractable you like your functions. As discussed earlier, the former has a nicer form, so it's fortunate that we can approximate it by judicious choice of candidate host galaxies $\mathcal C(r)$:

\begin{align*}
    &f : \mathcal R \to \mathcal G\\
    &f(r) = \underset{g \in \mathcal C(r)}{\mbox{argmax}}\ f_r(r, g) f_g(g).
\end{align*}

This is the approach we take in Alger+18, choosing $\mathcal C(r)$ to be all galaxies within a fixed angular separation of $r$. This is also implicitly how RGZ works, as volunteers can only see a $3' \times 3'$ (for FIRST) or $2' \times 2'$ (for ATLAS) patch of sky in which to choose the host galaxy.

If we instead choose the latter form, we get some kind of probability that $r$ and $g$ are related. What are $f_r$ and $f_g$? If we look to the integer linear programming approach, then we might think of $f_g$ as the $\mbox{hostiness}$ function. $f_r$ describes some aspect of cross-identification somehow not intrinsic to the host galaxy. We can also use these functions to specify $S$, with $S_{ij} = f_r(r_i, g_j) f_g(g_j)$. Then

\begin{align*}
    \mbox{hostiness}(g) &= f_g(g) \sum_r f_r(r, g)\\
    \mbox{density}(r) &= \sum_g f_g(g) f_r(r, g)\\
    \mbox{confusion}(r) &= -\sum_g f_g(g) f_r(r, g) \left(\log f_g(g) + \log f_r(r, g)\right).\\
\end{align*}

For results and more detailed information, see Alger+18.

## Application to FIRST

Accuracy with a sliding window approach and a random forest classifier with pixel features is 91%, compared to 86% for positional matching. A centroid approach where the weighted mean of an image is matched to the nearest WISE object achieves an accuracy of 93%. PCA (50 components) + random forests gives an accuracy of 85% and a random 10-component projection + random forests gives 81%.

# Links to morphology

Radio Galaxy Zoo has two sub-tasks that volunteers must perform. These are radio morphology identification and cross-identification. Under some reasonable assumptions, these tasks are related. Assume we have a cross-identifier $f : \mathcal R \to \mathcal G$. Then obtaining the radio morphology information RGZ seeks is simple: Given a radio component $r$, the set of radio components in the same source is $\{r' \mid r' \in \mathcal R, f(r) == f(r')\}$. Going from a set of radio components to a host galaxy is harder (and not well-defined) but a reasonable approximation is the average location of all components in the set.
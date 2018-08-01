---
title: "Bipartite edge prediction"
author: \protect\parbox{\textwidth}{\protect\centering Matthew J. Alger\\ \small Cheng Soon Ong, Dawei Chen, Angus Gruen, and Jakub L. Nabaglo}
geometry: margin=3cm
latex-engine: xelatex
mainfont: EB Garamond
numbersections: true
---

This document is a summary of conversations and thoughts on bipartite edge prediction.

# Introduction

*Bipartite edge prediction* (*edge prediction*) is a prediction task that generalises many different tasks, including

- music playlist recommendation,
- radio host galaxy cross-identification,
- knowledge graphs, and
- record linkage.

These tasks can all be formalised as a bipartite graph with unknown edges: the task is to predict the unknown edges given the nodes. If some nodes have no known edges then the problem is called a *cold-start* problem.

As the tasks share a common formalism they also have common solutions. Some methods to solve edge prediction problems include

- matrix/tensor factorisation,
- binary classification,
- structured prediction,
- maximum likelihood, and
- integer linear programming.

We define a bipartite graph as a tuple $(A, B, E)$, where

- $A$ is a set of nodes,
- $B$ is a set of nodes with $A \cap B = \emptyset$, and
- $E \subset A \times B$ is a set of edges.

# Concrete examples

## Music playlist recommendation

Music playlist recommendation attempts to find songs that suitably extend existing playlists. We have a set of users $U$, a set of known and new songs $S_{\mathrm{old}}$ and $S_{\mathrm{new}}$ respectively, and a set of playlists by a user $u \in U, P_u \subset 2^S$. We can also define $P = \bigcup_{u \in U} P_u$ and $S = S_{\mathrm{old}} \cup S_{\mathrm{new}}$. One variant of the problem (*cold-playlist recommendation* in Chen+18) is an edge prediction problem for a graph $(U, S, E)$ where $E_{ij} = 1$ iff song $s_j$ should be recommended for inclusion in a new playlist for user $u_i$, or if the song is already in a playlist by user $u_i$. Another variant is *playlist extension*, where we recommend a song for inclusion in an extant playlist: here, the graph is $(P_u, S, E)$ with a fixed user $u \in U$, or simply $(P, S, E)$ if we ignore user information.

These are cold-start problems because we have no known edges for new songs, i.e., new songs aren't in any playlists.

## Radio host galaxy cross-identification

*Radio host galaxy cross-identification* (*cross-identification*) is the problem of associating observed radio objects with their infrared counterparts (*host galaxies*). We have a set of observed radio objects $R$ and a set of observed infrared objects $G$. This is trivially a bipartite graph $(R, G, E)$ with $E_{ij} = 1$ iff radio object $r_i$ is associated with infrared object $g_j$. We assume that for any radio object there is at most one host galaxy.

This is a cold-start problem because of the constraint that there is only one edge per radio object, meaning that we can't have any known edges for a given test radio object $r$ (or we've already solved the task!).

## Record linkage

*Record linkage* is the problem of identifying entities in two databases. We have two databases $D_1$ and $D_2$, and we want to find a set of equivalence classes $E \in D_1 \times D_2$, where $d_1 \sim d_2$ iff $d_1 \in D_1$ and $d_2 \in D_2$ represent the same entity. We assume that the databases are *deduplicated*, meaning that there is at most one record in a given database for a given entity.

The deduplication assumption means this is an even more constrained problem than cross-identification: any node (in any set) can have at most one edge.

# Prediction tasks

Here are some ways to formalise the prediction task. Here, $A$ and $B$ are the two disjoint sets of nodes, $E$ are edges, and $G$ is the set of graphs. $2^A$ is the set of all subsets of $A$. Note that $A$ and $B$ are interchangable so we will assume that any asymmetries can be reflected, i.e. if $\nu(A, B)$ is a formalism then $\nu(B, A)$ is also a formalism. We want to find a prediction function with one of these function types, which we give different names to distinguish between them:

1. $f : A \times B \to \mathbb R$
2. $g : A \times B \to \mathbb B$
3. $h : A \to B$
4. $q : A \to 2^B$
5. $r : 2^A \times 2^B \to 2^E$
6. $s : G \to G$
7. $t : G \to 2^E$

Let's look at how we might interpret and convert between these. In subsequent discussion, $a$ is an arbitrary element in $A$ and $b$ is an arbitrary element in $B$.

## $f : A \times B \to \mathbb R$

$f$ can be thought of as scoring each possible combination of $a$ and $b$, or scoring each edge. To get a concrete graph, we may then (for example) threshold the scores or choose the maximising edge(s) for each node.

There's not always a unique way to map between $f$ and the other functions, but here are some possible ways (that ignore additional constraints):

2. $g(a, b) = \Theta(f(a, b))$
3. $h(a) = \underset{b \in B}{\mathrm{argmax}}\ f(a, b)$
4. $q(a) = \{b \mid b \in B \land f(a, b) > \theta\}$
5. $r(A, B) = \{(a, b) \mid a \in A \land b \in B \land f(a, b) > \theta\}$
6. $s(A, B, E) = (A, B, E \cup \{(a, b) \mid a \in A \land b \in B \land f(a, b) > \theta\})$
7. $t(A, B, E) = E \cup \{(a, b) \mid a \in A \land b \in B \land f(a, b) > \theta\}$

We can also write some example implementations. Take $\phi_A$ and $\phi_B$ to be feature maps for $A$ and $B$ respectively. Then some implementations are:

- $f(x_a, x_b) = \sigma(\phi_A(x_a)^T W \phi_B(x_b))$
- $f(x_a, x_b) = \tilde f(x_a, x_b) \sigma(w \cdot \phi_B(x_b))$ (Alger+18)
- $f(x_a, x_b) = (w_a + w)^T \phi_B(x_b)$ (Chen+18)
- $f(x_a, x_b) = w_a^T \phi_B(x_b)$ (e.g. Joachims+09; this is multiclass classification with $|A|$ classes)

<!-- Alger+18 formalise this as binary classification by classifying all potential host galaxies as either hosts or not hosts with a classifier $f$. The probability that a radio object $r$ and a potential host galaxy $s$ are associated is

$$
    p(r, s) = f(s) \mathcal{N}(\mathrm{separation}(r, s) \mid 0, \sigma)
$$

where $\mathrm{separation}(x, y)$ is the angular separation of objects $x$ and $y$.

Define the set of radio objects $R$ and the set of infrared galaxies $S$.
 We can then define a set of edges $\hat E = \{(r, s) \mid s \text{ is the host galaxy of } r\}$, giving a bipartite graph of cross-identifications $(R, S, E)$.

The predicted set of edges $E$ can be found from the predicted probability $p(r, s)$:

$$
E = \left\{\left(r, \underset{s \in S}{\mathrm{argmax}}\ p(r, s)\right) \mid r \in R\right\}.
$$

This is a cold-start problem because of the constraint that there is only one edge per radio object, meaning that we can't have any known edges for a given test radio object $r$ (or we've already solved the task!).

## Music playlist recommendation

*Music playlist recommendation* (Chen+18) is the problem of generating playlists for users. There are four scenarios:

- *playlist extension*, where we are given a set of newly-released songs (i.e. not in any playlist) and want to extend existing playlists;
- *playlist creation*, where we are given a user (and their existing playlists) and we want to make a new playlist for them; and
- *new user playlist creation*, where we are given a new user (with no playlists) and we want to make a new playlist for them.

These tasks are challenging because the data are sparse (most songs aren't in many playlists and most users only have a few playlists) and noisy (feature noise plus users may make random playlists). The latter three tasks are cold-start tasks.

The simplest problem is playlist extension, so we will focus on that for now. A common approach to solving playlist extension is matrix factorisation. Let $S_{\mathrm{old}}$ be the set of songs in playlists and $S_{\mathrm{new}}$ be the set of newly-released songs. Let $S = S_{\mathrm{old}} \cup S_{\mathrm{new}}$ be the set of songs and $P$ be the set of playlists. Index them arbitarily and let $s_i \in S$ and $p_i \in P$. Define the solution recommendation matrix $\hat M \in \mathbb{R}^{|S| \times |P|}$:

$$
\hat M_{ij} = \begin{cases}1 & s_i \in p_j\\0 & s_i \notin p_j\end{cases}.
$$

Define the problem matrix $M \in \mathbb{R}^{|S| \times |P|}$ as a partially-observed variant of $\hat M$, where rows corresponding to new songs are unobserved:

$$
M_{ij} = \begin{cases}\hat M_{ij} & s_i \in S_{\mathrm{old}}\\? & s_i \in S_{\mathrm{new}}\end{cases}.
$$

Define a mask $J \in \{0, 1\}^{|S| \times |P|}$ such that $J_{ij} = 0 \iff M_{ij} = ?$. Note that $\hat M \odot J = M \odot J$ where $\odot$ is the elementwise matrix product.

We assume that $M$ can be factorised into two low-rank matrices $U \in \mathbb{R}^{|S| \times k}$ and $V \in \mathbb{R}^{|P| \times k}$:

$$
M \approx UV^T.
$$

We can think of $U_i$ as the features of $s_i$ and $V_i$ as the features of $p_i$. We then seek to minimise $\mathcal{L}(U, V)$:

$$
\mathcal{L}(U, V) = ||(M - UV^T) \odot J||.
$$
-->
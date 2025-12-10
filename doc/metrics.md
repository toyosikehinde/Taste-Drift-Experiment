# Metrics for Taste Drift

This document defines the core metrics used in the Taste Drift Experiment. The focus is on simple, interpretable quantities that can be computed from listening logs and semantic feature vectors.

---

## 1. Cosine Similarity and Cosine Drift

### 1.1 Cosine similarity between two vectors

Given two non-zero vectors \( \mathbf{a}, \mathbf{b} \in \mathbb{R}^d \), the cosine similarity is:

\[
\text{cos\_sim}(\mathbf{a}, \mathbf{b}) =
\frac{\mathbf{a} \cdot \mathbf{b}}{\|\mathbf{a}\| \, \|\mathbf{b}\|}
\]

where \( \mathbf{a} \cdot \mathbf{b} \) is the dot product and \( \|\mathbf{a}\| \) is the Euclidean norm (L2 norm).

If embeddings are L2-normalised beforehand, each vector has norm 1 and the formula simplifies to:

\[
\text{cos\_sim}(\mathbf{a}, \mathbf{b}) = \mathbf{a} \cdot \mathbf{b}
\]

Values are in the range \([-1, 1]\), but in practice, semantic embeddings often lie in a smaller interval such as \([0, 1]\).

### 1.2 Cosine drift between consecutive taste profiles

A user’s taste profile for time bucket \( t \) is represented by a centroid vector \( \mathbf{p}(u, t) \) in some embedding space (for example, lyrics embeddings). The primary measure of taste drift between two consecutive periods \( t \) and \( t+1 \) is defined as:

\[
\text{drift}(u, t, t+1) = 1 - \text{cos\_sim}(\mathbf{p}(u, t), \mathbf{p}(u, t+1))
\]

Small values (close to 0) indicate stability (profiles are similar), while larger values indicate stronger change in taste.

### 1.3 Tiny numeric example

Suppose a user’s centroid in period 1 is:

\[
\mathbf{p}(u, t) = (1, 0)
\]

and in period 2 is:

\[
\mathbf{p}(u, t+1) = \left(\frac{\sqrt{2}}{2}, \frac{\sqrt{2}}{2}\right)
\]

Both are unit length. The cosine similarity is:

\[
\text{cos\_sim} = 1 \cdot \frac{\sqrt{2}}{2} + 0 \cdot \frac{\sqrt{2}}{2} = \frac{\sqrt{2}}{2} \approx 0.707
\]

The drift is:

\[
\text{drift} = 1 - 0.707 \approx 0.293
\]

This corresponds to a moderate change in taste between the two periods.

---

## 2. KL-Divergence Over Theme or Genre Distributions

Cosine drift focuses on geometric distance in embedding space. A complementary view treats taste as a probability distribution over themes or genres and measures how that distribution changes over time.

### 2.1 Theme distributions

Assume that for each track we can assign a theme label (for example, via clustering embeddings or via mood tags). For a given user \( u \) and time period \( t \), we count how many tracks fall into each theme and normalise to obtain a probability distribution:

\[
P(u, t) = (p_1, p_2, \ldots, p_K)
\]

where \( p_k \ge 0 \) and \( \sum_{k=1}^K p_k = 1 \).

### 2.2 KL-divergence between consecutive distributions

To quantify how the theme distribution changes between periods \( t \) and \( t+1 \), we use Kullback–Leibler divergence:

\[
\mathrm{KL}\bigl(P(u, t) \,\|\, P(u, t+1)\bigr) =
\sum_{k=1}^K p_k(u, t) \log \frac{p_k(u, t)}{p_k(u, t+1)}
\]

To avoid division by zero, it is common to add a small smoothing constant \( \epsilon \) (for example, \( 10^{-8} \)) to probabilities before normalising.

KL-divergence is always non-negative and equals zero if and only if the two distributions are identical.

### 2.3 Tiny numeric example

Suppose the theme distribution for a user in two consecutive periods is:

\[
P(u, t) = (0.7, 0.3), \quad P(u, t+1) = (0.5, 0.5)
\]

Using natural logarithms:

\[
\mathrm{KL}(P(u, t) \,\|\, P(u, t+1)) =
0.7 \log\left(\frac{0.7}{0.5}\right)
+ 0.3 \log\left(\frac{0.3}{0.5}\right)
\]

\[
\approx 0.7 \log(1.4) + 0.3 \log(0.6)
\approx 0.7 \cdot 0.3365 + 0.3 \cdot (-0.5108)
\approx 0.2356 - 0.1532
\approx 0.0824
\]

A value of 0.0824 indicates that the theme distribution has changed, but not radically.

---

## 3. Summary

- Cosine drift captures how far a user’s centroid in embedding space moves between periods.
- KL-divergence captures how the mixture of themes or genres changes over time.
- Both metrics are complementary and can be computed using only aggregated information from listening logs and content embeddings.

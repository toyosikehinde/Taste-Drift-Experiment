# Taste Drift Experiment Design

This document provides the complete methodological design for a taste drift experiment using realistically available datasets. It describes how user listening histories and semantic content representations of tracks can be combined to analyse how a listener’s musical preferences evolve over time and to explore potential causes of these changes. This is a design blueprint rather than a fully implemented system, due to licensing and dataset availability constraints.

---

## 1. Motivation

Listener preferences are not static. Over weeks and months, musical tastes shift as individuals explore new genres, respond to emotional or social contexts, or engage with recommender systems. To understand this evolution, a listener’s musical taste can be represented as a point in a semantic embedding space and tracked over time. Movement in this space reflects changing preferences.

The purpose of the taste drift experiment is to measure how these preference representations change across time periods, and to examine what might cause those changes. This aligns with broader goals in recommendation research where temporal modelling, novelty seeking, and stability of preference are important considerations.

---

## 2. Available Datasets and Roles

This design uses datasets that are realistically accessible to researchers:

### User–History Datasets (Listening Logs)
Examples include:
- Last.fm 1K users dataset
- LFM-2b dataset

These datasets typically include: # Taste Drift Experiment Design

This document provides the complete methodological design for a taste drift experiment using realistically available datasets. It describes how user listening histories and semantic content representations of tracks can be combined to analyse how a listener’s musical preferences evolve over time and to explore potential causes of these changes. This is a design blueprint rather than a fully implemented system, due to licensing and dataset availability constraints.

---

## 1. Motivation

Listener preferences are not static. Over weeks and months, musical tastes shift as individuals explore new genres, respond to emotional or social contexts, or engage with recommender systems. To understand this evolution, a listener’s musical taste can be represented as a point in a semantic embedding space and tracked over time. Movement in this space reflects changing preferences.

The purpose of the taste drift experiment is to measure how these preference representations change across time periods, and to examine what might cause those changes. This aligns with broader goals in recommendation research where temporal modelling, novelty seeking, and stability of preference are important considerations.

---

## 2. Available Datasets and Roles

This design uses datasets that are realistically accessible to researchers:

### User–History Datasets (Listening Logs)
Examples include:
- Last.fm 1K users dataset
- LFM-2b dataset

These datasets typically include: user_id, timestamp, artist_name, track_name 


They provide chronological sequences of listening events, which are essential for constructing time-based taste profiles.

### Content and Lyrics Datasets (Semantic Track Representations)
Examples include:
- Spotify metadata datasets
- Lyrics extraction from publicly accessible APIs (when permitted)
- Precomputed lyrics embeddings from contextual models such as all-mpnet-base-v2

These datasets provide:
- Audio features
- Track and artist metadata
- Lyrics embeddings representing semantic meaning

For this design, lyrics embeddings form the semantic space in which taste drift is measured.

---

## 3. Data Alignment Strategy

User logs and content datasets originate from different sources and must be aligned. Alignment is achieved using track metadata:

1. Extract unique `(artist_name, track_name)` pairs from the listening log.
2. Extract corresponding fields from the content dataset.
3. Clean both sides by:
   - Lowercasing
   - Removing parentheses, descriptors, “remix” labels
   - Collapsing whitespace
4. Perform an inner join on cleaned artist and track names.
5. Build a mapping: track_id_from_listening_log → embedding_row_index


The result is a reduced but fully aligned dataset where each listening event corresponds to a track with a valid lyrics embedding.

---

## 4. Lyrics Semantic Space Construction

A contextual embedding model such as **all-mpnet-base-v2** is used to create the lyrics semantic space.

For each aligned track:

1. Retrieve lyrics text (if available)
2. Preprocess the lyrics:
- Lowercase
- Remove boilerplate repeats
- Join lines into a single string
3. Encode using the transformer to produce a vector: e_i ∈ ℝ^d
4. L2-normalize embeddings to simplify similarity calculations

This produces a mapping: track_id → lyrics_embedding

This vector space captures emotional, thematic, and narrative properties of songs and serves as the foundation for taste drift measurement.

---

## 5. User Taste Profiles Over Time

Using timestamps from the listening logs, user activity is segmented into discrete time buckets such as months.

For a given user **u** and time period **t**:

- Let **S(u, t)** be the set of tracks the user listened to in that period and for which embeddings exist.
- Each track **i** has an embedding vector **e_i**.

The user’s **taste profile** for that period is the centroid of embeddings:
\[
\mathbf{p}(u, t) = \frac{1}{|S(u, t)|} \sum_{i \in S(u, t)} \mathbf{e}_i
\]

Profiles are only computed if the number of tracks satisfies a minimum threshold (e.g., 10 plays). This prevents instability from very small samples.

This yields a chronological sequence:
\[
\mathbf{p}(u, t_1),\, \mathbf{p}(u, t_2),\, \ldots,\, \mathbf{p}(u, t_T)
\]
Each point represents the user’s dominant lyrical preference during that month.

---

## 6. Measuring Taste Drift

Taste drift between two consecutive periods **t** and **t+1** for user **u** is measured using cosine similarity:

\[
\text{cos\_sim}(u, t, t+1) =
\frac{\mathbf{p}(u, t) \cdot \mathbf{p}(u, t+1)}
{\|\mathbf{p}(u, t)\| \, \|\mathbf{p}(u, t+1)\|}
\]

Drift is defined as:
\[
\text{drift}(u, t, t+1) = 1 - \text{cos\_sim}(u, t, t+1)
\]

Low drift means stable lyrical preferences.  
High drift indicates meaningful semantic change.

### Optional Alternative Measure
Tracks can be clustered into lyrical themes. For each period, compute a theme distribution **P(u, t)**, then compute:
\[
\mathrm{KL}(P(u, t) \| P(u, t+1))
\]

This captures changes in dominant themes rather than geometric shifts.

---

## 7. Analysing Causes of Taste Drift

To understand why drift occurs, the experiment designs several explanatory variables:

### Within-Period Diversity
Average pairwise distance between tracks in **S(u, t)**.  
Higher diversity can precede larger drift.

### Novelty
Average distance between new tracks in period **t+1** and the previous profile **p(u, t)**.  
Higher novelty may indicate exploratory behaviour.

### Shifts in Genre or Mood
If genre tags exist, measure:
- Change in distribution of genres between periods
- Change in mood polarity or sentiment

### Temporal Context
Seasonal or situational factors derived from timestamps (e.g., summer, exam periods) can shape listening behaviour.

### Modelling Drift
For each transition:
\[
\text{drift}(u, t, t+1)
\]
is treated as the dependent variable in a regression or mixed-effects model with predictors including novelty, diversity, theme shifts, and temporal covariates.

This allows hypothesis testing about what drives preference change.

---

## 8. Limitations and Status of This Repository

A fully implemented drift system requires access to:
- Licensed full lyrics
- A dataset that combines user logs and lyrics
- Rights to store or redistribute lyrics

Such a combined dataset is not available within this repository’s constraints. Therefore, this document serves as a complete experiment design, aligned with available public datasets such as Last.fm logs and Spotify-like content corpora, while avoiding licensing issues.

The repository focuses on documentation and conceptual pipelines rather than a full production system. The methodology presented here can be implemented in future work once appropriate datasets are accessible.

---

## 9. Relationship to Other Files in This Repository

- `log_schema.md` describes the minimal structure of listening histories required to run the experiment.
- `metrics.md` may contain explicit formulas and short mathematical examples for cosine drift and KL divergence.
- `implementation.md` is reserved for pseudocode that mirrors this design using synthetic or placeholder data.

This document (`experiment.md`) is the authoritative reference for the experiment design.







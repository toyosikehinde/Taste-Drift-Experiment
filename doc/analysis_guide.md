# Analysis Guide

# Taste Drift Analysis — Unified Documentation

This document provides a complete, self-contained reference for implementing and analysing taste drift using listening logs and semantic embeddings. It merges the contents of:  
1. Metrics  
2. Minimal implementation snippets  
3. Logging schema  
4. Analysis walkthrough  

Everything needed to understand or prototype the taste drift experiment is contained in this single file.

---

# 1. Metrics for Taste Drift

This section defines the mathematical metrics used to compute taste drift and changes in listener preference over time.

## 1.1 Cosine Similarity

Given two feature vectors \( \mathbf{a}, \mathbf{b} \in \mathbb{R}^d \):

\[
\text{cos\_sim}(\mathbf{a}, \mathbf{b}) =
\frac{\mathbf{a} \cdot \mathbf{b}}
{\|\mathbf{a}\| \, \|\mathbf{b}\|}
\]

If embeddings are L2-normalised:

\[
\text{cos\_sim} = \mathbf{a} \cdot \mathbf{b}
\]

## 1.2 Cosine Drift

A user’s taste profile for a period \( t \) is the centroid embedding \( \mathbf{p}(u, t) \).  
Taste drift between two periods is:

\[
\text{drift}(u, t, t+1) = 1 - \text{cos\_sim}(\mathbf{p}(u, t), \mathbf{p}(u, t+1))
\]

Values close to 0 indicate stability; larger values indicate change.

## 1.3 KL-Divergence for Theme Drift

If each period has a theme distribution \( P(u, t) \):

\[
\mathrm{KL}(P(u, t) \| P(u, t+1)) = 
\sum_{k} p_k \log \frac{p_k}{q_k}
\]

with epsilon smoothing to prevent division by zero.

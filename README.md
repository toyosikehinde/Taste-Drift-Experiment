# Taste-Drift-Experiment
*A conceptual and computational framework for analysing how musical preferences evolve over time.*

This repository presents a complete design blueprint for a **Taste Drift Experiment**, inspired by temporal preference modelling in recommender systems research. The goal of this project is to examine how a listener’s taste changes across time using listening logs and semantic song embeddings, and to explore potential drivers of those shifts.

Although a full implementation requires access to timestamped listening logs and licensed lyrics datasets, this repository documents the methodology, metrics, data schema, and minimal code needed to prototype the experiment on synthetic or privately-held datasets.

---

## 📌 Project Overview

Musical taste is not static. People move through different themes, moods, and genres as life changes. This project models musical taste as a **trajectory in semantic space** by:

- Building feature representations of songs (lyrics embeddings, audio descriptors, or hybrid vectors).  
- Constructing **time-based taste profiles** for each user.  
- Measuring **taste drift** using cosine distance and KL-divergence.  
- Exploring **possible causes**, such as novelty, diversity, genre shifts, or contextual factors.  

The full experiment design is documented in the `doc/` folder.

---

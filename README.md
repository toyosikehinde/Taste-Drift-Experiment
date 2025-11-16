# Taste-Drift-Experiment

This repository documents an experiment to track how a listener’s musical taste evolves over time and to test whether recommendations influence that evolution. It combines a narrative report with light, self-contained code snippets that illustrate how to compute a listener taste vector, daily cosine drift in acoustic feature space, and KL-divergence over genre distributions.

**What this is:** documentation + reference snippets.  
**What this is not:** a production system or full app.

## Contents
- `experiment.md` — full narrative (methods + equations)
- `metrics.md` — cosine drift and KL-divergence explained
- `implementation.md` — minimal code blocks to reproduce core calculations
- `log_schema.md` — tiny logging spec for impressions and interactions
- `analysis_guide.md` — how to run a small analysis on the sample files
- `snippets/` — importable .py helpers used in the docs

## Minimal setup
```bash
pip install -r requirements.txt

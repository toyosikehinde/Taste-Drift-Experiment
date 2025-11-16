# Metrics

**Cosine drift.** 1 - cosine similarity between consecutive taste vectors. Captures *how the sound signature moves*. Robust to scale; interprets as “change of vibe.”

**KL-divergence.** Asymmetric measure of change between categorical distributions (e.g., weekly genre mix). Captures *what kinds of songs* changed. Use smoothed probabilities; optionally report the symmetric Jensen–Shannon divergence for stability.

**Net displacement.** \(1 - \cos(v_T, v_0)\). Long-term movement from start to end.

**Out-of-profile adoption.** Share of accepted tracks whose cosine similarity to the pre-exposure \(v_t\) is below a threshold \(\tau\). Indicates genuine exploration.

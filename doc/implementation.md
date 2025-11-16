# Minimal Implementation Snippets

These examples are educational and self-contained. Vectors should be standardized with the same scaler used for track features.

```python
# snippets/profiles.py
import numpy as np

class UserProfile:
    def __init__(self, D, alpha=0.1):
        self.v = np.zeros(D, dtype=float)
        self.n = 0
        self.alpha = float(alpha)

    def update_on_accept(self, x_i):
        if self.n == 0:
            self.v = x_i.astype(float).copy()
        else:
            self.v = (1 - self.alpha) * self.v + self.alpha * x_i
        norm = np.linalg.norm(self.v) + 1e-12
        self.v = self.v / norm
        self.n += 1

# snippets/drift_metrics.py
import numpy as np

def cosine(u, v):
    num = float(np.dot(u, v))
    den = float(np.linalg.norm(u) * np.linalg.norm(v) + 1e-12)
    return num / den

def cosine_drift(v_prev, v_curr):
    return 1.0 - cosine(v_prev, v_curr)

def kl_divergence(p, q, eps=1e-10):
    p = np.asarray(p, dtype=float); q = np.asarray(q, dtype=float)
    p = p / (p.sum() + eps); q = q / (q.sum() + eps)
    return float(np.sum(np.where(p > 0, p * np.log((p + eps)/(q + eps)), 0.0)))


# snippets/quick_eval.py
import json, numpy as np
from .profiles import UserProfile
from .drift_metrics import cosine_drift, kl_divergence

def daily_cosine_drift(user_vectors_by_day):
    # dict date -> v_t (np.array)
    dates = sorted(user_vectors_by_day.keys())
    out = []
    for d_prev, d_curr in zip(dates[:-1], dates[1:]):
        out.append((d_curr, cosine_drift(user_vectors_by_day[d_prev],
                                         user_vectors_by_day[d_curr])))
    return out

def weekly_kl_divergence(genre_hist_by_week):
    # dict 'YYYY-WW' -> list/array of genre probs
    weeks = sorted(genre_hist_by_week.keys())
    out = []
    for w_prev, w_curr in zip(weeks[:-1], weeks[1:]):
        out.append((w_curr, kl_divergence(genre_hist_by_week[w_curr],
                                          genre_hist_by_week[w_prev])))
    return out

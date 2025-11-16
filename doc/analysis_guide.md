# Analysis Guide

1) Place your standardized track vectors and derived user vectors in `data/`.  
2) Compute daily cosine drift from saved `user_vec` snapshots; compute weekly KL-divergence from a genre tally per week.  
3) Compare control vs explore by grouping on `policy` and averaging drift.

Example:
```python
import json, pandas as pd, numpy as np
from snippets.drift_metrics import cosine_drift, kl_divergence

# Load logs
logs = [json.loads(line) for line in open("data/sample_logs.jsonl")]
df = pd.DataFrame(logs)

# Daily cosine drift per user
df['date'] = pd.to_datetime(df['t']).dt.date
v_t = (df.groupby(['user_id','date'])['user_vec']
         .last().apply(np.array))  # last snapshot per day
drifts = []
for uid, series in v_t.groupby(level=0):
    dates = list(series.index.get_level_values('date').unique())
    dates.sort()
    for d0, d1 in zip(dates[:-1], dates[1:]):
        drifts.append({'user_id': uid,
                       'date': d1,
                       'cosine_drift': cosine_drift(series[d0], series[d1])})
pd.DataFrame(drifts).to_csv("data/daily_drift.csv", index=False)


---

## snippets/profiles.py

```python
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
        self.v /= (np.linalg.norm(self.v) + 1e-12)
        self.n += 1

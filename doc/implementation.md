# Minimal Implementation 

This document provides minimal, self-contained code snippets that illustrate how to compute taste profiles and drift metrics using user listening logs and precomputed track feature vectors. The code is illustrative and uses generic CSV filenames and column names.

All examples assume:

- Python 3
- `pandas`
- `numpy`

Install requirements with:

```bash
pip install -r requirements.txt

user_id, track_id, timestamp

and an embeddings file where track_id can be mapped to a row index in an array of shape (n_tracks, d).

import pandas as pd
import numpy as np
import json

# Load listening log
log_df = pd.read_csv("data/listening_log.csv")
log_df["timestamp"] = pd.to_datetime(log_df["timestamp"], utc=True)

# Load embeddings (for example, lyrics or hybrid embeddings)
embeddings = np.load("data/track_embeddings.npy")  # shape (n_tracks, d)

# Mapping from track_id to row index in embeddings
with open("data/track_id_to_idx.json", "r") as f:
    track_id_to_idx = json.load(f)

# MONTHLY EMBEDDING COMPUTATION
from typing import Dict, List

def add_month_bucket(df: pd.DataFrame,
                     time_col: str = "timestamp") -> pd.DataFrame:
    df = df.copy()
    df["month"] = df[time_col].dt.to_period("M").dt.to_timestamp()
    return df

log_df = add_month_bucket(log_df)

def compute_user_month_profiles(df: pd.DataFrame,
                                embeddings: np.ndarray,
                                track_id_to_idx: Dict[str, int],
                                user_col: str = "user_id",
                                track_col: str = "track_id",
                                bucket_col: str = "month",
                                min_tracks: int = 5) -> pd.DataFrame:
    rows: List[dict] = []
    grouped = df.groupby([user_col, bucket_col])

    for (user, month), group in grouped:
        idxs = []
        for tid in group[track_col]:
            if tid in track_id_to_idx:
                idxs.append(track_id_to_idx[tid])

        if len(idxs) < min_tracks:
            continue  # skip buckets with too few tracks

        arr = embeddings[np.array(idxs)]
        centroid = arr.mean(axis=0)

        rows.append({
            user_col: user,
            bucket_col: month,
            "profile": centroid,
            "n_tracks": len(idxs),
        })

    return pd.DataFrame(rows)

profiles_df = compute_user_month_profiles(
    log_df,
    embeddings=embeddings,
    track_id_to_idx=track_id_to_idx,
)

# COMPUTE COSINE DRIFT BETWEEN CONSECUTIVE MONTHS
def cosine_similarity(a: np.ndarray, b: np.ndarray) -> float:
    denom = np.linalg.norm(a) * np.linalg.norm(b)
    if denom == 0:
        return 0.0
    return float(np.dot(a, b) / denom)

def compute_cosine_drift(profiles: pd.DataFrame,
                         user_col: str = "user_id",
                         bucket_col: str = "month") -> pd.DataFrame:
    rows: List[dict] = []

    for user, group in profiles.groupby(user_col):
        group = group.sort_values(bucket_col).reset_index(drop=True)
        if len(group) < 2:
            continue

        for i in range(len(group) - 1):
            p_prev = group.loc[i, "profile"]
            p_next = group.loc[i + 1, "profile"]

            cos_sim = cosine_similarity(p_prev, p_next)
            drift = 1.0 - cos_sim

            rows.append({
                user_col: user,
                "t_prev": group.loc[i, bucket_col],
                "t_next": group.loc[i + 1, bucket_col],
                "cos_sim": cos_sim,
                "drift": drift,
                "n_prev": group.loc[i, "n_tracks"],
                "n_next": group.loc[i + 1, "n_tracks"],
            })

    return pd.DataFrame(rows)

drift_df = compute_cosine_drift(profiles_df)
drift_df.head()

#COMPUTING SIMPLE THEME DISTRIBUTION AND KL-DIVERGENCE
from collections import Counter

# Example: a small mapping from track_id to theme label
# In practice, this might be loaded from a CSV or derived from clustering.
track_to_theme = {
    "track_1": "happy",
    "track_2": "sad",
    "track_3": "party",
    # ...
}

def theme_distribution(df: pd.DataFrame,
                       track_col: str = "track_id") -> Dict[str, float]:
    labels = []
    for tid in df[track_col]:
        if tid in track_to_theme:
            labels.append(track_to_theme[tid])
    if not labels:
        return {}

    counts = Counter(labels)
    total = sum(counts.values())
    return {k: v / total for k, v in counts.items()}

def kl_divergence(p: Dict[str, float],
                  q: Dict[str, float],
                  eps: float = 1e-8) -> float:
    # make sure all keys are in both distributions
    keys = set(p.keys()) | set(q.keys())
    kl = 0.0
    for k in keys:
        p_k = p.get(k, 0.0) + eps
        q_k = q.get(k, 0.0) + eps
        kl += p_k * np.log(p_k / q_k)
    return float(kl)


---


 

---

# Logging Schema for Taste Drift Analysis

This document defines a minimal logging schema that would allow the Taste Drift Experiment to be run in a real system. It focuses on the data needed to reconstruct user taste profiles and analyse drift over time.

---

## Core Listening Log

At minimum, the system must log one record per listening event (or per completed play) with the following fields:

```text
user_id        string   Anonymised user identifier.
track_id       string   Unique identifier for the track.
artist_name    string   Human-readable artist name.
track_name     string   Human-readable track name.
timestamp      datetime UTC timestamp of when the track was played.
source         string   Optional: source of play (e.g., "search", "radio", "playlist").
context_id     string   Optional: identifier for session, playlist, or station.


# CONTENT EMBEDDINGS AND METADATA
track_id       string   Unique track identifier (same as in listening log).
artist_name    string   Artist name.
track_name     string   Track name.
album_name     string   Optional: album title.
genre          string   Optional: coarse genre label.
lyrics_id      string   Optional: key to a lyrics store.
embedding_id   string   Optional: key to an embedding store.

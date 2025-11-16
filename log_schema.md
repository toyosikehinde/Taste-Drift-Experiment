---

## docs/log_schema.md

```markdown
# Minimal Logging Schema

Each impression or interaction row (CSV or JSONL):

- `t` (ISO timestamp)
- `user_id`, `session_id`, `track_id`
- `policy` ("control" | "explore")
- `event` ("play_30s" | "replay" | "like" | "save" | "skip" | null)
- `user_vec` (array before exposure; same dim as track vectors)
- `track_vec` (array for the shown track)
- optional: `rank`, `dwell_s`

Sample JSONL:
{"t":"2025-11-15T14:00:00Z","user_id":"u1","session_id":"s1","track_id":"t42","policy":"explore","event":"save","user_vec":[...],"track_vec":[...]}

**Tracking User Taste Drift in Music Recommendation Systems**

Understanding how a listener’s musical preferences evolve over time is central to evaluating both personalization and influence within a recommender system. This experiment proposes a framework for modeling and observing taste drift the gradual movement of a listener’s “center of musical gravity” across time as they replay, like, or save songs. The objective is to determine whether recommendations simply reinforce what a listener already enjoys or play an active role in shaping new preferences.

**Conceptual Framework**
Each track in the dataset is represented as a feature vector containing its acoustic and timbral descriptors such as MFCC means, spectral centroid, roll-off, flatness, and tempo. Each listener is modeled by a taste vector (vₜ), which summarizes the kind of sounds they are currently drawn to based on their interactions. When a user engages positively with a new track by replaying it, listening for more than thirty seconds, liking, or saving it nudges their taste vector slightly toward that song’s coordinates in the feature space.

The update rule follows an exponential moving average model:

Vt+1 =(1−α)⋅Vt +α⋅xi

Here, xᵢ is the feature vector of the accepted track and α (alpha) is a memory parameter that controls how much influence new songs have compared to previous ones. A small α keeps the listener’s profile stable, reflecting consistent listening habits, while a larger α models faster-changing preferences, capturing more exploratory behavior. Over time, vₜ becomes a concise, continuous representation of a listener’s evolving taste.

**Quantifying Change**

To measure how much a listener’s taste has changed, the experiment applies two complementary metrics: Cosine Drift and Kullback–Leibler (KL) Divergence.

**Cosine Drift (Acoustic Directional Change)**

Cosine drift quantifies how the overall “direction” of a listener’s taste vector changes between sessions. It is computed as:

Driftt =1−cos(Vt ,Vt−1 )

This metric captures acoustic or timbral shifts such as moving from mellow Afrobeats to bright house, or from R&B to jazz-infused electronica. A low drift value means the listener’s sonic preferences are consistent, while higher values indicate notable stylistic transitions. Aggregating these values across sessions or weeks provides a timeline of how stable or dynamic a listener’s listening patterns are.

**KL-Divergence (Categorical Distributional Change)**

While cosine drift operates in continuous feature space, KL-divergence measures how the distribution of categorical preferences (e.g., genres, moods, or regions) changes over time. It is defined as:

DKL (P∥Q)=i∑ P(i)logQ(i)P(i)
​	
Here, P(i) represents the proportion of genres in the current listening window, and Q(i) represents the proportions from the previous period. A small KL value indicates stability listeners are staying within familiar genres while a larger value signals exploration into new categories. This measure adds a semantic dimension to the drift analysis by highlighting what kinds of songs have changed rather than how they sound.

Together, cosine drift and KL-divergence provide a dual perspective: cosine drift reveals shifts in sound and production style, while KL-divergence reveals shifts in musical identity, genre, or mood.

**Experimental Design**

To test whether recommendations influence taste evolution, the experiment introduces two groups of listeners:
* Control group: Receives standard, nearest-neighbor recommendations based on their current taste vector (vₜ). These recommendations reinforce existing listening patterns.
* Exploration group: Receives a blend of familiar and novel tracks that differ in tempo, harmonic texture, or cultural origin. This group tests whether diversity in recommendations expands taste boundaries.

All listening events are logged, capturing user_id, timestamp, track_id, policy (control or explore), and engagement type (play ≥30s, replay, like, save, skip). Alongside these, snapshots of vₜ before each exposure and the feature vector of the track are stored. This structured logging enables longitudinal analysis of how each user’s taste vector and genre distribution evolve.

If the exploration group shows higher cosine drift and KL-divergence values especially when paired with increased positive engagement and replays it suggests that the system is not only predicting existing preferences but actively broadening them.

**Interpretation**

Short-term drift (daily or session-based) often reflects contextual changes such as mood or environment, while long-term drift (over weeks or months) reflects sustained preference shifts. The balance between the two helps characterize listener types some users maintain a steady sonic identity, while others exhibit cyclical or exploratory listening patterns.

Visualizing both drift metrics as time-series curves can reveal these dynamics at a glance: cosine drift tracks acoustic movement through feature space, while KL-divergence captures the categorical reshaping of a listener’s profile. These trajectories together form a “taste map” that illustrates how the recommender interacts with human musical evolution.

**Ethical and Analytical Outlook**

From an ethical perspective, tracking taste drift provides insight into algorithmic influence on cultural consumption. Systems designed for exploration should encourage diversity and serendipity without steering listeners too forcefully. Monitoring both metrics over time ensures a balance between familiarity and discovery allowing users to remain the active agents of their own musical journeys.

By combining continuous acoustic modeling (cosine drift) with discrete distributional analysis (KL-divergence), this experiment establishes a quantitative yet human centered method for observing how listening preferences evolve and how recommendation systems might amplify, stabilize, or transform them.

Tracking User Taste Drift in Music Recommendation Systems

Understanding how a listener’s musical preferences evolve over time is central to evaluating both personalization and influence within a recommender system. This experiment introduces a framework for quantifying taste drift - the way a user’s listening center of gravity moves through the feature space as they interact with songs over days or months. The objective is to determine whether the recommender merely mirrors existing preferences or actively shapes them.

Each track in the dataset is represented as a feature vector, capturing timbral, rhythmic, and spectral information such as MFCC means, spectral centroid, roll-off, and tempo. Each listener, in turn, is modeled by a taste vector (vₜ), representing the weighted average of the audio features from the songs they have positively engaged with (through replays, saves, or likes). The taste vector updates dynamically according to an exponential moving average rule:

Vt+1 =(1−α)⋅Vt +α⋅xi

Here, xᵢ is the feature vector of a newly accepted track, and α (alpha) controls how strongly new interactions influence the listener’s taste. A smaller α value makes the profile stable and slow to adapt, while a larger α value allows faster changes that reflect exploratory or transient moods. This formulation mirrors the natural way people form musical habits, where repeated exposure gradually reshapes their sense of “familiar sound.”

To measure how much a listener’s taste changes between sessions, the experiment applies two complementary metrics: Cosine Drift and KL-Divergence.

Cosine Drift captures the directional change in the listener’s taste vector, indicating shifts in the overall acoustic style or timbral signature of the songs they engage with. It is computed as:

Driftt =1−cos(Vt ,Vt−1 )

A low value of Driftₜ implies a consistent listening pattern, whereas higher values reveal notable changes, such as transitions from smooth Afrobeats to energetic house tracks or from acoustic jazz to synthesized R&B. Over longer timescales, cumulative cosine drift provides an estimate of how far a listener’s sonic identity has migrated in the feature space since the beginning of observation.

KL-Divergence (Kullback–Leibler divergence) measures distributional change in a listener’s preferences across categorical dimensions such as genre, mood, or region. It compares how the proportions of these categories differ between two periods:
DKL (P∥Q)=i∑ P(i)logQ(i)P(i)

Here, P(i) represents the genre distribution for the current time window and Q(i) represents the previous one. A low KL value indicates that the listener’s genre balance is stable, while a higher value shows that they are engaging with new categories or moving into unexplored regions of sound. Together, cosine drift and KL-divergence provide a nuanced view: one tracks how the sound signature changes, while the other measures what kinds of songs are driving that change.

To test whether the recommender itself contributes to these shifts, the experiment compares two groups of listeners. The control group receives standard nearest-neighbor recommendations similar to their current taste vector, while the exploration group is exposed to a mixture of familiar and diverse tracks that differ in tempo, harmonic color, or timbral space. Interaction data such as play duration, replays, skips, likes, and saves are logged alongside snapshots of each listener’s taste vector and the features of the songs they were shown. By comparing changes in cosine drift and KL-divergence across both groups, the system can estimate whether exploration-oriented recommendations genuinely expand the listener’s taste or simply create short-term novelty.

Daily analysis captures short-term context shifts, such as mood-driven listening changes, whereas cumulative metrics over weeks or months reveal enduring transformations in preference. The framework also supports ethical and fairness assessments, ensuring that the algorithm respects listener autonomy and diversity by balancing discovery with relevance. In this way, the system evolves from predicting what a listener might enjoy to understanding how their musical identity evolves through continued interaction.

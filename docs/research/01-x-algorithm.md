# How X delivers the For You feed

Researched 2026-10-05. Source tiers used throughout:

- **[primary]** read directly from X's published code (`xai-org/x-algorithm`: README, `home-mixer/params/param.rs`, `xai-value-model/scoring.rs`).
- **[secondary]** blogs and news articles.
- **[inference]** our own reading. Not verified.

Weights change roughly every four weeks (the repo promises periodic updates). Everything below is a dated snapshot. The `param.rs` header says: "mirrored from config feature-switch defaults; last sync 2026-10-02". Production values may differ from these defaults. A future `x-algo-refresh` skill should re-read the repo and regenerate this file.

## The pipeline [primary]

For You is assembled per request by `home-mixer`. Four things happen, in order.

| Stage | What happens |
|---|---|
| **1. Find** | Query hydration loads the viewer's recent actions (the main model input), following list, blocks, mutes, muted keywords, seen posts. Candidate sources run in parallel: **Thunder** (recent posts from followed accounts, in memory), **Phoenix retrieval** (posts nearest the viewer in embedding space), **SimClusters** (clusters of accounts and posts by who engages with what). |
| **2. Filter (pre-scoring)** | Duplicates, posts older than **48 hours**, the viewer's own posts, reposts/replies from accounts the viewer does not follow (`OONRetweetReplyFilter`), blocked/muted accounts, muted keywords, already seen or served, subscriber-only posts, topic filters, `NewUserMinEngagementFilter` (new accounts only, out-of-network, default disabled), holdout filters. |
| **3. Rank** | **Phoenix** (a Grok-based transformer) predicts, for this viewer, the probability of each action on each post. `RankingScorer` computes `score = sum(weight_i x P(action_i))`, then applies three adjustments (below). Candidates cannot attend to each other during inference, so one post's score does not depend on the rest of the batch. |
| **4. Blend** | `VMRanker` re-orders the scored posts with a determinantal point process over embeddings: it gives up a little score for less similarity between neighbours. Post-selection filters (`VFFilter`, `AncillaryVFFilter`, `DedupConversationFilter`) apply visibility decisions. The blending pipeline then adds ads, Who-to-Follow and prompts. |

Visibility (whether a post may be shown at all) is decided separately from ranking, by `visibility-filtering/` using the viewer's blocks/mutes and labels from other systems (`scarecrow`, `botmaker`, abuse services). Some rules are deliberately not published.

### Three post-score adjustments [primary]

- **Author diversity:** each post after an author's first is multiplied by a decaying factor down to a floor. Formula in `scoring.rs`: `(1 - floor) * decay^k + floor`. **Production decay and floor values were not found.** The numbers in the code's unit test (decay 0.5, floor 0.25) are test fixtures, not production.
- **Out-of-network discount:** posts from accounts the viewer does not follow are multiplied by a factor below 1. Replies and reposts from followed accounts are also discounted. Production factor not found.
- **New-author boost:** posts from authors whose impressions are below a threshold are lifted toward a target position. `LowImpressionsMaxPositionRatio = 0.97`. Threshold not found.

## Weights [primary, defaults synced 2026-10-02]

Weights multiply the **predicted probability** of the action for each viewer, not raw counts. The code comments say this explicitly: reading "report = 468x like" as "one report cancels 468 likes" is wrong. Baseline report probability is more than 1000x lower than a like, so it is weighted up to matter at all. Recommendations are personalized, so reports from bad actors mostly affect viewers similar to those actors.

Only actions on a post **served in the Home timeline** count. Navigating to a post directly (for example from a group chat) has no ranking impact.

| Positive action | Weight |
|---|---|
| Share via copy-link | **20.0** |
| Reply | 5.0 (+15.0 when viewer and author follow each other) |
| Quote | 5.0 |
| Share via DM | 5.0 |
| Follow author | 4.0 |
| Share | 2.0 |
| Repost | 1.0 |
| Like | 0.5 |
| Click | 0.3 |
| Open link | 0.2 |
| Video open | 0.07 |
| Photo expand | 0.05 |
| Dwell | 0.05 |
| Quoted-post click | 0.05 |
| Post unexplored | 0.02 (appears to apply to in-network posts only) |
| Continuous dwell time | 0.004 |
| Profile click, video quality view, quoted VQV | 0.0 |

| Negative action | Weight |
|---|---|
| Report | **-234.0** |
| Mute author | -58.8 |
| Not interested | -47.52 |
| Block author | -31.2 |
| Not dwelled | -0.02 |

Other defaults worth knowing: `EnablePhoenixOonReplies = false`, `EnableSimclustersSource = true`, `EnableTweetMixerSource = false`, `EnablePopularPostsSource = false`, `PhoenixMaxResults = 1000`, `ThunderMaxResults = 1200`, `PhoenixColdStartMaxResults = 200`.

## What the code does NOT confirm

- **Stale numbers in blogs [secondary].** Many 2026 growth blogs quote reply-back = 75, reply = 13.5, bookmark = 10, repost = 1, like = 0.5, block = -3. Those come from the 2023 `twitter/the-algorithm` release. One of them admits the production weights are not fully published and calls the 2023 numbers "directionally accurate". They do not match the current file (reply is 5.0, not 13.5 or 75; no bookmark weight was found in the weights retrieved).
- **External link penalty [unverified].** Blogs claim a 30-50% (or up to 94%) reach penalty for posts with links and say "the open-source code confirms" it. It was **not found** in the README or in the weights retrieved. "Open link" carries a small positive weight (0.2). Not every file was read. Treat as unverified. Our own measurement on 28 accounts (see `02-ai-accounts-study.md`) found only about -10%.
- **Reply-first growth [secondary, hypothesis].** Growth blogs recommend spending most time replying to larger accounts. Note `EnablePhoenixOonReplies = false` in the mirrored defaults, so replies from non-followed accounts may not enter For You by default. If so, reply-first growth works through thread visibility, notifications and profile visits, not the For You feed [inference, low confidence].
- **"Verified accounts get 2-4x reach" [secondary].** Not checked against code.

## Implications for a skill [inference built on the primary facts]

1. Every post is scored per viewer. A post needs a specific audience cluster, not "everyone".
2. The high-value actions are share-by-copy-link (20), reply (5, up to 20 with a mutual follow), quote (5), DM share (5), follow (4). Likes are the weakest positive.
3. Polarizing posts risk "not interested" (-47.5), mute (-58.8) and report (-234) from the viewers they annoy.
4. Posts live about 48 hours in For You.
5. Posting similar topics back to back runs into the diversity re-rank, and many posts from one author run into the repeat-author decay.
6. New accounts get a new-author boost for low-impression authors. The threshold is unknown.
7. Weights drift, so hardcoding them goes stale. Read them from the repo.

## Sources

- https://github.com/xai-org/x-algorithm (README, notable updates 2026-08-13, 2026-08-14, 2026-09-18)
- https://raw.githubusercontent.com/xai-org/x-algorithm/main/home-mixer/params/param.rs
- https://raw.githubusercontent.com/xai-org/x-algorithm/main/xai-value-model/scoring.rs
- https://techcrunch.com/2026/08/13/x-open-sources-its-ranking-algorithm-letting-users-see-if-theyve-been-shadowbanned/ (transparency tool pilot for accounts at least a year old; some Grok-based violation systems not released)
- https://alexberman.com/x-algorithm-github (source of the stale 2023 weights table)

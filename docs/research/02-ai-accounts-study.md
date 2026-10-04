# What works for AI accounts on X (observed data)

Run 2026-10-05. Question: which AI accounts on X do well without being employees of frontier labs (DeepMind, OpenAI, Anthropic, xAI and similar), what do they post, and what do they have in common?

## Method and limits

- **Data:** for each of 28 handles, the public embed endpoint (`syndication.twitter.com/srv/timeline-profile/screen-name/<handle>`) returned about 100 posts with text, like count, reply count, date, quote flag, media and URL entities. No login. This endpoint is undocumented and may break or be restricted at any time.
- **Selection:** the 28 handles were chosen from list articles and my own knowledge, not at random. No lab employees are included. Some are employees of ordinary companies (Roboflow, Databricks) or run their own companies.
- **Skew:** the endpoint returns about 100 posts per account spanning several years (2018 to 2026), skewed toward popular posts. Medians and "top posts" therefore describe what wins, not an average day.
- **Likes only.** No follower-growth data, no impressions, no reply quality. Follower counts are as of the fetch.
- **Classification** of what each account "does" is my reading of the top posts.
- Raw JSON was not committed (about 15 MB, and it is other people's posts). The fetch is easy to reproduce.

## The accounts

| Handle | Followers | Bio (short) |
|---|---|---|
| levelsio | 968k | Solo founder, indie products |
| rasbt | 520k | ML/AI research engineer, author |
| svpino | 457k | Teaches AI/ML engineering |
| mattshumer_ | 397k | AI investor, ex-CEO HyperWrite |
| emollick | 396k | Wharton professor studying AI |
| bindureddy | 393k | CEO Abacus.AI |
| minchoi | 391k | "Sharing what's wild, what's practical" |
| omarsar0 | 323k | DAIR.AI founder |
| Yuchenj_UW | 301k | Databricks, ex-Hyperbolic co-founder/CTO |
| nearcyan | 273k | Independent |
| deedydas | 256k | Partner at Menlo Ventures |
| elder_plinius | 243k | Prompt jailbreaker, pseudonymous |
| simonw | 231k | Creator of Datasette, co-creator of Django |
| mckaywrigley | 228k | "I build & teach AI stuff" |
| tom_doerr | 222k | GitHub repo curation |
| swyx | 197k | Latent Space, AI engineer scene |
| rohanpaul_ai | 159k | Daily AI analysis newsletter |
| kimmonismus | 149k | Tech analyst, editor-in-chief of a newsletter |
| iScienceLuvr | 91k | CEO Sophont AI, ex-Stability AI |
| jerryjliu0 | 85k | CEO LlamaIndex |
| TheAhmadOsman | 79k | Founder, local/self-hosted AI |
| teortaxesTex | 77k | Pseudonymous, pro-open-weights, DeepSeek-focused |
| badlogicgames | 75k | Mario Zechner, libGDX creator, "pi" agent harness |
| burkov | 59k | ML author |
| skalskip92 | 50k | Open-source lead at Roboflow |
| vikhyatk | 29k | Builder of Moondream |
| Dorialexander | 25k | Open AI infrastructure at Pleias |
| bycloudai | 12k | YouTube AI research explainer |

## What the smaller accounts actually do (top posts by likes)

- **@skalskip92 (50k): build demos.** Videos of his own computer-vision work (football and NBA tracking) at 5k to 13k likes. He posts a thank-you at each GitHub-star milestone of his library (15k, 20k, 25k, 30k stars), each with commit and contributor counts.
- **@vikhyatk (29k): deadpan builder.** One-line engineer memes with an image; honest founder statements ("your data is safe. we don't train on it. because it would make the model worse"); a short anecdote about buying GPUs. Median post length 86 characters.
- **@Dorialexander (25k): narrative corrector.** Compressed explainers ("DeepSeek situation summarized: they are not a small engineer team but one of the leading frontier labs...") plus dry quote-posts about China and markets. Heavily quote-based.
- **@teortaxesTex (77k, pseudonymous): a worldview.** Interprets the Chinese AI/open-weights world that Western accounts under-cover. Quote-post and reply heavy. His biggest hits are Chinese hardware and robotics videos and culture posts.
- **@TheAhmadOsman (79k): local AI as identity.** Greentext memes (`>be me`), "just use the API" versus "run it yourself" stances, practical tricks (add `/.json` to a Reddit link), hardware builds.
- **@burkov (59k): blunt generalist.** Parables, contrarian reactions to corporate news and AI hot takes. Several of his biggest posts are not about AI.
- **@iScienceLuvr (91k): insider humor and lore.** Jokes, GPT-3 nostalgia, Math StackExchange lore, one-line takes on DeepSeek.
- **@badlogicgames (75k): grumpy practical dev.** Low like-to-follower ratio. His most useful post: "Don't build MCP servers. Build CLI tools with a --llm flag."
- **@bycloudai (12k): YouTube-first.** Benchmark charts and casual lowercase commentary. Low X engagement; X is secondary to his videos.

Larger non-lab accounts, for contrast:

- **Curators** (@minchoi, @rohanpaul_ai, @kimmonismus, @tom_doerr): a video or screenshot of a demo or repo with 4 to 8 words, or "10 wild examples:" lists. @minchoi's top posts reach 40k to 86k likes.
- **Analysts** (@simonw, @emollick): evidence screenshots of what a model actually did, and one-line analogies. Simon's "quitting programming over LLMs is like quitting carpentry over the table saw" reached 11.9k likes.

## Pooled measurement

1,931 posts from the 28 accounts dated 2024-06-01 or later. Each post is scored by its likes relative to that account's median likes (geometric mean, smoothed). Higher means better than the account's usual post.

| Feature | Posts | Relative likes | Rest |
|---|---|---|---|
| Length under 40 chars | 228 | 1.43 | 1.21 |
| Length 40-100 | 433 | 1.36 | 1.20 |
| Length 100-200 | 460 | 1.28 | 1.22 |
| Length 200-280 | 654 | 1.10 | 1.31 |
| Length over 280 | 156 | 1.12 | 1.25 |
| Video | 307 | 1.35 | 1.21 |
| Lowercase opening | 364 | 1.32 | 1.21 |
| Greentext | 102 | 1.31 | 1.23 |
| Photo | 972 | 1.24 | 1.23 |
| Question mark | 167 | 1.22 | 1.24 |
| Contains a number | 896 | 1.20 | 1.26 |
| Quote-post | 325 | 1.16 | 1.25 |
| External link | 115 | 1.11 | 1.24 |
| Thread cue | 32 | 1.11 | 1.24 |
| Hashtag | 12 | 1.00 | 1.24 |

Directional only. The sample is popular-post-skewed, so effects are probably compressed, and differences of about 0.1 are within noise for the smaller groups. A first-pass "has media" counter was broken (it reported 100% everywhere) and was discarded. In the pooled run, photo and video flags come from the first item's media type in the post's extended entities.

## Patterns

1. **A stable lens, not a topic.** Each account has a recognizable worldview or stance (China and open weights, local AI, builder honesty, blunt skepticism). People follow for the lens.
2. **Top posts are rarely paper summaries.** They are insider humor, a news item compressed to one line, a demo of the author's own build, an analogy, or a numbered chain ("How money works: 1. ...").
3. **Many top posts are not about AI.** Robotics videos, parables, finance and culture jokes. The audience rewards taste.
4. **Something exists off X.** Almost every account has a company, library, book, newsletter or YouTube channel behind it. X is distribution.
5. **Short wins** in this sample: under 100 characters beat 200-280 by roughly 1.4x versus 1.1x.
6. **Video and informal register** help a little. **External links** hurt only slightly in this sample (about -10%).

## Archetypes (for `x-profile` lens-picker)

1. **Builder-demo** (skalskip92, vikhyatk): show a build on video; milestone and gratitude posts.
2. **Worldview / niche lens** (teortaxesTex, Dorialexander, TheAhmadOsman, burkov).
3. **Humor insider** (vikhyatk, iScienceLuvr, TheAhmadOsman, Yuchenj_UW).
4. **Curator** (minchoi, rohanpaul_ai, tom_doerr, kimmonismus).
5. **Trusted analyst** (simonw, emollick): firsthand evidence plus a one-line takeaway.

## Also found [secondary]

- X removed about 42,000 accounts that used AI chatbots to automate replies and generic "thought-leadership" content (X trending page).
- Independent credibility is cited as a reason for audience trust: Simon Willison reportedly refuses sponsored content, and his newsletter is a diff of his blog (his own site).
- Growth blogs claim reply-first growth under 5k followers and 3 to 5 posts a day. Unverified (see `01-x-algorithm.md`).

## Open gap

X is login-walled, so there is no verified list of small, fast-growing accounts and no per-account growth data. Phase 0 of the MVP should be: the user picks 10 to 15 accounts they consider "great and small" and pastes or screenshots their recent posts, so we can fill this with real observations.

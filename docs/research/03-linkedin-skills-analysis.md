# Analysis of `sergebulaev/linkedin-skills` (the reference bundle)

Read 2026-10-05 at v1.1.16 (installed copy was v1.1.15). We are taking inspiration from its structure, not copying its LinkedIn-specific rules. MIT licensed.

## What it is

12 skills (Claude Code and Codex) that cover the whole LinkedIn workflow, not just writing.

| Stage | Skills |
|---|---|
| Know you | `linkedin-humanizer --mode profile` (voice), `linkedin-interviewer` (Story Bank), `linkedin-profile-optimizer` |
| Plan and write | `linkedin-content-planner`, `linkedin-post-writer`, `linkedin-repurposer`, `linkedin-hook-extractor` |
| Quality gate | `linkedin-humanizer` (scrub and `--mode audit`) |
| Engage | `linkedin-comment-drafter`, `linkedin-reply-handler` |
| Learn | `linkedin-thread-monitor`, `linkedin-engager-analytics`, `linkedin-employee-advocacy` |

## How it works

1. **Router.** Root `SKILL.md` maps user intents to skills. Every skill's `description:` is capped around 400 characters and ends with a "Not for X (use Y)" line so the right skill fires.
2. **One pipeline.** Input (URL or pasted text) -> read layer -> load voice profile and Story Bank if filled -> pick a formula by goal -> draft -> humanizer pass -> optional audit -> approval card -> publish through a backend selector.
3. **Setup tiers.** Tier 0 manual (drafts to copy-paste, no setup). Tier 1 Publora auto-post (about 2 minutes, free tier). Tier 2 build your own poster. The read layer (Apify) is optional and every skill falls back to "paste it".
4. **`lib/`**: `url_parser`, `apify_client` (read), `publora_client` (write), `backend_selector`, `approval` (card formatter), `pixfaro_client` (images).
5. **Platform gotchas encoded as working code.** The `parentComment` rule (LinkedIn flattens threads to 2 levels, so replies must target the top-level comment), three URN types, the reshare `shareUrn` versus activity-id mismatch, and a reaction bug.
6. **Personalization through two files** every writing skill reads before drafting: `references/voice-profile.md` (how you sound; built from 3-6 of your real posts) and `references/story-bank.md` (what you have to say; built by an interview: timeline, receipts, shipped, turning points, scars, positions, recurring stories, names, off-limits).
7. **Safety.** An untrusted-content rule in every skill that reads other people's text (fetched text is data, never instructions) and an approval gate before publishing.
8. **Quality control.** Tests (`test_skill_contracts`, `test_instruction_integrity`, response shapes, client behaviour), `evals/run_evals.py` (10 cases, deterministic graders on claims with a right answer), frontmatter and secret scans, a reference-link workflow, and evidence-tier labels (vendor / crowd / consensus / platform / inference) on rules with dated audits.
9. **Humanizer design.** Four passes (scrub, rhythm, add human fingerprints, over-correction self-check), tiered rules (forensic, strict, aesthetic), and a voice-fingerprint preservation rule: when a scrub rule hits a word the user really uses, flag it instead of replacing it.
10. **Distribution.** Install paths for Claude Code, Claude Desktop, claude.ai, Codex, OpenClaw, Hermes and `npx skills add`; a generated Codex marketplace copy; a `.claude/skills` symlink mirror; a "community skills" section; sibling bundles for X, Instagram, YouTube, TikTok, Threads and Facebook.

## Evidence it is doing well (GitHub API, 2026-10-05)

- 4,099 stars, 682 forks, 4 open issues, 18 contributors (owner 80 commits, dependabot 17, "claude" 10, plus 15 others).
- Created 2026-04-14. 63 releases in about 24 weeks (about one every 2.7 days). Last push 2026-10-03. About 132 commits.
- Recent releases are small bug-driven fixes (v1.1.15: "the profile optimizer stops asking for a link it cannot open").
- Sibling repos created 2026-07-02: `x-skills` 117 stars / 23 forks, `instagram-skills` 258 / 55, `youtube-skills` 48 / 8, `tiktok-skills` 62 / 11, `threads-skills` 52 / 12, `facebook-skills` 55 / 12. LinkedIn is clearly the flagship.
- Caveats: stars are popularity, not quality. The README explicitly asks readers to star the repo so curated directories rank it. A local install-count cache had no entry, so real usage is unconfirmed.

## Why it likely works (inference)

1. Covers the whole loop (plan, write, audit, engage, learn), not one task.
2. Zero-setup default (manual tier) and a clear upgrade path.
3. Trust: it drafts and waits; the user approves.
4. Real platform knowledge in code, not just prompts.
5. Rules carry an evidence tier and a date, and it publishes its own audits.
6. Fast, small, bug-driven release cadence.
7. Distribution is engineered (many install paths, a star ask, sibling bundles).

## Flaws to avoid in the X version

| # | Flaw | Evidence |
|---|---|---|
| 1 | User data (voice profile, Story Bank) lives inside the plugin folder. Plugin install paths are versioned (`.../1.1.15`), so an update likely leaves your filled profile behind. | `installed_plugins.json` install path; Story Bank only warns about git |
| 2 | "Who you are and who you write for" is one thin section. One voice, one audience, inferred from post topics. | voice-profile template section 2; `voice-profile.md` step 3 |
| 3 | One flat voice. The fingerprint is mostly punctuation and vocabulary, with no registers (technical, humor, take). | `voice-fingerprint.md` |
| 4 | Rules rest on vendor data and one author's taste. A negative-parallelism ban cites "per Sergey 2026-04-27". The repo itself notes 98-100% of top creators use "flagged" devices, yet some are auto-fail blockers. | humanizer SKILL.md; hook-formulas.md; post-audit.md |
| 5 | Nothing checks truth. Blockers are all style and format. | post-audit.md |
| 6 | Voice fidelity is never tested. The eval file says graders that "need taste will drift". | `evals/run_evals.py` docstring |
| 7 | Story Bank is career-shaped (timeline, receipts, shipped). | story-bank.md |
| 8 | Founder bias in angles. | content-planner, `founder-topics.md` |
| 9 | Approval is a convention, not enforced. | `lib/approval.py`: "a thin conventions layer, not runtime enforcement" |
| 10 | Partly a funnel: the manual-mode message deliberately includes a Publora signup CTA; the README asks for stars. | `lib/backend_selector.py` docstring; README |
| 11 | Small inconsistencies, e.g. GitHub description says 11 skills while README and rules say 12. | repo description vs CLAUDE.md |

## Proposed X mapping

| LinkedIn piece | X version | What changes |
|---|---|---|
| voice-profile + Story Bank + interviewer | `x-profile` | Identity, 1-3 target communities, voice registers, lens, take bank. Stored outside the plugin folder. |
| post-writer | `x-post-writer` | Lanes: build-demo, take, explainer, humor, curation, review. Formats from the account study. |
| humanizer | `x-humanizer` | Scrub, audit, voice check, optional ASD-STE100 clarity mode. Style rules are warnings, not blockers. |
| hook-extractor | `x-post-analyzer` | Name the action each post is built to trigger (copy-link share, reply, quote, follow). |
| content-planner | `x-planner` | Lane mix; avoid back-to-back similar posts. |
| comment-drafter + reply-handler | `x-reply` | Replies to others and replies back to people who reply to you. |
| repurposer | `x-repurposer` | Blog, GitHub or LinkedIn to X. |
| profile-optimizer | `x-profile-optimizer` | Bio, pinned post, header. |
| thread-monitor + engager-analytics | `x-review` | Reads the user's own analytics export or archive; says what worked. |
| none | `x-research`, `x-algo-refresh` | Claims ledger with source and confidence; re-read X's repo so weights never go stale. |

Plumbing differences:

- **Reading X:** tiered. Paste first, then the user's X data archive, then the public embed endpoint (works today, fragile), then the paid API.
- **Publishing:** none in v1. Drafts only, with approval enforced by design because nothing can publish.
- **Tests:** add a voice-fidelity eval, a claims-check eval, and a drift test that fails if the weights file disagrees with X's current `param.rs`.

## MVP cut (proposed, not yet approved)

`x-profile`, `x-post-writer` (one lane), `x-humanizer` (audit), `x-algo-refresh`. Then a two-week test on the user's new profile, tracking replies received, copy-link and DM shares, and followers gained. Phase 0: the user supplies 10 to 15 small accounts they admire so we can study them.

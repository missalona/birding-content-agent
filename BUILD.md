# BUILD: What We Built on the Claude Platform

This is the build log for question 5 of the exercise: the agent from [SPEC.md](SPEC.md), built for real in the Claude Console (Managed Agents), step by step.

> The client and the channel are examples for a course exercise. Platform IDs and account details are left out on purpose.

---

## 1. What was built

We built **phase A of the plan: the weekly research agent**. It researches the coming week in US birding, proposes 8 topics with sources, and keeps a notebook between runs so it never repeats a topic. Production, YouTube upload, email and the client approval flow are the next phases (section 8).

| Component (console screen) | Name | Status |
|---|---|---|
| Agent (Agents) | `birding-weekly-producer` | Version 3, tested |
| Research room (Environments) | `birding-research` | Cloud, limited network, 14 trusted hosts |
| Production room (Environments) | `birding-studio` | Cloud, limited network, no hosts yet (for phase B) |
| Notebook (Memory) | `birding-notebook` | Read and write, 2 files, updated by the agent |
| Sessions | 4 test runs | See section 5 |
| Schedule (Deployments) | Not created yet | Next step |
| Vault (Credential vaults) | Not created yet | Needed only for phase B |

Build order, as in the lesson (slide 16): **Agent → Environment → one session by hand → Memory → (next) Deployment**.

---

## 2. The agent configuration

| Setting | Value | Why |
|---|---|---|
| **Model** | `claude-opus-5-5` | The model the whole plan is built on |
| **Effort** | Medium | Enough for research; cheaper than High during tests |
| **Built-in tools** | `read`, `write`, `edit`, `glob`, `grep`, `web_search`, `web_fetch` on (Auto) | Research and saving files |
| **bash** | Off | Phase A runs no commands. Least access: no tool it does not need |
| **MCP servers, custom tools, skills, multi-agent** | None | Not needed in phase A |

The full configuration, exported from the console's **Raw** view (version 3):

```yaml
name: birding-weekly-producer
model:
  effort:
    type: medium
  id: claude-opus-5-5
  speed: standard
description: "Weekly research agent for a US birding YouTube channel. Version 3: proposes topics with a research budget, and keeps a notebook (memory store) between runs to avoid repeats (no production, no upload, no email)."
system: |-
  WHO
  You are the research agent of a YouTube channel for US birdwatchers.
  This is version 3: you only research and propose topics. You do not
  produce videos, upload anything or send emails.

  STEPS
  0. Read your notebook first (the birding-notebook memory store, under
     /mnt/memory/): topics-history.md, and last-run.md if it exists.
  1. Find what is seasonal in US birding next week (migration, feeding,
     counts, events).
  2. Research reliable sources: Cornell Lab (All About Birds, eBird,
     BirdCast, FeederWatch), Audubon, USFWS, ABA, peer-reviewed papers.
  3. Find the questions birders are actively discussing, as patterns
     only. Reddit is not reachable from this room; use what the allowed
     sources show (FAQs, comments, articles that answer common questions).
  4. Propose 5 Short topics, 1 long video topic and 2 backup topics.
     For each: title, format, angle (seasonal / community question /
     general), why now, and 2-3 sources with links.
     - Never repeat a topic that is already in topics-history.md.
     - Look at backup-unused topics first. Reuse one only if it still
       fits the season, and say so.
  5. Save the result as topics.md in /mnt/session/outputs, and end with
     a short summary (titles only, plus what you could not check).
  6. Update the notebook:
     - Add this week's 8 topics to topics-history.md (status planned or
       backup-unused). Mark any backup you reused as backup-used.
     - Rewrite last-run.md: date, week covered, searches and page reads
       used, what you could not check, one lesson for next time.

  RESEARCH BUDGET (keeps each run cheap)
  - Work from search results first. Titles and snippets are usually
    enough to choose a topic.
  - Use web_fetch only to confirm a key fact for a topic you already
    chose. Never fetch home pages, index pages or the same URL twice.
  - At most 12 web_search calls and at most 6 web_fetch calls per run.
  - When the budget is used up, finish with what you have and list
    what you could not check.

  RULES
  - Never invent a fact, number, quote or source.
  - Never quote or name individual users; describe patterns only.
  - The notebook holds notes only: never keys, passwords or personal data.
  - Never delete lines from topics-history.md; only add lines or change
    a status.
  - If a site blocks reading, say so and move on.
  - If a case is not covered here, stop and report.

  DONE
  topics.md exists in /mnt/session/outputs with 8 topics (5 Shorts,
  1 long, 2 backups), none repeated from the notebook, each with sources;
  topics-history.md and last-run.md are updated.
mcp_servers: []
tools:
  - configs:
      - enabled: false
        name: bash
        permission_policy:
          type: always_allow
        type: bash
    default_config:
      enabled: true
      permission_policy:
        type: auto
    type: agent_toolset_20260401
skills: []
metadata: {}
```

---

## 3. The rooms (Environments)

### `birding-research` (phase A)

| Setting | Value |
|---|---|
| Hosting | Cloud (runs even when my computer is off) |
| Network | Limited |
| Package managers | Off |
| MCP servers | Off |
| Allowed hosts | 14 of 16 |

```
*.allaboutbirds.org
ebird.org
birdcast.info
feederwatch.org
birdsoftheworld.org
*.cornell.edu
*.audubon.org
*.fws.gov
*.usgs.gov
*.aba.org
stateofthebirds.org
*.smithsonianmag.com
*.birdforum.net
finchnetwork.org
```

**Why this list matters:** with limited networking, this list also controls `web_search` and `web_fetch` (section 6). Search results come only from these sites, so "reliable sources only" is enforced by the platform, not just by the instructions.

### `birding-studio` (phase B, later)

Cloud, limited network, no hosts yet. It will get the production hosts (ElevenLabs, Google APIs, Wikimedia and so on) when the production phase is built. A separate room is needed because one room allows at most 16 hosts, and because research and production run in separate sessions anyway ("one room per kind of job", slide 18).

---

## 4. The notebook (Memory store)

| Setting | Value |
|---|---|
| Name | `birding-notebook` |
| Access in sessions | Read and write |
| Description (shown to the agent) | "The research agent's notebook, kept between weekly runs. Read topics-history.md before choosing topics: never repeat a topic from it, and reuse unused backups first if they still fit the season. At the end of every run, add this week's topics and backups to topics-history.md and rewrite last-run.md. Notes only: never store keys, passwords or personal data." |

| File | Written by | What is in it |
|---|---|---|
| `topics-history.md` | Seeded by me, then updated by the agent | Every topic, one line each, with a status: planned / backup-unused / backup-used |
| `last-run.md` | The agent | Date, week covered, budget used, verified facts worth reusing, what it could not check, one lesson |

The lesson the agent wrote for itself after its first run with the notebook:

> "Search results often surface last winter's articles as if current. Check the publish date (one fetch) before building a 'this winter' claim, and search event dates early because festivals may fall before the target week."

The console shows **"Updated by"** and the session for every file change, which is the audit trail from slide 21.

---

## 5. The test sessions

| # | Session | Room | Notebook | Result | Cost | Tokens read |
|---|---|---|---|---|---|---|
| 1 | v1 test 1 | `birding-studio` (0 hosts) | No | Every search failed. The agent **stopped and reported** instead of inventing topics | $0.07 | 20K |
| 2 | v1 test 2 | `birding-research` (13 hosts) | No | 8 sourced topics. Flagged out-of-date articles and a conflict between two sources | $2.02 (hit the $2 cap) | 2.1M |
| 3 | v2 test 1 | `birding-research` (14 hosts) | No | 8 sourced topics, read this winter's finch forecast. Used 12 searches and 3 page reads | $1.22 | 560K |
| 4 | v3 test 1 | `birding-research` (14 hosts) | Yes | Read the notebook first, repeated nothing, reused both unused backups, updated both notebook files | $1.09 | 622K |

**Total spent on tests: $4.40.** Every session had a $2 budget cap.

---

## 6. What the build taught us (and what changed in the plan)

| Finding | What it changed |
|---|---|
| With **limited** networking, the room's allowed hosts **also apply to `web_search` and `web_fetch`**, even though these tools run on Anthropic's servers. An empty list means no search results at all. (Slide 17 suggested otherwise; the platform docs are explicit.) | A separate research room with a list of trusted hosts |
| A bare host matches only itself: `audubon.org` does not match `www.audubon.org`. `*.audubon.org` matches every subdomain but not `audubon.org` itself | Most hosts are written with `*.` |
| The platform **refuses `reddit.com`** as an allowed host: Reddit blocks Claude's crawler | Reddit questions need another route: Apify, run by Make (see section 8) |
| `web_fetch` opens only URLs that came from a search result or from the user | Search first, then read |
| Cost is driven by **what the agent reads**, not what it writes: every fetched page stays in the context and is re-read on every turn | A research budget in the instructions cut the cost by about 40% |
| There is no per-tool setting to cap page length | The instructions are the cost lever |
| Every edit to the agent creates a **new version**; older versions stay | v1 → v2 → v3, each tested before moving on |
| A memory store is attached **per session** (Resources), not to the agent | Every schedule must attach the notebook |
| Files in `/mnt/session/outputs` are kept after the session; the rest of the room is deleted | The agent saves `topics.md` there |
| The session form pre-selects an environment, not always the right one | Check the room every time a session is created |

---

## 7. Version history

| Version | What changed | Why |
|---|---|---|
| 1 | Research only: model, prompt, tools, `bash` off | Start small and test |
| 2 | Research budget (12 searches, 6 page reads, search results first, short final summary) | v1 cost $2.02 per run |
| 3 | Reads the notebook first; never repeats; reuses unused backups; updates the notebook at the end | Memory between weekly runs |

---

## 8. Not built yet (next steps)

| Step | What it needs |
|---|---|
| Weekly schedule | A Deployment for phase A, created **paused**, with the research room, the notebook and a budget cap attached (about $1.1 per run) |
| Reddit questions | A Make scenario with Apify that collects birding questions into a table the agent reads, as in Lesson 5 |
| Client approval flow | Vault, Gmail and Drive connectors, and the Make reply watcher and reminder (SPEC sections 10 and 15) |
| Production (phase B) | The `birding-studio` room with production hosts, `bash` on, the production skills, ElevenLabs and YouTube credentials in the vault |
| Monitoring | Webhook to Make for alerts when a session stops |

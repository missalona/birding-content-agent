# SPEC: Weekly Content Agent for a Birding Channel

| | |
|---|---|
| **Agent name** | `birding-weekly-producer` |
| **Platform** | Claude Platform, Managed Agents |
| **Model** | Claude Opus 5.5 (`claude-opus-5-5`) |
| **Channel** | A sample client's YouTube channel about birdwatching, for a US audience |
| **Operator** | A YouTube channel manager who produces automated content |
| **First run** | Saturday, December 5, 2026, 09:00 Israel time |
| **Automation live from** | Sunday, December 6, 2026 (first email, first video goes live) |
| **Document status** | Draft 1, November 26, 2026, planning only |

> Note: the client and the channel are examples for a course exercise. Identifying details (names, addresses, email) were replaced with generic names. The document is written as a plan for a client, with minimal permissions and guardrails.

---

## 1. Summary in one sentence

Every Saturday morning the agent checks what is already on the channel, researches new topics from reliable sources and birding communities, produces 5 Shorts and 1 long video, uploads and schedules them on YouTube for Sunday to Thursday, and updates a content plan in Google Drive. Every Sunday morning the owner gets an email with the week's videos.

## 2. The client's problem

* The channel already has a few videos, but the owner has no time to make more.
* The owner wants the channel to grow: more videos, on a steady schedule, on topics US birders actually search for.
* No duplicates of videos that already exist.
* In birding the season matters a lot (migration, winter feeding, nesting), so content must fit the time of year.

## 3. Why an agent and not a Make automation

Using the four checks from Lesson 6, slide 12:

| Check | Answer | Why |
|---|---|---|
| Is the work multi-step and hard to specify in advance? | Yes | Choosing a topic, checking facts, writing a script and reviewing a video's quality all need judgement. It cannot be drawn as a fixed chart of boxes. |
| Is it worth the cost? | Yes | 6 videos a week instead of several days of manual work. |
| Can Claude do this kind of task? | Yes | We have already produced videos for this channel with Claude, using the existing production skill. |
| Can a mistake be caught and undone? | Yes, with conditions | Videos are uploaded scheduled, not published at once, so there is time to stop them. The Sunday email shows everything before anything goes live. |

The fixed parts (sending an email at a set time, starting on Saturday) could run in a simple automation. The part that needs an agent is the research, the choice of topics and the production.

---

## 4. The goal: what the agent delivers every week

### Weekly deliverables

| # | Deliverable | Details |
|---|---|---|
| 1 | 5 Short ideas | Vertical 9:16, 45-60 seconds, one idea per video |
| 2 | 1 long video idea | 16:9, 5-10 minutes |
| 3 | 6 edited, finished videos | Made with the skill, passed the quality gate |
| 4 | 6 videos scheduled on YouTube | Each with title, description, tags and publish time |
| 5 | Updated content plan in Drive | Every topic: published and planned |
| 6 | Weekly email | Sunday, 09:00 Israel time |
| 7 | **Immediate** alert email, only when something is wrong | When a video gets stuck for any reason: no credits, a tool failure, a failed quality gate, rejected or locked on upload, or not published on time |
| 8 | Notice email before production, only when needed | When a video needs an external platform other than ElevenLabs (for example Higgsfield), or ElevenLabs credits are not enough |

### When the agent knows it is done (DONE)

The Saturday run is done when all of these are true:

1. 6 new rows in the content plan, one per video, with status `scheduled` or `needs_a_look`.
2. Every video that passed the quality gate is uploaded to YouTube as private and scheduled.
3. The video files are saved in this week's Drive folder.
4. The "last run" file in the memory notebook is updated.
5. A one-line report is written: what was uploaded, and what needs the owner's attention.
6. For every video that got stuck during the run, an alert email was sent **the moment the problem was found**, not at the end of the run.

The Sunday run is done when the email has been sent once.

The daily publish check is done when every video of the day is marked `published`, or an alert email was sent about the ones that are not.

---

## 5. Schedule

### Setup

| Date | What happens |
|---|---|
| By Dec 3, 2026 | Set up every component on the platform (section 18) |
| Dec 4, 2026 | Rehearsal: one manual run in `DRY_RUN` mode (no YouTube upload, no email), then read the whole run in the session viewer |
| **Saturday, Dec 5, 2026, 09:00** | **First real production run** (plans and produces the week of Dec 6-10) |
| **Sunday, Dec 6, 2026, 09:00** | **First weekly email.** The first video goes live the same day, US time |

### The weekly cycle (Israel time)

| Day and time | What runs |
|---|---|
| Saturday 09:00 | Production run: research, planning, production, upload and scheduling |
| Sunday 09:00 | Weekly email run |
| Sunday to Thursday 21:30 | Publish check: did today's videos really go live? |
| Sunday to Thursday | Videos go live as scheduled, in US hours |

> The platform starts scheduled runs with up to 9 minutes of jitter (Lesson 6, slide 25). So "09:00" really means "by 09:10".

### How the videos are spread over the week

6 videos over 5 days, so one day gets two:

| Day | Short | Long video |
|---|---|---|
| Sunday | Short 1 | |
| Monday | Short 2 | |
| Tuesday | Short 3 | |
| Wednesday | Short 4 | The long video, at a different hour from the Short |
| Thursday | Short 5 | |

The day of the long video is a proposal, listed in the open decisions (section 19).

---

## 6. How the agent works: the steps of the Saturday run

This is the list of steps. The full flow diagram, with every component of the system, is in `Architecture.md` (question 4 of the exercise).

| Step | What the agent does | Where it is saved |
|---|---|---|
| **0. Start** | Reads the notebook (`last-run.md`, `lessons.md`) and the channel rules | Memory |
| **1. What is already on the channel** | Pulls the list of every video on the channel (titles, descriptions, dates) and compares it with the content plan | Read only |
| **2. Season** | Finds what is happening in US birding next week: migration, feeding, counts, events | Run context |
| **3. Research** | Scans the sources in a fixed order (section 12): authoritative sources, science news, birding communities, search data | A research file per topic |
| **4. Choose topics** | Chooses 5 Short topics and 1 long video topic. Each topic is checked against the existing videos to avoid duplicates. Also chooses 2 backup topics | Content plan in Drive, status `planned` |
| **5. Keywords** | Finds a search phrase for each topic with high demand and low or medium competition | Research file |
| **5a. External platform check** | Plans where each video's images and clips will come from. If any video needs an external platform other than ElevenLabs (for example Higgsfield), or ElevenLabs credits are not enough, sends **one notice email before any production starts** | Content plan, `notes` column |
| **6. Production** | Produces each video with the skill (section 11): script, narration, music, sound effects, licensed images and clips, animation, render, mix | Folder in the container |
| **7. Quality gate** | Runs the skill's quality gate. Fixes and runs it again, up to 3 rounds | QA report per video |
| **8. Save** | Uploads the video file and its upload package to this week's Drive folder | Drive |
| **9. Upload and schedule** | Uploads to YouTube as private with `publishAt`, title, description, tags and the synthetic content flag | YouTube + content plan, status `scheduled` |
| **9a. Check after upload** | Reads each video's status on YouTube: is it really scheduled, or was it rejected or locked as private? If there is a problem, sends one alert email (section 15) | Content plan, status `blocked` |
| **10. Finish** | Updates the notebook and writes a one-line report | Memory |

**A video that fails the quality gate after 3 rounds is not uploaded.** It is marked `needs_a_look` in the content plan, and the agent produces a backup topic instead.

**Every video that gets stuck, for any reason and at any step, triggers an immediate alert email** (section 15). The agent sends the email first, and only then moves on to the next video or the backup topic.

### The loop and the brakes

The platform runs the loop (ask the model, run a tool, return the result, repeat). The brakes I set:

* **A dollar budget per session** (section 10). When it runs out, the session stops and waits for me.
* **Up to 3 fix rounds** per video in the quality gate.
* **Up to 2 attempts** for any external action that failed (upload, email), with a check before every retry (section 14).

---

## 7. Agent definition (Agents screen)

```yaml
name:        birding-weekly-producer
model:       claude-opus-5-5
effort:      high
system:      the full instructions in Appendix A
tools:       bash, read, write, edit, glob, grep     on
             web_search, web_fetch                  on   (research needs the open web)
mcp:         google-drive · gmail · vidiq
custom:      youtube_list_channel_videos · youtube_upload_scheduled · youtube_get_video_status
skills:      birding-video-production · video-research-sources · youtube-publishing
version:     1   (every session is pinned to a version)
```

### The tools, and what each one may do

| Tool | What for | Reads or writes | Permission policy |
|---|---|---|---|
| `bash`, `read`, `write`, `edit`, `glob`, `grep` | Work inside the container: rendering, ffmpeg, quality gate scripts | Writes only inside the room | `auto` |
| `web_search`, `web_fetch` | Research: sources, communities, season | Reads | `auto` |
| `youtube_list_channel_videos` (custom tool) | List of the channel's videos, to avoid duplicates | Reads | `auto` |
| `youtube_upload_scheduled` (custom tool) | Upload a video as private with a publish time | **Writes to the client's system** | `auto` (my decision, see below) |
| `youtube_get_video_status` (custom tool) | A video's status: scheduled, public, private, rejected | Reads | `auto` |
| Google Drive | Content plan and video folder | Reads and writes | `auto` |
| Gmail | Weekly email and alert emails | **Sends messages** | `auto`, and only to one address |
| VidIQ | Keyword demand and competition, outlier videos | Reads | `auto` |
| ElevenLabs API (from `bash`, key in the vault) | Narration in the Aris voice, sound effects and background music | Writes (costs credits) | `auto`, after a credit check (section 11) |

**The agent has no tool for any other external platform** (such as Higgsfield). All production happens inside Claude, and ElevenLabs is the only external production service. If a video really needs another platform, the agent sends a notice email before production (section 15), and the decision to connect it is mine.

**A conscious decision:** Lesson 6 recommends `always ask` for every write to a client's system. Here I chose to let the process run end to end without my approval. What replaces the approval:

1. Videos are uploaded **private and scheduled**, not public. There is time to stop them before they go live.
2. An automatic quality gate: a video that fails is not uploaded.
3. The Sunday email arrives before the first video goes live.
4. The agent has **no** tool to delete, replace or edit existing videos. It only adds.

**The custom tools** run on my side. The model asks, my server calls the YouTube Data API, and returns the result. The key never enters the room.

---

## 8. Memory

Following the lesson's rule of thumb (slide 7): **context for this run, notebook for the next run, database for everyone.**

| Layer | What is in it | How long it lives | Where |
|---|---|---|---|
| **Context** | Instructions, search results, scripts in progress, tool results | Only during the session | Inside the session. The platform compacts it when it fills up |
| **Notebook (Memory stores)** | Channel rules, lessons, what happened in the last run | Between runs | Two memory stores on the platform |
| **The world (System of record)** | The videos themselves, the list of every topic, the video files | Permanent, shared by everyone | YouTube, the content plan in Drive, the video folder in Drive |

### Memory store 1: `birding-channel-rules` (read only)

The channel's fixed rules. The agent reads them and never writes. Only I update them.

```
/mnt/memory/birding-channel-rules/
    channel.md          who the audience is, the channel's goal, what was decided and why
    brand.md            logo, color palette, fonts, watermark
    shorts-rules.md     length, opening, pace, safe area, call to action, birdsong at the end
    account-limits.md   YouTube account limits (no links in descriptions, no custom thumbnails)
    sources.md          the birding source profile, by tier
```

Store description (shown to the agent): *"Birding channel rules. Read before planning or producing any video. Never contradict these rules; if a rule blocks the work, stop and report."*

### Memory store 2: `birding-notebook` (read and write)

The agent's notebook. This is where it writes what it learned and what it did.

```
/mnt/memory/birding-notebook/
    last-run.md          2026-12-05: 6 produced, 5 scheduled, 1 needs_a_look (reason)
    lessons.md           "source X blocks reading, use search instead"
    animation-log.md     the animation style and font pair of every video, to avoid repeats
    keyword-notes.md     which search phrases worked, based on view data
    seasonal-calendar.md seasonal events already covered, and what is planned for the coming weeks
```

### What does **not** go into memory

* **Passwords, API keys and tokens.** Never. They live in the vault (section 9).
* **The full topic list and the video files.** That is data, and it lives in Drive and on YouTube.
* **Usernames and personal details from the communities.** Only patterns: which question keeps coming back, which mistake is common.

### The content plan in Drive (the database)

A Google Sheets file called `Birding Channel: Content Plan`. Each row is one video, past or future.

| Column | Example |
|---|---|
| `week_of` | 2026-12-06 |
| `format` | `short` / `long` |
| `topic` | Why do chickadees cache seeds in winter? |
| `angle` | seasonal / community question / myth / general topic |
| `primary_keyword` | chickadee winter food caching |
| `sources` | Cornell Lab, All About Birds; Audubon |
| `status` | `planned` / `produced` / `scheduled` / `published` / `needs_a_look` / `stuck` / `blocked` |
| `publish_at_et` | 2026-12-06 10:30 ET |
| `youtube_url` | https://youtu.be/... |
| `drive_folder` | link to the week's folder |
| `notes` | why it was chosen, or why it was flagged |

### Managing memory over time

* **Before every run** the agent reads `last-run.md` and `lessons.md`.
* **At the end of every run** it rewrites `last-run.md` (only the latest run) and adds new lessons to `lessons.md`.
* **Every write is saved as a version** on the platform, so changes can be reviewed and rolled back.
* **Once a month** I review `lessons.md`, remove duplicates, and move any permanent rule into the rules store.
* Store limits: up to 100 kB per file, up to 8 stores per session, and a store can only be attached when the session is created.

---

## 9. The vault (Credential vaults)

Vault name: `birding-client`. The room only sees a placeholder, and the platform swaps in the real value on the way out (Lesson 6, slide 20).

| Key | Type | Minimal permissions |
|---|---|---|
| Google (Drive) | OAuth login for the connector | `drive.file`: only files the agent itself created |
| Google (Gmail) | OAuth login for the connector | `gmail.send` only, no reading of the inbox |
| YouTube | Kept on the custom tools' server, not in the room's vault | `youtube.upload` and `youtube.readonly`. **No** delete permission |
| ElevenLabs | Environment variable `ELEVENLABS_API_KEY` | A key dedicated to this agent, from the client's account: narration (the Aris voice), sound effects and music only, with a credit cap (section 11) |
| VidIQ | Connector login | Read only |
| Pixabay, Pexels | Environment variable | Search for licensed clips and images |

---

## 10. The room and the schedules

### The room (Environments screen)

| Setting | Value |
|---|---|
| Name | `birding-studio` |
| Type | Cloud |
| Packages | Node.js, Python, ffmpeg, Chromium (for rendering), faster-whisper (to check the narration) |
| Network | **Limited.** Only the hosts the work needs |

Allowed hosts: `api.elevenlabs.io`, `commons.wikimedia.org`, `upload.wikimedia.org`, `pixabay.com`, `api.pexels.com`, `huggingface.co` (Whisper model), the package registries (npm, PyPI), fonts (`fonts.gstatic.com`, `cdn.jsdelivr.net`), and the Google and VidIQ connectors.

> The web search tools run on Anthropic's side and are not bound by this list (slide 17). They are turned on only because the research really needs them.

### The schedules (Deployments screen)

| | Production run | Email run |
|---|---|---|
| **Name** | `birding-saturday-production` | `birding-sunday-email` |
| **Agent** | `birding-weekly-producer`, pinned version | Same agent, same version |
| **Schedule** | `0 9 * * 6` (Saturday 09:00) | `0 9 * * 0` (Sunday 09:00) |
| **Time zone** | `Asia/Jerusalem` | `Asia/Jerusalem` |
| **Memory stores** | Both | Both |
| **Budget** | Set after the rehearsal (see below) | Very low, this is a short run |
| **First event** | "Plan and produce next week's 5 Shorts and 1 long video, upload and schedule them, update the content plan." | "Send the weekly email for the videos scheduled this week." |
| **Activation** | Created paused, turned on Dec 4 after the rehearsal | Same |

### Third schedule: the daily publish check

| | |
|---|---|
| **Name** | `birding-publish-check` |
| **Agent** | Same agent, same version |
| **Schedule** | `30 21 * * 0-4` (Sunday to Thursday, 21:30) |
| **Time zone** | `Asia/Jerusalem` |
| **Why this hour** | After the Short's publish time (17:30 Israel time) and the long video's (20:00 Israel time), with a margin |
| **Budget** | Very low: reading, plus an email if needed |
| **First event** | "Check that today's scheduled videos are public. Alert by email if any is not." |

**The budget:** I do not have a real number yet. In the rehearsal I will read in the session viewer how much a full run cost, and set a cap of about 1.5 times that. There are also costs outside the platform: ElevenLabs and VidIQ credits.

---

## 11. The video production skill

Name: `birding-video-production`. It is based on the production skill we have built so far, with the changes the platform requires.

### What stays the same (the rules that already worked)

**Shorts**
* 45-60 seconds, 1080x1920, 60 frames per second. One idea per video. A rich topic is split into two parts.
* **The first frame is already the hook:** a close-up of the bird and a big question on screen. No title card and no logo in the first 3 seconds.
* A visual change every 1.5-2.5 seconds. Nothing frozen for more than 3 seconds.
* Always narration (ElevenLabs, the Aris voice), music and sound effects. **Every video ends with real birdsong** under a suitable license.
* A call to action at the end of every video: an invitation to subscribe.
* Natural colors only, no neon.
* **Text never sits under the Shorts interface:** inside x 120-960, above y 1440, and left of x 820 below y 880.
* Real, licensed clips and images first (Public domain, CC0, CC BY). No clip? A licensed image, or an illustration and infographic built in code inside Claude.
* A new animation style and a new font pair for every video.
* Every fact has a source. No numbers without a source.

**Long video (5-10 minutes)**
* 16:9, 1920x1080, following the long-film process we already used: chapters, big-number screens, designed silences, music throughout.
* A 20-second area at the end for YouTube's end screen.

**The quality gate (mandatory)**
* Automatic checks: every text is readable, no overlaps, everything inside the safe area, loudness of -14 LUFS, no single-frame glitches, Whisper on the final narration.
* A check "by eye" of still frames: the middle of every scene, every transition, zoomed crops of every text.

### What changes on the platform

| Until now | In the agent |
|---|---|
| Narration and music through the ElevenLabs website in a browser | Through the ElevenLabs API, with a key in the vault |
| Upload to YouTube through YouTube Studio in a browser | Through the YouTube Data API, with a custom tool |
| My approval of the topics and of every video | An automatic quality gate, upload as private and scheduled, and a Sunday email |
| Files on my computer | Files in the container, and from there to Drive |
| Reddit through Google search in a browser | Through `web_search` (some sites may block it) |
| Illustrations and photoreal images in Higgsfield | Illustrations and infographics built in code inside Claude, alongside licensed images and clips |
| Sound effects made with ffmpeg | Sound effects from ElevenLabs, with the existing effects library as a fallback |

### Narration: the Aris voice from ElevenLabs, through the API

Until now we produced the narration on the ElevenLabs website, in a browser. The agent on the platform has no browser with a signed-in account, so it works through **the API**: a "door" that software uses to ask ElevenLabs for narration, with no screen.

**Why the API and not another way**

| Option | Why or why not |
|---|---|
| **API directly from the room** (chosen) | The simplest and most stable. One key in the vault, one call per narration |
| ElevenLabs website in a browser | The agent has no signed-in browser. It was also fragile for us (typing a long text froze the page) |
| ElevenLabs MCP connector | The official connector runs on a local computer, and an agent on the platform needs a connector that runs in the cloud. Check at setup whether such a version exists |

**How it works**

1. The ElevenLabs key is stored **in the vault** as an environment variable (`ELEVENLABS_API_KEY`). The room only sees a placeholder, and the platform swaps in the real key only when the request leaves.
2. **The voice ID** of Aris is stored in the channel rules store. It is not a secret. If Aris is a voice from the ElevenLabs voice library, it is added once to the account's voices, and only then is it available through the API.
3. **The settings are sent with every request**, because the website saved them in the account and the API does not: the same voice model we chose on the website, stability 0.55, similarity 0.75, speed 0.84. The exact model name in the API is checked at setup.
4. **Narration with timestamps:** ElevenLabs can return the timing of every character together with the audio. This keeps the animation in sync with the narration from the first request. Whisper stays only as a final quality check.
5. **Sound effects and background music** also come from the ElevenLabs API, with the same key. The effects we already made stay in the skill as a fallback, in case an ElevenLabs effect comes out too quiet (this has happened to us before).
6. **Before production** the agent checks how many credits are left in the account. If there are not enough for the whole week, it sends a notice (section 15) **before** it starts producing.

**When the agent is built for a client**

* **The client's account, not mine.** The commercial license on ElevenLabs belongs to the account that generates the narration. So the client should open their own paid account with a commercial license and API access, and pay for the credits directly.
* **A separate key for this agent.** The client creates a dedicated API key and passes it into the vault. The account's general key is not used.
* **A restricted key**, if the account allows it: narration, music and reading the voice list only, with a monthly credit cap.
* **One key per client.** If a key has to be replaced or revoked, other clients are not affected.
* **The key is never written** in the instructions, the memory, the repo or an email. Only in the vault.

### Other skills

* `video-research-sources`: the existing skill for scanning sources in a fixed order.
* `youtube-publishing`: a new skill for titles, descriptions, tags and publish times (section 13).

---

## 12. Research: where the ideas come from

Every week the agent scans the same sources, in the same order:

| Tier | Sources | What for |
|---|---|---|
| **A. Authoritative** | Cornell Lab of Ornithology (All About Birds, eBird, BirdCast, Project FeederWatch), Audubon, American Birding Association, USFWS, USGS Bird Banding Lab, State of the Birds, peer-reviewed papers | The facts. A fact counts as verified only if it is in a primary source and does not contradict another authoritative source |
| **B. Science news** | Cornell Chronicle, Audubon Magazine, Smithsonian, Science News | What is new this week. Always cite the study itself, not the article |
| **C. Birding communities** | r/birding, r/whatsthisbird, r/BackyardBirding, r/hummingbirds, BirdForum, public Facebook groups only | The hottest questions and discussions, and the audience's own words |
| **D. Demand** | VidIQ: search volume, competition, outlier videos | That people really search for the topic |
| **E. Assets and licenses** | Wikimedia Commons, USFWS and NPS (public domain), Pixabay, Pexels | Licensed clips and images |

### The weekly topic mix

| Type | How many | Example for December |
|---|---|---|
| Seasonal | 2-3 | Winter feeding, the Christmas Bird Count, snowy owls, birds migrating south |
| A hot community question | 1-2 | A question that keeps coming up this week on r/birding |
| A general topic US birders care about | 1 | Identification, gear, birding ethics |

### Research rules

* **Never invent** a number, quote, date or source.
* **When sources disagree**, report both and choose the more authoritative one.
* **Anything we could not read** (a blocked site, a closed group) is noted in the research file. Never sign in to any account and never join groups.
* Never quote or name users. Only patterns.
* **No duplicates:** a topic that already exists on the channel, or in the content plan in the last 6 months, is not chosen again. Unless it has a truly new angle, written in the `notes` column.

---

## 13. Uploading to YouTube

### What is written for every video

| Field | Rule |
|---|---|
| **Title** | The main search phrase first, then the curiosity. 45-70 characters for Shorts. The agent writes 3 options and picks the strongest |
| **Description** | What the video answers, one tip, sources **by name only** (no links, see the account limits), credits for clips, music and voice |
| **Tags** | 8-12: main phrase, secondary phrases, species names, question phrases |
| **Hashtags** | 3, for example `#birdwatching #shorts` |
| **Settings** | Not made for kids, category Pets & Animals, language English (US), synthetic content = yes (AI voice) |
| **Privacy** | `private` with `publishAt`, so the video goes live on its own at the set time |

### Publish time

* The audience is in the US, so the time is set in US Eastern time (ET) and stored in UTC.
* **Default:** Shorts at 10:30 ET (17:30 Israel time), the time the channel has used so far. The long video at 13:00 ET.
* **Once there is enough data**, the agent adjusts the time to the channel's data (when its viewers are on YouTube), and writes the change in `keyword-notes.md`.

### Account and YouTube limits

* **No links in descriptions** and **no custom thumbnails** while advanced features are not available on the account. Instead of a thumbnail, a frame from the video is chosen.
* **Never replace or re-upload** a video that is already up. Only fix texts.

---

## 14. Error handling and safe retries

| What happened | What the agent does |
|---|---|
| A key fact cannot be verified | Drops the topic and moves to a backup topic |
| A video fails the quality gate after 3 rounds | Does not upload it. Marks it `needs_a_look`, **sends an alert email at once**, and produces the backup topic |
| ElevenLabs fails | Tries once more. If it fails again, marks the video `stuck`, **sends an alert email at once**, and moves on to the next video |
| Credits run out in the middle of production | Stops every video that still needs narration, marks them `stuck`, and **sends an alert email at once** with the list of stuck videos |
| Any other failure that stops a video | Marks it `stuck`, **sends an alert email at once** with the step and the reason, and moves on to the next video if possible |
| The YouTube upload failed | **Before retrying**, checks the channel's video list to see if the video is already up (by title). Only if it is not, tries again. This avoids a duplicate upload |
| YouTube rejected the upload, or the video was locked as private | Does not try to work around it. Marks it `blocked` in the content plan and sends an alert email (section 15) |
| A video did not go live at its set time | The daily publish check finds it, marks it `blocked` and sends an alert email |
| The session stopped midway (budget, failure) | The agent cannot send an email once it has stopped, so **the webhook sends the alert** (section 16). In the next run the agent reads the content plan and `last-run.md`, and finishes only what is missing |
| The case is not covered by the instructions | **Stops and reports.** Does not improvise |

---

## 15. The emails

### The weekly email

| | |
|---|---|
| **When** | Every Sunday at 09:00 Israel time, from Dec 6, 2026 |
| **To** | `OWNER_EMAIL` only (the address is stored in the agent's settings, not in the repo) |
| **Data from** | This week's rows in the content plan with status `scheduled` |
| **Subject** | `Birding Channel: this week's videos (Dec 6-10)` |

Email content (in the owner's language):

| Video title | Publish date on YouTube | Link to the video |
|---|---|---|
| Why Do Chickadees Hide Seeds in Winter? | Sunday, Dec 6, 2026, 10:30 ET | https://youtu.be/... |
| ... | ... | ... |

At the end of the email, only if there are any: one line about videos marked `needs_a_look` or `blocked`, and why.

> The link works at once, but the video can only be watched from its publish time.

### Immediate alert email: a video got stuck

| | |
|---|---|
| **When** | **At once** when the problem is found, at any step: production, quality gate, upload, or the daily publish check |
| **About** | Any video that got stuck for any reason: no credits, an ElevenLabs or other tool failure, a failed quality gate, rejected or locked on YouTube, a video that did not go live on time |
| **To** | The same address: `OWNER_EMAIL` |
| **Subject** | `Birding Channel: alert, a video is stuck` |
| **How many** | One email per event, the moment it happens. If several videos get stuck at the same moment for the same reason (for example every upload was blocked), they go into one email |

Email content:

| Video title | Stuck at which step | What happened | What the agent did | Link |
|---|---|---|---|---|
| Why Do Chickadees Hide Seeds in Winter? | Narration | ElevenLabs credits ran out | Stopped this video, continuing with videos that already have narration | Link to the Drive folder |
| Snowy Owls Are Back | Upload | Locked as private. Most likely the API project has not passed the audit yet | Marked `blocked`, did not try to work around it | Link to YouTube Studio |

At the end of the email: what I can do (for example add credits, or publish manually in Studio), and a note that the agent did **not** try to work around the problem.

### Notice email: an external platform before production

| | |
|---|---|
| **When** | On Saturday, after the topics are chosen (step 5a) **and before any video is produced** |
| **Sent if** | A video needs an external platform other than ElevenLabs (for example Higgsfield), or ElevenLabs credits are not enough for the whole week |
| **To** | The same address: `OWNER_EMAIL` |
| **Subject** | `Birding Channel: notice before production, external platform needed` |
| **How many** | One email per run, listing every video that needs an external platform |

Email content:

| Video title | Platform | Why it is needed | Estimated credits |
|---|---|---|---|
| Snowy Owls Are Back: Where to See Them | Higgsfield | No licensed clip or image of a snowy owl in this pose | About 2 credits (8 images) |

**What counts as an "external platform":** any service that produces part of the video and is not Claude or ElevenLabs. The goal is to keep all production inside Claude, with ElevenLabs connected only for narration, sound effects and background music. Licensed media sources (Wikimedia, Pixabay, Pexels) are sources of raw material, not production tools, so they do not count. Google (storage and email) and VidIQ (research) do not count either.

**What the agent does after the email:** it does not use the platform. It replaces the visual with an illustration or infographic built in code, or switches to a backup topic. Only if I decide to connect the platform is it added to the agent as a new version.

---

## 16. Guardrails and monitoring

### The lesson's six guardrails (slide 28)

| Guardrail | How it is set here |
|---|---|
| **A dollar budget** | A cap per session and per schedule. A credit cap on the ElevenLabs key |
| **Permission policy** | `auto` for the room's tools. Tools that write to outside systems are limited by their own permissions: upload only, email to one address, only files the agent created |
| **Limited networking** | Only the hosts in section 10 |
| **A pinned version** | All three schedules run on the same version. The version changes only after testing |
| **A rehearsal** | Dec 4, 2026, a manual run in `DRY_RUN` before the schedule is turned on |
| **A finish line** | The DONE definition in section 4, and a success rubric: "every video passed QA, no duplicate topic, every fact has a source" |

### My review (outside the automation)

* After the Sunday email I review every video of the week, while they are already on YouTube, private and scheduled.
* If a video has a problem, **the decision is mine**: cancel the schedule, remake it, or leave it.
* A remake happens outside the automation, not through the agent. The agent never deletes or replaces a video on its own.
* When I replace a video, I update its row in the content plan, so the agent knows in the next run what really happened.
* A channel rule: videos that are already published are not re-uploaded. While a video is still private and scheduled the risk is smaller, but it is better to fix it before its publish time.

### What the agent must not touch

* Videos already on the channel: no deleting, replacing or editing.
* The channel's settings, logo, banner and description.
* Old rows in the content plan: it only adds rows and updates status.
* Accounts: no signing up, no signing in, no entering passwords.

### How I monitor

| Monitoring channel | When |
|---|---|
| **The session viewer** in the console | After every Saturday run in the first month: what the agent did, what it cost, where it got stuck |
| **Webhook** | When the session itself stops (budget ran out, failure), fails, or a login stops refreshing. The platform rings a small scenario in Make, and Make emails me an alert at once. This way I get an alert even when the agent itself cannot send an email |
| **Run records** | "Did it run last Saturday?" and how much it cost this month |

---

## 17. Risks and prerequisites

| Risk | Why it matters | What we do |
|---|---|---|
| **A new YouTube API project that has not passed the audit** | This has nothing to do with my approval of the videos. YouTube checks **the software that uploads** (the API project in Google Cloud), not the content. Under YouTube's policy, videos uploaded through the API from a project that has not passed the audit are locked as private, and then scheduling will not work | Apply for the audit as early as possible. Until it is approved, the agent uploads as private and I publish manually. **Check before Dec 5** |
| **Daily API quota** | Uploading a video "costs" many units of the daily quota. 6 videos in one day may get close to the limit | Check the quota at setup. If needed, split the uploads between Saturday and Sunday or ask for an increase |
| **Advanced features not available on the account** | No links in descriptions and no custom thumbnails | Already in the rules. Check again once a month |
| **Sites that block reading** | Reddit and closed Facebook groups | Search instead of direct reading. Whatever we could not reach is noted as a gap |
| **A long production run** | 6 videos in one session is a lot of work and a lot of context | Option: split into sub-agents, one per video. Decide after the rehearsal |
| **The platform is in beta** | Memory stores and other features may change | Check the screens at setup |
| **No human approval before upload** | A mistake could reach the channel | Upload as private and scheduled, the quality gate, and the Sunday email |

---

## 18. Setup plan on the platform

Following the lesson's build order (slide 16): **Agent → Environment → Vault → one session by hand → Deployment**.

| # | Step | By |
|---|---|---|
| 1 | Build the custom tools for the YouTube Data API and apply for the audit | Nov 29 |
| 2 | Build the `birding-video-production` skill from the existing skill, with the changes in section 11 | Dec 1 |
| 3 | Create the two memory stores and fill the rules store | Dec 1 |
| 4 | Create the agent (Agents), version 1 | Dec 2 |
| 5 | Create the room (Environments) with limited networking | Dec 2 |
| 6 | Create the vault and connect every login (I sign in myself) | Dec 2 |
| 7 | Create the content plan and the folder in Drive | Dec 2 |
| 8 | A small manual session: one video only, in `DRY_RUN` | Dec 3 |
| 9 | A full rehearsal in `DRY_RUN`, read the whole run, set the budget | Dec 4 |
| 10 | Create the three schedules and turn them on | Dec 4 |
| 10a | Create the webhook and the alert scenario in Make, and test it by stopping a session on purpose | Dec 4 |
| 11 | **First run** | **Dec 5, 09:00** |
| 12 | **First email, first video** | **Dec 6** |

---

## 19. Open decisions

1. **Which day does the long video go up?** Proposal: Wednesday.
2. **After a notice email about an external platform,** the current plan is that the agent does not use it, and switches to a visual built in code or to a backup topic. Should it wait for my answer instead?
3. **One big Saturday run, or a split** into a sub-agent per video?
4. **Publish times:** keep 10:30 ET, or try another hour in the first weeks?

---

## Appendix A: The agent's instructions (System prompt)

Following the lesson's shape (slide 8): who, steps, rules, done.

```text
WHO
You produce a client's YouTube channel for US birdwatchers. Every week
you plan, produce, upload and schedule 5 Shorts and 1 long video, and
keep the content plan current.

STEPS (Saturday run)
1. Read your notebook (last-run.md, lessons.md) and the channel rules.
2. List every video already on the channel and every row in the
   content plan. Never repeat a topic.
3. Find what is seasonal in US birding next week.
4. Research with the video-research-sources skill, tiers A to E.
5. Choose 5 Short topics, 1 long topic and 2 backups. Write them to
   the content plan as "planned".
5a. Plan the media for each video. Check the ElevenLabs credit
   balance. If any video would need an external platform other than
   ElevenLabs (for example Higgsfield) or the credits are not enough
   for the week, send ONE notice email before producing anything:
   video, platform, why, estimated credits. Then do not use that
   platform: build the visual in code or switch to a backup topic.
6. Produce each video with the birding-video-production skill
   and run its full quality gate. Up to 3 fix rounds per video.
7. Save each video and its upload package to this week's Drive folder.
8. Upload each passing video as private with publishAt (Sun-Thu,
   US Eastern time). Mark it "scheduled" in the content plan.
9. Read the status of every uploaded video. If YouTube rejected it
   or locked it as private, mark it "blocked" and send ONE alert
   email that lists every problem found in this run.
10. Update last-run.md and lessons.md.

STEPS (Sunday run)
1. Read this week's "scheduled" rows in the content plan.
2. Send one email in the owner's language: title, publish date, link
   per video, plus one line about any "needs_a_look" or "blocked" item.

STEPS (daily publish check, Sunday to Thursday)
1. Read today's "scheduled" rows in the content plan.
2. Check each video's status on YouTube.
3. Public: mark it "published". Not public: mark it "blocked".
4. If anything is "blocked", send ONE alert email in the owner's
   language: title, planned publish time, what happened, Studio link.

RULES
- Never invent a fact, number, quote or source.
- Only licensed media (PD, CC0, CC BY). Credit everything.
- Disclose the AI voice (synthetic content = yes).
- No URLs in descriptions. No custom thumbnails.
- Never delete, replace or edit a video that is already on the channel.
- Never edit old rows in the content plan; add rows and update status.
- Never put keys or passwords in memory or in any file.
- Before retrying an upload, check that the video is not already there.
- A video that fails the quality gate is not uploaded: mark it
  "needs_a_look" and produce a backup topic instead.
- All production happens inside Claude. The only external production
  service is ElevenLabs: narration (always the Aris voice, with the
  voice ID and settings from the channel rules), sound effects and
  background music.
- If YouTube blocks a video, never try to work around it: mark it
  "blocked" and send the alert email.
- Whenever a video gets stuck for any reason (credits, a tool failure,
  the quality gate, the upload), send the alert email IMMEDIATELY,
  before moving on: video, step, what happened, what you did next.
- If a case is not covered here, stop and report. Do not improvise.
- If DRY_RUN is in the first message: do everything except the
  YouTube upload and the email.

DONE
Saturday: 6 rows in the content plan, every passing video scheduled,
files in Drive, notebook updated, a one-line report.
Sunday: one email sent.
Daily check: every video of the day marked "published", or one
alert email sent about the ones that are not.
```

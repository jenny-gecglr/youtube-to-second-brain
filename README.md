# YouTube to Second Brain

> Turn a YouTube channel or video into verified, linked notes in your own knowledge base, without spending your best model's budget.

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Claude Code skill](https://img.shields.io/badge/Claude%20Code-skill-blueviolet)
![Approval first](https://img.shields.io/badge/notes%20written%20only%20after-your%20approval-brightgreen)

**Contents:** [Why](#why-this-exists) · [How it works](#how-it-works) · [Who does what](#who-does-what) · [What you get](#what-you-get) · [Video selection](#video-selection-rules) · [Guardrails](#guardrails) · [Requirements](#requirements) · [Install](#install) · [Example prompts](#example-prompts) · [FAQ](#faq) · [Limitations](#limitations) · [License](#license)

## Why this exists

Learning from a whole YouTube channel by hand takes days. Pasting every transcript into your strongest model is slow, expensive and invites invented details. This skill splits the job so that each tool does only what it is good at:

- **Scripts and NotebookLM** collect the video list, the verbatim transcripts and the summaries.
- **A cheap model** sorts files and pairs summaries with links.
- **A stronger model** only verifies, sets context and presents the result.
- **You** approve before anything is written to your knowledge base.

## How it works

```mermaid
flowchart TD
    A["Channel or video link"] --> B["Script lists videos<br/>(yt-dlp)"]
    B --> C["NotebookLM:<br/>verbatim transcripts<br/>and summaries"]
    C --> D["Cheap model<br/>organizes files"]
    D --> E["Script + stronger model<br/>verify"]
    E --> F{"You approve?"}
    F -- yes --> G["Notes written to<br/>your knowledge base"]
    F -- no --> H["No notes written<br/>(only the raw folder is filled)"]
```

| # | Step | What happens |
|---|---|---|
| 1 | Start | Reads your knowledge base's rules and recent log (if it has them). |
| 2 | List | A script lists the channel's videos (and Shorts, if you choose to include them) and applies the selection rules below. |
| 3 | Links and chapters | Description links and chapter markers are extracted into their own files. |
| 4 | Notebook | All videos of a channel go into one NotebookLM notebook named after the channel. |
| 5 | NotebookLM jobs | Verbatim transcript, bullet summary, rules and lessons, workflow, and exercises when the video is educational. |
| 6 | Organize | A cheap model sorts files; it never edits transcripts. |
| 7 | Verify | A script greps numbers and names against the transcript, then a stronger model spot-checks and writes a check report. |
| 8 | Present | You see videos, context clusters, rules, workflows, links, the check report and any gaps. |
| 9 | Approval gate | Nothing is written to pages, index or log until you say yes. |
| 10 | Write | Source cards, lesson pages, rule pages and a hub page, then index and log updates. |

## Who does what

| Who | Responsible for |
|---|---|
| Scripts (`yt-dlp`, `nlm`) | Lists, exact view counts, verbatim transcripts, link and chapter extraction |
| NotebookLM | Summaries, rules, lessons, workflows, exercises |
| Cheap model or second CLI | Sorting files, simple judgment tasks |
| Stronger model | Verification, check report, context, the final presentation |
| You | Choices (Shorts, video-watching pass) and the approval to write |

## What you get

After you approve, the skill writes into your markdown knowledge base:

- **Source cards** for the videos.
- **Lesson pages** and **rule pages**, one pair per context cluster, plus one "all rules" page.
- A **hub page** that everything links to (a large channel gets its own hub; a single video goes under a topic hub).
- **Narrative formulas** (title, hook, body and call-to-action patterns) stored separately as patterns only.
- An updated **index page** and one new **log line**.

Contradictions with existing pages are shown to you separately, and claims without a source are labeled `[UNVERIFIED]`.

## Video selection rules

| Channel size | What is taken |
|---|---|
| 80 videos or fewer | All of them |
| More than 80 | The 80 most viewed in the last 6 months, plus any of the 50 most viewed of all time not already included, up to 130 in total. A second model scans for important videos the view-count rules missed. |
| Shorts | You decide per channel, after seeing how their median views compare with long videos. |

A per-channel `_processed.json` resume file records progress, so after a quota limit or an error it resumes instead of restarting, and later runs only process new videos.

## Guardrails

- Raw transcripts are never retyped, translated or edited by a model.
- No invented summaries: a video without a transcript is marked "unavailable".
- Other creators' titles and signature phrases are never copied; only patterns are kept.
- Private content is never sent to external tools.
- No deleting or bulk overwriting without your approval.
- If NotebookLM or its CLI fails, you are told; a browser session is a last resort and needs your permission.

## Requirements

| Needed | For |
|---|---|
| Claude Code (or another Claude client with skills) | Running the skill |
| `yt-dlp` | Video lists, exact metadata |
| NotebookLM and its CLI (`nlm`) | Transcripts and summaries |
| A markdown knowledge base (an Obsidian vault works well) | Where the notes are written |
| Optional: a second model CLI such as Gemini | Simple judgment tasks |

## Install

Copy the skill into your Claude skills folder:

```bash
git clone https://github.com/jenny-gecglr/youtube-to-second-brain.git ~/.claude/skills/youtube-to-second-brain
```

Or download the folder and place it at `~/.claude/skills/youtube-to-second-brain`.

## Example prompts

- "Add this channel to my second brain: `<channel link>`"
- "Summarize this video and file it in my notes: `<video link>`"
- "Check this channel for new videos since last time."

## FAQ

**Does it work without Obsidian?** Yes. Any markdown knowledge base works; Obsidian is just a convenient example.

**Will it copy another creator's content?** No. It stores patterns (for example how a hook is built), never their titles or signature phrases.

**What if a video has no transcript?** It is marked "unavailable" and listed for you. No summary is invented.

**What if I hit a quota limit halfway through a channel?** Stop and resume later; the resume file picks up where it left off.

**Can it run on a schedule?** A monthly "any new videos?" check can be offered, but it is never set up without your approval.

## Limitations

- It depends on NotebookLM and its CLI being available and logged in.
- NotebookLM reads speech only. For videos where code or screen content matters, a video-capable model can fill the gaps, on the videos you choose.
- The quality of the notes depends on the transcripts that exist for the videos.

## License

MIT. See [LICENSE](LICENSE).

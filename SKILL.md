---
name: youtube-to-second-brain
description: Turns a YouTube channel or video link into structured notes in a markdown knowledge base (for example an Obsidian vault). Heavy work (transcripts, summaries, rules, workflows) goes to NotebookLM and cheaper models; the main model only handles context, checks and approval. Use when someone says "add this channel/video to my second brain" or "summarize and file this video".
---

# YouTube to Second Brain

Goal: keep expensive model usage low. Scripts, NotebookLM and cheap models do the bulk work; a stronger model only verifies, sets context and presents the result. **Nothing is written to the knowledge base until the user approves.**

## Tools

- `yt-dlp`: channel video lists, exact view counts, dates, durations, descriptions, chapter markers. (Add `--js-runtimes node` if YouTube extraction fails.) A quick flat listing shows rounded views and no dates, so fetch exact values with a second pass per video.
- NotebookLM CLI (`nlm`): `nlm notebook query`, `nlm source content <id> -o <file>` (verbatim transcript, no model in between), `nlm source list`.
- A second model CLI (for example Gemini, or Antigravity: `agy -p "task" --print-timeout 90s`) for simple judgment tasks; write its output to a file and read only the file.

Terms used below: **cheap model** = a low-cost model or the second model CLI; **stronger model** = your main, more capable Claude model. The **raw folder** is the knowledge base's folder for unprocessed source material (raw transcripts and script outputs); if it has none, use a temporary folder.

## Setup check (first run)

Before step 1, check that every tool this skill needs is installed and logged in. For anything missing, say what it is and what it is for, then **ask before installing**. Never type or store passwords: Google sign-in is done by the user in the browser.

| Tool | Check | Install |
|---|---|---|
| `yt-dlp` | `yt-dlp --version` (or `python -m yt_dlp --version`) | `pip install yt-dlp` |
| Node.js | `node --version` | Needed by `yt-dlp --js-runtimes node` when YouTube extraction fails. Install from nodejs.org. |
| NotebookLM CLI (`nlm`) | `nlm --version` | `uv tool install notebooklm-mcp-cli` (or `pipx install notebooklm-mcp-cli`) |
| NotebookLM sign-in | `nlm notebook list` | `nlm login --storage protected`. The user signs in to Google in a separate Chrome profile. |
| NotebookLM as an MCP server (optional) | the `notebooklm` tools are listed | `claude mcp add --scope user notebooklm <path to notebooklm-mcp>`, then start a new session |
| Second model CLI (optional) | for example `gemini --version` or `agy --version` | Install the Gemini CLI or the Antigravity CLI and sign in. Without one, skip the "second model scan" and use a cheaper Claude model for simple judgment tasks. |

If a required tool cannot be installed, stop and tell the user what is missing instead of continuing with a weaker substitute.

## Steps

1. **Start.** Read the knowledge base's rules file and recent log, if it has them. Never put personal data or employer names into prompts for other tools.
2. **List (script; the model does not read the output).** Get the channel's /videos and /shorts lists and verify the count.
   - 80 videos or fewer: take all.
   - More than 80: take the 80 most viewed in the last 6 months, then take the 50 most viewed videos of all time and add those not already in the 80, up to 130 videos in total. Have a second model scan the full list for important videos that the view-count rules missed. Apply this selection every time a channel has more than 80 videos.
   - Shorts: compare the median views of Shorts with those of long videos, show the user, and let the user decide per channel whether to include them.
   - **Resume file:** keep `<raw folder>/transcripts/<channel>/_processed.json` per channel (video id and step status: added, transcript fetched, summary fetched, checked). On quota or errors, resume from it instead of restarting.
3. **Description links (script).** Extract links from descriptions into a links file with the video title. Pull chapter markers ("0:00 Intro...") into their own file to verify the summary structure. Optionally fetch top comments for corrections: test first that it works, and mark anything taken from comments as unverified.
4. **Notebook rule.** All videos of one channel go to the same NotebookLM notebook, named after the channel. For single videos, look for a fitting notebook first, otherwise create one, and show the user which video went where.
5. **NotebookLM jobs (scripted; the model does not read the output).** For each notebook, request the following and save each result to a file:
   - verbatim transcript (`nlm source content -o`, saved under `<raw folder>/transcripts/<channel>/` with a title header line; never retyped or paraphrased by a model),
   - detailed bullet summary (hook, main thesis, sections, tools/models, lessons, call to action, narrative patterns),
   - rules and lessons,
   - step-by-step workflow if one exists, rewritten for the user's own tools,
   - exercises or practice questions if the video is educational.

   Handling the results:
   - Ask for large answers one at a time, each in a new conversation (`--new-conversation`).
   - Label NotebookLM summaries as such; only `source content` output is verbatim.
   - Relevant description links go into the summary; researching a link is delegated to the second model CLI only after asking the user.
   - If no transcript is available, do not invent a summary: mark the video "unavailable" in the resume file and show it in the list.
   - NotebookLM reads speech only. For key videos where code or screen content matters, a video-capable model can fill the gaps. Ask the user which videos get this extra pass, and test first that the model can actually watch video.
6. **Move and organize (cheap model).** A cheap model sorts files into a temporary folder and pairs summaries with links. It never edits transcripts.
7. **Verify (stronger model).** A script first greps numbers and names in the transcript and flags anything not found. Then a stronger model spot-checks the cheap model's work for invented or missing content and writes a short check report (what it fixed, dropped, doubted). Unchecked work does not pass.
8. **List and context.** Present the videos, context clusters, rules, workflows, links, check report and gaps, then:
   - Compare new rules with existing pages so nothing is duplicated: link to existing pages, and show any contradiction to the user separately.
   - Label claims `[SOURCED]` when a video supports them and `[UNVERIFIED]` when they have no source.
   - Use a **hub page** (the main page that other pages link to): a large channel gets its own hub; a single video goes under a topic hub (an existing one, otherwise a new one). Each context cluster gets a lesson page and a rule page, plus one "all rules" page, all linked to the hub.
   - End with one question: "Write this to the knowledge base?"
9. **Approval gate.** Before the user approves, nothing is written to the knowledge base pages, its index page or its log. Only the raw folder may be filled.
10. **Write (after approval).** Source cards, lesson pages and rule pages, all linked to the hub page. Store narrative formulas (title, hook, body, call-to-action patterns) separately: note the pattern only, never copy another creator's titles or signature phrases. Use meaningful page names, then update the index page and append one line to the log. Run the knowledge base's own validation routine, if it has one, and fix what it reports until it shows zero errors. If you add a new folder and your graph view uses color groups, let the user pick the color. Finish with a short report to the user.

## New videos later

When the same channel is given again, compare against `_processed.json` and process only the new videos into the existing notebook and hub. A monthly automatic "any new videos?" check may be offered, but never set up without the user's approval.

## Don't

- Don't have a model retype, translate or edit raw transcripts.
- Don't insist after a quota error or invent summaries; tell the user.
- Don't send private content to external tools.
- Don't ingest images or thumbnails. Don't rewrite the user's own titles or thumbnails.
- Don't delete or bulk overwrite without approval.

## Fallbacks

If NotebookLM or its CLI fails, tell the user. Use a browser session only as a last resort, since it is expensive; ask first. An alternative transcript tool may serve as a backup only if its quota allows.

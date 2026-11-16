---
name: youtube-to-second-brain
description: Turns a YouTube channel or video link into structured notes in a markdown knowledge base (for example an Obsidian vault). Heavy work (transcripts, summaries, rules, workflows) goes to NotebookLM and cheaper models; the main model only handles context, checks and approval. Use when someone says "add this channel/video to my second brain" or "summarize and file this video".
---

# YouTube to Second Brain

Goal: keep expensive model usage low. Scripts, NotebookLM and cheap models do the bulk work; the strongest model only verifies, sets context and presents the result. **Nothing is written to the knowledge base until the user approves.**

## Tools

- `yt-dlp`: channel video lists, exact view counts, dates, durations, descriptions, chapter markers. (Use `--js-runtimes node` if needed.) The fast list shows rounded views and no dates; fetch exact values with a second pass per video.
- NotebookLM CLI (`nlm`): `nlm notebook query`, `nlm source content <id> -o <file>` (verbatim transcript, no model in between), `nlm source list`.
- A second model CLI (for example Gemini) for simple judgment tasks; write its output to a file and read only the file.

## Steps

1. **Start.** Read the knowledge base's rules file and recent log. Never put personal data or employer names into prompts for other tools.
2. **List (script, nobody reads it).** Get the channel's /videos and /shorts lists and verify the count.
   - 80 videos or fewer: take all.
   - More than 80: take the 80 most viewed in the last 6 months, then add the 50 most viewed of all time that are not already in those 80 (have a second model check for missed important videos), up to 130 videos in total. Neither rule is skipped.
   - Shorts: compare Shorts' median views with long videos' and let the user decide include/exclude per channel.
   - **Resume file:** keep `_processed.json` per channel (video id, step status). On quota or errors, resume from it instead of restarting.
3. **Description links (script).** Extract links from descriptions into a links file with the video title. Pull chapter markers ("0:00 Intro...") into their own file to verify the summary structure. Optionally fetch top comments for corrections (mark as unverified).
4. **Notebook rule.** All videos of one channel go to the same NotebookLM notebook, named after the channel. For single videos, look for a fitting notebook first, otherwise create one, and show the user which video went where.
5. **NotebookLM jobs (scripted).** For each notebook request and save to files:
   - verbatim transcript (`nlm source content -o`; never retyped or paraphrased by a model),
   - detailed bullet summary (hook, main thesis, sections, tools/models, lessons, call to action, narrative patterns),
   - rules and lessons,
   - step-by-step workflow if one exists, rewritten for your tooling,
   - training-style content if the video is educational.
   Ask for large answers one at a time, each in a new conversation. Relevant description links go into the summary; researching a link is delegated to a second model only after asking the user. If a transcript cannot be fetched, record "unavailable" in the resume file too. Label NotebookLM summaries as such; only `source content` output is verbatim. If no transcript is available, do not invent a summary: mark it "unavailable" and show it in the list. NotebookLM reads speech only, so for key videos where code or screen content matters, a video-capable model can fill the gaps. Ask the user which videos get this extra pass, and test first that the model can actually watch video.
6. **Move and organize (cheap model).** A cheap model sorts files into a temporary folder and pairs summaries with links. It never edits transcripts.
7. **Verify (strongest model).** A script first greps numbers and names in the transcript and flags anything not found. Then the strongest model spot-checks the cheap model's work for invented or missing content and writes a short check report (what it fixed, dropped, doubted). Unchecked work does not pass.
8. **List and context.** Present the videos, context clusters, rules, workflows, links, check report and gaps. Compare new rules with existing pages so nothing is duplicated (link to existing pages; flag contradictions). Use a **hub page** (the main page that other pages link to): a large channel gets its own hub; a single video goes under a topic hub (an existing one, otherwise a new one). Each context cluster gets a lesson page and a rule page, plus one "all rules" page, all linked to the hub. Label conflicts and unsourced claims as `[SOURCED]` or `[UNVERIFIED]`. End with one question: "Write this to the knowledge base?"
9. **Approval gate.** Only the raw folder may be filled before approval.
10. **Write (after approval).** Source cards, lesson pages and rule pages, all linked to the hub page. Store narrative formulas (title, hook, body, call-to-action patterns) separately: note the pattern only, never copy another creator's titles or signature phrases. Use meaningful page names, then update the index page, append one line to the log, and run your vault health check (it must report zero errors). If you add a new folder and your graph view uses color groups, let the user pick the color. Finish with a short report to the user.

## New videos later

When the same channel is given again, compare against `_processed.json` and process only the new videos into the existing notebook and hub. A monthly automatic "any new videos?" check may be offered, but never set up without the user's approval.

## Don't

- Don't have a model retype, translate or edit raw transcripts.
- Don't insist after a quota error or invent summaries; tell the user.
- Don't send private content to external tools.
- Don't ingest images or thumbnails. Don't rewrite the user's own titles or thumbnails.
- Don't delete or bulk overwrite without approval.

## Fallbacks

If NotebookLM or its CLI fails, tell the user. Use a browser session only as a last resort, since it is expensive; ask first.

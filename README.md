# YouTube to Second Brain

A Claude skill that turns YouTube channels and videos into verified, linked notes in your own markdown knowledge base (for example an Obsidian vault), without spending your best model's budget.

## What problem it solves

Watching and summarizing a whole channel by hand takes days, and asking a top-tier model to read every transcript is expensive and error-prone. This skill splits the work: scripts and NotebookLM do the heavy lifting (lists, verbatim transcripts, summaries), a cheap model organizes files, and a stronger model only verifies and presents. **Nothing is written to your knowledge base until you approve it.**

## How it works

1. **Scripted video list** with `yt-dlp`: up to 80 recent top videos, plus the all-time top 50 not already included, capped at 130. Shorts are included only if you decide so.
2. **One NotebookLM notebook per channel.** Verbatim transcripts come straight from the source content (no model in between); summaries, rules, lessons and workflows are requested from NotebookLM, with large answers asked for one at a time.
3. **Cheap model organizes**, a **script greps** numbers and names against the transcript, and a **stronger model spot-checks** and writes a check report.
4. **You see the result first:** videos, context clusters, rules, workflows, links, check report and gaps. Only after your "yes" are lesson pages, rule pages and a hub page written.
5. **Resumable:** a per-channel `_processed.json` lets you continue after a quota limit or error, and later runs only process new videos.

## Guardrails

- Never retypes, translates or edits raw transcripts.
- No invented summaries: videos without a transcript are marked "unavailable".
- Narrative patterns (title, hook, call-to-action formulas) are stored as patterns only; other creators' titles and signature phrases are never copied.
- No deleting or bulk overwriting without approval; private content is never sent to external tools.

## Requirements

- Claude Code (or any Claude client that supports skills)
- `yt-dlp`
- NotebookLM and its CLI (`nlm`)
- A markdown knowledge base (an Obsidian vault works well)
- Optional: a second model CLI such as Gemini for simple judgment tasks

## Install

Copy the `youtube-to-second-brain` folder into `~/.claude/skills/`.

## Example prompts

- "Add this channel to my second brain: <channel link>"
- "Summarize this video and file it in my notes: <video link>"
- "Check this channel for new videos since last time."

## Files

- `SKILL.md`: the full step-by-step workflow Claude follows.

## License

MIT. See [LICENSE](LICENSE).

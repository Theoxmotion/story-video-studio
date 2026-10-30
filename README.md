# Story Video Studio: one Claude prompt that makes an animated survival story video

Claude writes, draws, voices and renders a vertical animated story of about 65 seconds, ready for Reels, Shorts or TikTok:
- the hero, the animal and the threat look the same in every shot;
- one real survival rule and a fact with a number;
- a hard choice, a cost, a twist;
- an ending people argue about in the comments.

Every frame is drawn in code. There are no image or video models and no stock art.

## What you need

A Claude subscription (Claude Code is included with Pro and Max). Nothing else: no other accounts, API keys or paid tools.

Claude installs the free tools itself:
- Python packages;
- ffmpeg;
- a free voice model that runs on your own computer, downloaded once (about 500 MB).

## Make a video

1. Make an empty folder and open Claude Code in it.
2. Paste the whole of `story_video_prompt.md`. Claude saves it as `CLAUDE.md` in that folder.
3. Claude pitches 10 story ideas and scores them. Pick one, or answer "you pick".
4. Wait. Claude sets up, writes the script, checks it against the story checklist, voices it, draws it, checks stills, renders, and checks the pacing.
5. You get:
   - `part1/part1.mp4`: 1080x1920, about 65 s, under 10 MB;
   - `part1/cover.jpg`;
   - `part1/post.md`: the caption, the vote question and hashtags.

**How long it takes:** in our tests, one video took a fresh Claude Code session about 45 minutes from an empty folder, setup included, with nobody helping.

**Music:** every video gets a dark background music bed made in code (no downloaded tracks), ducked under the voice. Say "no music" to turn it off.

**Before you post:** listen to it once. Claude can't hear audio, so the voice and the mix are the one thing only you can check.

**Want a Part 2?** In the same folder, say "make part 2" and how the vote went. The cast and the numbers carry over.

## Rate it: good or ass

`rater/` holds the AI animation rater:
- `rate_prep.py` pulls the frames and numbers out of your MP4;
- `RATER_PROMPT.md` makes Claude score it out of 120, give a plain verdict and list the 3 fixes that would raise it most.

`rater/HOWTO.md` explains it. We checked the rater against a real series that went viral (3 parts, all GOOD) and against a deliberately bad video (ASS).

## Proof: what fresh Claude sessions made from this prompt

Each run used an empty folder, with "you pick" as the only answer. The ratings come from separate fresh sessions using the rater, blind.

| video | story / craft | total | verdict |
|---|---|---|---|
| `test_videos/replication_A_snowbound.mp4` (62 s) | 56 / 55 | 111 / 120 | GOOD |
| `test_videos/replication_B_night_desk.mp4` (61 s) | 55 / 55 | 110 / 120 | GOOD |
| `test_videos/hero_floor7_part1.mp4` (70 s, male narrator) | 56 / 51 | 107 / 120 | GOOD |
| for comparison: the viral reference series, Parts 1 to 3 | 45 to 49 / 48 to 51 | 96 to 99 | GOOD |

The hero was rated before its music bed was added. With the music, voice clarity went from 0.867 to 0.871 and loudness stayed at -14.3 LUFS.

## Files

| file | what it is |
|---|---|
| `story_video_prompt.md` | the one prompt to paste (with the 4 engine scripts inside) |
| `rater/rate_prep.py`, `rater/RATER_PROMPT.md`, `rater/HOWTO.md` | the rater |
| `test_videos/` | videos fresh Claude sessions made from this prompt |

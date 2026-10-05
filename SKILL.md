---
name: "mv_free_pipeline"
description: "End-to-end workflow for producing a full-length music video on raphael.app's free tier: song analysis, storyboard, free image keyframes, chained 4s AI video segments, and ffmpeg assembly. Use when the user asks for a music video, MV, or long AI video built without paid credits."
---

# MV Free Pipeline

## Purpose
Produce a complete, full-song music video spending zero credits: raphael.app free image keyframes + free MiniMax H3 Turbo 480P/4s video segments chained via last-frame continuation + ffmpeg assembly. Proven on a 4:10, 29-shot, 70-segment MV.

## Workflow
1. **Song analysis** — duration, BPM, energy/transient curve per 5s; split into 4–8 acts aligned to energy shifts. See `references/song-and-storyboard.md`.
2. **Storyboard** — shots with target durations summing to song length; each shot gets an image prompt (character bible + style bible appended) and a motion prompt. Lock the storyboard with the user before generating.
3. **Keyframes** — raphael.app free image tier, 2–4 variants per shot, pick one HERO per shot (80% rule: only regenerate on clear failure). Crop the free-tier badge (832×512 → 832×468). See `references/raphael-free-tier.md`.
4. **Chained video generation** — free 480P/4s/0-credit image-to-video; chain segments per shot via last-frame extraction + `"The scene continues: <motion prompt>"`. Strictly serial: exactly 1 in-flight segment. QC every segment. See `references/chain-generation.md`.
5. **ffmpeg assembly** — concat demuxer (never xfade chains), per-shot retime to target durations, baked white-flash transitions, teal-orange grade + 35mm grain, titles via drawtext, original song as audio. See `references/ffmpeg-assembly.md`.
6. **Delivery** — verify duration/frames/specs + spot-check frames; deliver a share-friendly encode; archive everything (clips, edits, finals, docs) to Google Drive.

## Output Contract
- Final MP4: 16:9, 1080P, 24fps, H.264 + AAC, duration == song duration (±0.1s), faststart.
- Original song as the main audio track (no voiceover covering it); titles only at open/end unless the user asks for lyric subtitles.
- Delivery report: file path, exact duration, resolution/fps/codec, file size, and an honest note that 480P source upscaled to 1080P is soft.

## Operating Rules
1. Free tier only unless the user explicitly approves spending credits. Never click paid models; verify "0 credits" on Generate before each submit.
2. No reverse-engineering, no API abuse, no rate-limit evasion. Ordinary manual-speed usage.
3. Never fabricate lyrics or burn subtitles without a verified lyric source.
4. Never use still-image pan/zoom + crossfade slideshows to pad runtime — only real generated video, retimed.
5. One in-flight generation at a time; re-confirm queue state from the tail of the ledger before every submit/download.
6. Report progress numbers-first (`n/N`); report exceptions as outcomes, not tool errors.

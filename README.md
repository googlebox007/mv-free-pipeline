# mv-free-pipeline

End-to-end workflow for producing a **full-length music video spending zero credits**: free image keyframes + free 4-second AI video segments chained via last-frame continuation + ffmpeg assembly.

Proven on a real delivery: a 4:10, 29-shot, 70-segment MV (*Against the Headwind*), built entirely on raphael.app's free tier (MiniMax H3 Turbo, 480P/4s/0 credits).

## How it works

1. **Song analysis** — energy/transient curve → act structure.
2. **Storyboard** — shots with target durations summing to the song length (locked with the client first).
3. **Keyframes** — free-tier image generation, one HERO frame per shot.
4. **Chained video generation** — each 4s segment continues from the previous segment's last frame (`The scene continues: …`), strictly one in-flight segment at a time, QC on every segment.
5. **ffmpeg assembly** — concat (no xfade chains), per-shot retiming, baked white-flash transitions, teal-orange grade + 35mm grain, titles, original song as audio.
6. **Delivery** — spec verification, share-friendly encode, archive to cloud storage.

## Contents

- `SKILL.md` — the operational workflow (for AI agents).
- `references/raphael-free-tier.md` — measured free-tier facts, site quirks, pre-submit checklist.
- `references/chain-generation.md` — the chaining method, prompts, QC checklist, serial discipline.
- `references/song-and-storyboard.md` — song analysis and storyboard method, timeline math.
- `references/ffmpeg-assembly.md` — assembly recipes, retime math, grade, titles, transitions, and three hard-won pitfalls (inverted setpts factor, xfade swallowing frames, `aac` used as a filter).

## Notes

- Free video is for personal, non-commercial use.
- No reverse-engineering or rate-limit evasion — ordinary manual-speed usage.
- 480P source upscaled to 1080P is soft; that's the price of free.

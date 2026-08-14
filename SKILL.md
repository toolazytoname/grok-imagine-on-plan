---
name: imagine-on-plan
description: >
  Generate images and videos on a SuperGrok / Grok coding-plan quota by
  calling Grok Build (native Imagine tools, or `grok -p` from any other
  agent). Use when the user asks to generate, draw, or make an image,
  cover, poster, still, clip, or video; says /imagine, /imagine-video,
  /imagine-on-plan; wants another agent to use Grok; mentions 套餐、额度、
  封面、竖屏、720p、图生视频; or wants Imagine without paying the xAI API.
user-invocable: true
---

# Imagine on plan

Spend the logged-in SuperGrok / coding-plan weekly pool. Never spend console API credits.

Do not call `https://api.x.ai`. Do not use `XAI_API_KEY`, `xai_sdk`, or an OpenAI-compatible Imagine client. Unset `XAI_API_KEY` on any `grok` process you spawn.

There is no text-to-video. Video = still first, then animate.

---

## If you are not Grok Build

You do not have `image_gen`. Drive the user's **already-logged-in** `grok` CLI.

1. `command -v grok` (common path: `~/.grok/bin/grok`). Missing → tell the user to install Grok Build and `grok login`.
2. Confirm `~/.grok/auth.json` exists. Missing → `grok login`. Do not fall back to an API key.
3. Decide settings from **Phrase → settings** below. Put them in the prompt; `grok -p` has no `--aspect-ratio` flag.
4. Run (prefer `--prompt-file` so quotes survive):

```bash
unset XAI_API_KEY
grok -p --yolo --verbatim \
  --tools "image_gen,image_edit,image_to_video,reference_to_video" \
  --max-turns 8 \
  --prompt-file /tmp/imagine-on-plan-prompt.txt
```

Prompt file shape:

```
Follow imagine-on-plan. Use only the allowed Imagine tools. Do not call api.x.ai.

Task: <user ask>
Still prompt: <verbatim or drafted 2–5 sentences>
aspect_ratio: 9:16
If video: duration 6, resolution_name 480p
Motion (one only): A slow cinematic push-in. Subject holds still.

Print every output file as an absolute path. Do not describe the pixels.
```

5. Parse absolute `images/` / `videos/` paths from stdout. Copy into the user's dest if they named one.
6. On ZDR / `output.upload_url`, quote it and stop.

Outputs live under `~/.grok/sessions/<urlencoded-cwd>/<session-id>/images|videos/`.

`grok -p` is itself an agent turn (Build quota + Imagine). Keep the prompt tight. One still or one clip per invocation unless the user asked for a sequence.

---

## If you are Grok Build

Call the tools yourself. Do not shell out to another `grok`.

| Need | Tool |
|---|---|
| New still | `image_gen` |
| Change an existing still | `image_edit` |
| Animate one still | `image_to_video` |
| Multi-ref or preset voice | `reference_to_video` |

Verbatim user prompts go into `prompt` unchanged.

On ZDR / `output.upload_url`, quote it and stop.

After success, give the saved path. Do not narrate the pixels.

Charts, exact text, numbers, comic grids: code, not `image_gen`.

Named real people: `image_edit` from a real reference. Never `image_gen` a named person.

Recurring character: one hero still, then `image_edit` / seed video from that file.

---

## Image

`image_gen`: `prompt` (required), `aspect_ratio` (optional). No `quality`, `resolution`, `n`, or `count`. Extra stills = extra calls.

`image_edit`: `prompt` (what changes + what stays), `image` (path, https, or data URI). Single-image edits keep the source ratio.

`aspect_ratio`: `1:1` avatar; `16:9` landscape / YT cover; `9:16` phone / Story; `4:3` `3:4` `3:2` `2:3` photo; `auto` only when the user does not care.

### Draft vs cover

No quality slider. Quality is ratio + prompt + optional edit.

**Draft** (sketch, inline, "just look"): one `image_gen`, short prompt (subject + place + light), matching ratio, no second pass.

**Cover** (cover, poster, hero, "ship it"):

1. Lock ratio. Never `auto`.
2. One large subject, mid-shot, readable as a thumbnail.
3. Concrete light (rim, golden hour, candle, Rembrandt) + shallow DOF.
4. No poster lettering in the prompt — type goes on later.
5. If close: `image_edit` one issue. If choosing: two `image_gen` variants, pick the clearer silhouette.

Skip busy wide shots, crowds, mirrors, and ten-element collages for covers.

---

## Video

Default: one clip. Longer / multi-shot / narrative: many 6s shots, then concat.

### One clip

1. Stage frame 1 with `image_gen` / `image_edit` (or the user's still). Ratio is decided here.
2. `image_to_video` that file.
3. Return the mp4 path.

`image_to_video`:

| Arg | Values | Default |
|---|---|---|
| `image` | path / https / data URI | required |
| `prompt` | present tense, 1–2 sentences, **one** move | optional |
| `duration` | `6` or `10` only | `6` |
| `resolution_name` | `480p` or `720p` | `480p` |

No 1080p, bitrate, or fps knobs. Do not crop in the video call.

`reference_to_video` only if the user names it, needs several refs, or a preset voice (`ara`, `eve`, `leo`, `rex`). Tag `<IMAGE_0>` / `<AUDIO_0>`. `duration` 1–15 (default 6). `aspect_ratio` required. Prefer compose with `image_edit` then `image_to_video`.

### Quality

| Intent | Settings |
|---|---|
| Preview / save quota | Draft or simple still + `480p` + `6` |
| Ship | Cover-grade still + `720p` + `6` |

Warp comes from a busy still or a multi-action prompt, not from 480p. Simplify the still or freeze the subject and move only the camera.

Use one of these prompts, not two at once:

- Slow cinematic push-in. Subject holds still.
- Slow pull-back revealing the room. Subject holds still.
- Camera orbits a few degrees right. Subject stays still.
- Subtle sideways parallax; subject locked.
- Hair / silk / candlelight moves. Camera locked off.
- Head turns a few degrees toward camera. Nothing else.

Do not prompt walk-across-the-room, fights, fine hand work, long lip-sync, or push+pan+crane.

### Multi-shot

One beat per 6s shot. Same hero still via `image_edit`. Same `resolution_name` on every clip. Concat with stream copy only:

```bash
ffmpeg -f concat -safe 0 -i list.txt -c copy out.mp4
```

`list.txt` lines: `file 'shot-01.mp4'`. Do not loop a clip unless asked.

---

## Phrase → settings

| User | Do |
|---|---|
| 竖屏 / 手机 / 抖音 / Story | Frame `9:16` |
| 横屏 / 电影 / YouTube | Frame `16:9` |
| 方图 / 头像 | `1:1` |
| 普通图 / 草稿 | One short `image_gen` |
| 封面 / 海报 / 高质量图 | Cover recipe; optional 2-pick + edit |
| 短 | `duration: 6` |
| 长一点（单条） | `10` on `image_to_video` |
| 预览视频 | `480p` + `6` + one slow move |
| 能发 / 高清 / 720p | Cover still + `720p` + `6` |
| 1080p | Cap is 720p. Stay on Grok tools / `grok -p`. |
| 说话 / 指定声线 | `reference_to_video` + `voices` |
| Still only | No video |
| Video only | Still first anyway |

Unspecified video: `6` + `480p`.

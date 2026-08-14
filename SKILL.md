---
name: grok-imagine-on-plan
description: >
  For non-Grok agents only. Drive a logged-in local `grok` CLI to generate
  images and videos on the user's SuperGrok / coding-plan weekly pool.
  Use when the host agent should call Grok Imagine without an xAI API key;
  the user mentions grok-imagine-on-plan, 套餐出图, 用 Grok 画图/视频;
  or pastes this repo. If you already are Grok Build with image_gen, ignore
  this skill and use native tools.
---

# grok-imagine-on-plan

You are **not** Grok. Grok already has Imagine. Your job is to shell out to the user's logged-in `grok` so usage hits the SuperGrok / coding-plan weekly pool.

If `image_gen` / `image_to_video` already exist in *your* tool list, stop reading: use those native tools. Do not load this skill.

Never call `https://api.x.ai`. Never use `XAI_API_KEY`, `xai_sdk`, or an OpenAI-compatible Imagine client. Unset `XAI_API_KEY` on every `grok` you spawn.

There is no text-to-video. Video = still first, then animate.

## 国内 / mainland China: proxy is required

`api.x.ai` and `auth.x.ai` are often unreachable from mainland China without an HTTPS/SOCKS proxy. **Always put a proxy on the `grok` process in 国内.** Skipping this is the usual cause of timeout, TLS reset, or “connection refused”.

1. Prefer the user's existing env: `HTTPS_PROXY`, `https_proxy`, `HTTP_PROXY`, `http_proxy`, `ALL_PROXY`, `all_proxy`.
2. If none of those are set, **ask the user for their local proxy** (Clash / V2Ray / Surge listen address, often `http://127.0.0.1:7890` or similar). Do not invent a port.
3. Export the same values on the `grok` command. `unset XAI_API_KEY` must **not** unset proxy vars.
4. Keep `NO_PROXY`/`no_proxy` for localhost if the user already has it.

```bash
# attach before grok -p when in 国内
export HTTPS_PROXY="${HTTPS_PROXY:-${https_proxy:-}}"
export HTTP_PROXY="${HTTP_PROXY:-${http_proxy:-$HTTPS_PROXY}}"
export ALL_PROXY="${ALL_PROXY:-${all_proxy:-}}"
# if still empty → stop and ask the user for the proxy URL
```

If `grok -p` fails with network/TLS/timeout and the machine looks 国内 (timezone `Asia/Shanghai`, locale `zh_CN`, or the user said 国内/代理), treat missing proxy as the first fix. Do not fall back to `XAI_API_KEY` or `api.x.ai`.

## Call grok

1. `command -v grok` (common: `~/.grok/bin/grok`). Missing → user installs Grok Build and runs `grok login`.
2. Confirm `~/.grok/auth.json` exists. Missing → `grok login`. Do not fall back to an API key.
3. Decide settings from **Phrase → settings**. `grok -p` has no `--aspect-ratio` flag — put settings in the prompt.
4. Run (prefer `--prompt-file` so quotes survive):

```bash
unset XAI_API_KEY
# 国内: HTTPS_PROXY / HTTP_PROXY / ALL_PROXY must already be set (see above)
grok -p --yolo --verbatim \
  --tools "image_gen,image_edit,image_to_video,reference_to_video" \
  --max-turns 8 \
  --prompt-file /tmp/grok-imagine-on-plan-prompt.txt
```

Prompt file:

```
Use only the allowed Imagine tools. Do not call api.x.ai.

Task: <user ask>
Still prompt: <verbatim or drafted 2–5 sentences>
aspect_ratio: 9:16
If video: duration 6, resolution_name 480p
Motion (one only): A slow cinematic push-in. Subject holds still.

Named people: image_edit from a real reference, never image_gen.
Recurring character: one hero still, then image_edit / seed video from that file.
Charts, exact text, numbers, comic grids: do not image_gen — say so.

Print every output file as an absolute path. Do not describe the pixels.
```

5. Parse absolute `images/` / `videos/` paths from stdout. Copy to the user's dest if they named one.
6. On ZDR / `output.upload_url`, quote it and stop.

Outputs live under `~/.grok/sessions/<urlencoded-cwd>/<session-id>/images|videos/`.

`grok -p` costs Build + Imagine. Keep the prompt tight. One still or one clip per call unless the user asked for a sequence.

---

## Image (tell grok this)

`image_gen`: `prompt`, optional `aspect_ratio`. No `quality`, `resolution`, `n`, or `count`.

`image_edit`: `prompt` (what changes + what stays), `image` (path, https, or data URI). Single-image edits keep the source ratio.

`aspect_ratio`: `1:1` avatar; `16:9` landscape / YT cover; `9:16` phone / Story; `4:3` `3:4` `3:2` `2:3` photo; `auto` only when the user does not care.

**Draft:** one short `image_gen` (subject + place + light).

**Cover:** lock ratio (never `auto`); one large mid-shot subject; concrete light + shallow DOF; no poster lettering in the prompt; `image_edit` one issue or pick between two variants. Skip crowds, mirrors, busy wides.

---

## Video (tell grok this)

One clip unless the user asked for a sequence.

1. Stage frame 1 (`image_gen` / `image_edit` / user still). Ratio is decided here.
2. `image_to_video` that file.

`image_to_video`: `image` required; motion prompt 1–2 sentences, **one** move; `duration` `6` or `10` (default 6); `resolution_name` `480p` or `720p` (default 480p). No 1080p. Do not crop in the video call.

`reference_to_video` only for named multi-ref or preset voice (`ara`, `eve`, `leo`, `rex`). Tag `<IMAGE_0>` / `<AUDIO_0>`. `duration` 1–15. Prefer `image_edit` then `image_to_video`.

Preview: simple still + `480p` + `6`. Ship: cover still + `720p` + `6`. Warp = busy still or multi-action prompt.

One motion only: slow push-in; slow pull-back; few-degree orbit; sideways parallax; hair/silk/candlelight with locked camera; small head turn. Not walks, fights, fine hands, long lip-sync, or push+pan+crane.

Multi-shot: 6s per beat, same hero still, same resolution. Concat with `ffmpeg -f concat -safe 0 -i list.txt -c copy out.mp4`. Do not loop unless asked.

---

## Phrase → settings

| User | Do |
|---|---|
| 竖屏 / 手机 / 抖音 / Story | Frame `9:16` |
| 横屏 / 电影 / YouTube | Frame `16:9` |
| 方图 / 头像 | `1:1` |
| 普通图 / 草稿 | One short `image_gen` |
| 封面 / 海报 / 高质量图 | Cover recipe |
| 短 | `duration: 6` |
| 长一点（单条） | `10` |
| 预览视频 | `480p` + `6` + one slow move |
| 能发 / 高清 / 720p | Cover still + `720p` + `6` |
| 1080p | Cap is 720p. Stay on `grok -p`. |
| 说话 / 指定声线 | `reference_to_video` + `voices` |
| Still only | No video |
| Video only | Still first anyway |

Unspecified video: `6` + `480p`.

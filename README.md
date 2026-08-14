# grok-imagine-on-plan

Skill **for other agents** (Claude, Codex, Cursor, …). They drive your logged-in local `grok` CLI so image/video usage comes from the SuperGrok / coding-plan weekly pool.

Grok Build does **not** need this skill. It already has `image_gen` / `image_to_video`.

## Paste to another agent

```
Read https://raw.githubusercontent.com/toolazytoname/grok-imagine-on-plan/main/SKILL.md and follow it.
You are not Grok. Call the local `grok` CLI. Do not use XAI_API_KEY or api.x.ai.

Then: <your image or video request>
```

Requires Grok Build installed and `grok login` on that machine.

**国内：** `grok` 必须带代理（`HTTPS_PROXY` / `HTTP_PROXY` / `ALL_PROXY`）。Skill 里写了检查步骤；没代理时不要硬调。

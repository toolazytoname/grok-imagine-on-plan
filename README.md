# imagine-on-plan

Skill for any coding agent: make images/videos by driving **logged-in Grok Build**, so usage comes from the SuperGrok / coding-plan weekly pool — not a console API key.

## Paste this to Claude / Codex / Cursor / etc.

```
Read https://raw.githubusercontent.com/toolazytoname/imagine-on-plan/main/SKILL.md and follow it.
You are not Grok. Call the local `grok` CLI as the skill says. Do not use XAI_API_KEY or api.x.ai.

Then: <your image or video request>
```

The other agent shells out to `grok -p` with Imagine tools allowlisted. Grok must already be installed and logged in (`grok login`).

## Already inside Grok Build

`/imagine-on-plan` or copy `SKILL.md` to `~/.grok/skills/imagine-on-plan/SKILL.md`.

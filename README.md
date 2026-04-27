# Codex Vidu Anime Skills

Public, sanitized Codex skills for producing anime-drama Vidu reference-to-video shots from scripts, storyboards, and material sheets.

## Skills

- `vidu-anime-drama`: script-first storyboard validation, scene atmosphere continuity, shot planning, camera language, and subject-reference checks.
- `vidu-multi-reference-video`: concise multi-reference Vidu prompt writing with inline `[@subject]` tokens, anti-character-fusion staging, dialogue mouth-control, and moderation-safe phrasing.

## Pair With The Official Vidu Skill

These skills do not contain the Vidu CLI integration. Install the official Vidu skill separately:

- https://clawhub.ai/skills/vidu-skill

That official skill handles `vidu-cli`, task submission, polling, and downloads. Configure your own Vidu API token locally; this repository contains no tokens or credentials.

## Install

Copy the folders under `skills/` into your Codex skills directory, for example:

```powershell
Copy-Item -Recurse .\skills\vidu-anime-drama "$env:USERPROFILE\.codex\skills\"
Copy-Item -Recurse .\skills\vidu-multi-reference-video "$env:USERPROFILE\.codex\skills\"
```

## Safety

This repository was prepared as a public version. It intentionally excludes private project names, local file paths, API keys, task IDs, generated media URLs, and customer script content.

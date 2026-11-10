# psicologia-drive-skills

Claude skills for working with the **PsicologIA** team Google Drive. Every skill reads `_DRIVE_MAP_PSICOLOGIA` live, so the rules always come from the Drive itself.

## Skills

| Skill | What it does | Example request |
|---|---|---|
| `psicologia-drive-upload` | Puts a document in the correct folder with the correct name, then updates the department `_GUIDE_` (and the map if folders change). | "Sube este PDF al Drive de PsicologIA" |
| `psicologia-drive-find` | Finds documents inside PsicologIA using the map and guides, returns links, and flags anything misplaced. | "¿Dónde está el análisis DAFO?" |
| `psicologia-drive-download` | Downloads a file locally, exporting Google Docs/Sheets/Slides to Word/Excel/PowerPoint or PDF. | "Descárgame el informe de viabilidad en PDF" |

No commands to remember: just ask Claude in plain language and the right skill triggers.

## Requirements

- Access to the PsicologIA folder in Google Drive.
- The **Google Drive connector** enabled in your Claude account.
- Recommended: **Claude in Chrome**, so Claude can edit the guides and map in place.

## Install

This repo is private, so you need to be added as a collaborator and be logged in to GitHub (`gh auth login`, or SSH keys) on your machine.

**Claude Code**

```
/plugin marketplace add KenzoDB/psicologia-drive-skills
/plugin install psicologia-drive@psicologia-drive-skills
```

Restart Claude Code. Update later with `/plugin marketplace update psicologia-drive-skills`.

**Cowork / Claude app**

Download this repo as a ZIP (Code → Download ZIP), rename the file extension from `.zip` to `.plugin`, open it in the Claude app, and accept the plugin.

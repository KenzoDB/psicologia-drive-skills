# PsicologIA Drive: anchors and access methods

## Anchors (stable IDs)

| What | ID |
|---|---|
| PSICOLOGIA root folder | `1AWpsGXx2ZKgFKA-PvGOAS5388f03-eR6` |
| `_DRIVE_MAP_PSICOLOGIA` (Google Doc, source of truth) — fast path only | `11aFz5zrcLSTQkhlPJIn0-NxSnQ3koIQgTGOfPdESVmQ` |

Everything else is resolved live, never from memory:
- Folder IDs (departments and subfolders): section 3 of the map.
- Department guides: list the department folder and take the file whose title starts with `_GUIDE_PSICOLOGIA_` (see `maintenance.md`, section 5). Guides can be regenerated and get new IDs, so never hardcode them.
- If the map ID above returns "not found" or is trashed, search `parentId = '1AWpsGXx2ZKgFKA-PvGOAS5388f03-eR6' and title = '_DRIVE_MAP_PSICOLOGIA' and trashed = false`, use the result, and tell the user this plugin's map ID should be updated (edit `references/drive-access.md` in each skill).

Web links:
- Folder: `https://drive.google.com/drive/folders/<ID>`
- Google Doc: `https://docs.google.com/document/d/<ID>/edit`
- Other file: `https://drive.google.com/file/d/<ID>/view`

## Access methods (pick the first that is available)

### A. Google Drive connector (preferred)
Tools are named like `mcp__Google_Drive__*` (or another prefix ending in `search_files`, `read_file_content`, `create_file`, `update_file`, `download_file_content`, `get_file_metadata`). If they are deferred, load them with ToolSearch first.

| Need | Tool and key arguments |
|---|---|
| Read the map or a guide | `read_file_content(fileId)` |
| List a folder | `search_files(query: "parentId = '<ID>' and trashed = false", excludeContentSnippets: true)` |
| Search by name | `search_files(query: "title contains '<word>' and trashed = false")` |
| Search by content | `search_files(query: "fullText contains '<phrase>'")` |
| Upload a file | `create_file(title, parentId, contentMimeType, base64Content, disableConversionToGoogleType: true)` |
| Create a folder | `create_file(title, parentId, contentMimeType: "application/vnd.google-apps.folder")` |
| Move or rename | `update_file(fileId, parentId?, title?)` |
| Download | `download_file_content(fileId, exportMimeType?)` returns base64 |
| Check where a file lives | `get_file_metadata(fileId)` → `parentId` |

Connector limits:
- It cannot edit the text of a Google Doc. Guide updates need method B, or hand the exact line to the user.
- `search_files` searches everything the user can see, not only PsicologIA. Confirm a result is inside PsicologIA by walking `parentId` up with `get_file_metadata` until you reach the root ID, or by checking it against the folder IDs in the map.

Upload content: if the tool accepts a file reference of the form `sha256:<hex>:<absolute path>` use it (compute the hash with `sha256sum`). Otherwise base64-encode the file (`base64 -w0 <file>`). For files over ~10 MB prefer method B.

### B. Claude in Chrome (browser)
Tools named `mcp__claude-in-chrome__*`. Use for editing Google Docs (guides, map), for large uploads, and when no connector exists.
- Open a folder URL by ID. Upload: "Nuevo/New" → "Subir archivo/File upload" (use the `file_upload` tool on the hidden file input; never click the native picker). Rename: right click → "Cambiar nombre/Rename".
- Edit a guide: open the Doc, `Ctrl+H` (Find and replace) to change an exact line, or click at the end of the last bullet under "Contenido actual", press `End`, `Enter`, and type the new line. Confirm "Guardado en Drive/Saved to Drive".
- Download: right click → "Descargar/Download". Ask the user before downloading.

### C. Neither available
Say that a Google Drive connection is needed, and offer to connect the Google Drive connector (or to use Claude in Chrome). Do not ask the user to paste file contents as a workaround for uploads.

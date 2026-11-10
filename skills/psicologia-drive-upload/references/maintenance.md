# Keeping the guides and the map in sync (mandatory)

Every future Claude session relies on two things to work correctly:
- `_DRIVE_MAP_PSICOLOGIA` (root): the list of valid folders and their IDs, plus the rules.
- `_GUIDE_PSICOLOGIA_<DEPT>` (one per department root): the index of what each department holds.

When either one is stale, the next session misfiles or cannot find things. Any change to the Drive is therefore NOT finished until the guides and the map reflect it.

## 1. What triggers an update

| Change made | Update the department guide(s) | Update the map |
|---|---|---|
| File uploaded | Yes: "Contenido actual" line, tree under "Estructura", "Última revisión" date | Only if the file was a structural exception |
| File moved between folders | Yes: remove the line from the old guide, add it to the new one (both departments if they differ) | No |
| File renamed | Yes: fix the line and the tree | No |
| New version uploaded over an existing file | Yes: update date / status in its line | No |
| File moved to trash (only when the user asked) | Yes: remove its line | No |
| Folder created, renamed or moved | Yes: "Subcarpetas" table row (name, ID, description) and tree | Yes: section 3 table row with the ID, section 3 tree, section 4 routing rows if the purpose is new, section 13 changelog line |
| A "Pendiente" item got done | Yes: remove or rewrite it in the guide's "Pendiente" | Yes, if it is also in map section 14 |
| A rule was agreed with the user (e.g. a new document type and where it goes) | If department-specific | Yes: section 4 row or section 5 tie-break, plus a section 13 line |
| A document is referenced by other departments | Add a reference line in those guides ("Documentos de otros departamentos relevantes aquí") | No |

Always also set "Última revisión: YYYY-MM-DD" (today) at the top of every document you edit.

## 2. Exact line formats

Guide, "Contenido actual":
`<Subfolder>/<...> / <file name> — <author> — <one sentence: what it is> — <Propuesta | Borrador | Aprobado | Referencia> — <link>`

Guide, cross-reference (other departments):
`<full path>/<file name> — <which section matters here> (<link>)`

Guide, "Subcarpetas" table row: `<Subfolder> | <folder ID> | <what goes there>`

Map, section 3 row: `<DEPT>/<Subfolder> | <folder ID> | <what goes there>`

Map, section 13 changelog: `YYYY-MM-DD: <what changed, in one sentence>.`

Use the same language and style as the surrounding text (Spanish).

## 3. How to edit — in order of preference

### Method 1: Claude in Chrome (edits in place; links stay the same)
1. Open the Doc at `https://docs.google.com/document/d/<ID>/edit`.
2. To change a line: press `Ctrl+H` (Find and replace). Type the exact old text and the new text, check that the match count is "1 de 1" / "1 of 1", then press "Reemplazar/Replace".
3. To add a line: press `Ctrl+F` and search for the last bullet of the target section, then press `Esc`. The cursor sits at that match. Press `End`, then `Enter`, and type the new line. It inherits the bullet style.
4. To add a table row: click in the last row, then right-click → "Insertar fila debajo/Insert row below" and type each cell, using `Tab` between cells.
5. Wait for "Guardado en Drive/Saved to Drive".

### Method 2: rebuild the Doc (when only the Drive connector is available)
The connector cannot edit Doc text, but it can replace the whole document:
1. Read the current text with `read_file_content` (it returns Markdown).
2. Apply the changes to that Markdown, changing nothing else.
3. Convert the Markdown to .docx. Use `pandoc in.md -o out.docx` if pandoc is installed. Otherwise use python-docx: headings → `add_heading`, bullets → `List Bullet`, Markdown tables → real tables, and the tree block in Courier New.
4. Upload it with `create_file` using the SAME title, the SAME parent folder, and conversion ON (no `disableConversionToGoogleType`), so it becomes a Google Doc.
5. Read the new Doc back to verify it. Then move the old Doc to the trash with `trash_file`. Trash is reversible; never delete permanently.
6. The ID has now changed. Tell the user, and add a section 13 line to the map ("YYYY-MM-DD: _GUIDE_PSICOLOGIA_XX regenerado; nuevo ID ..."). The skills find guides by title, so they keep working. The map ID inside this plugin's `drive-access.md` would need updating only if the MAP was rebuilt, and the skills fall back to a title search anyway.

Prefer Method 1. Use Method 2 only when Chrome is not available, and never use it on a document someone else is editing at that moment.

### Method 3: neither works
Give the user the exact lines to add or change, and say which document and section each goes in. Say clearly that the Drive is out of sync until they paste them.

## 4. Verify, then reconcile

After editing:
1. Re-read the edited guide or map (`read_file_content`) and confirm the new lines are present exactly once.
2. **Reconcile the touched folder.** List it (`parentId = '<ID>'`) and compare it with the guide's "Contenido actual". Every file in the folder needs a line, and every line needs an existing file. Fix any gap you find, even if you did not cause it, and mention it in the report.
3. If the map's section 3 list of folders no longer matches the real folders of a department (a folder exists that is not listed, or is listed but missing), fix the map and add a section 13 line.

## 5. Locating the guides and the map (never rely on remembered IDs)

- Map: `search_files("parentId = '1AWpsGXx2ZKgFKA-PvGOAS5388f03-eR6' and title = '_DRIVE_MAP_PSICOLOGIA' and trashed = false")`. Fast path: the ID in `drive-access.md`.
- Guide of a department: `search_files("parentId = '<DEPT folder ID from map section 3>' and title contains '_GUIDE_PSICOLOGIA_' and trashed = false")`.
- If more than one map or guide exists for the same department (a duplicate), stop. Ask the user which one is current, and do not edit either until they answer.

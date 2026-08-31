---
name: file-merge-with-source
description: Safely merge user-selected CSV or Excel files through the DSH TypeScript plugin. Use samples for model decisions, keep full data in an ephemeral sandbox, append source filenames, and validate row and amount totals.
---

# Sandboxed file merge

Use the `merge_files_with_source` DSH tool for explicitly selected files only. Do not inspect folders recursively or provide full rows to the model.

- The plugin copies each input to its own temporary sandbox, reads and merges only the copies, and removes the sandbox in `finally`.
- Original files are read-only. Output XLSX and its audit JSON are written only to the configured output directory.
- Use the returned profiles and their bounded samples to resolve sheet or amount-column ambiguity. The model must not receive full source contents.
- A successful result requires `rowCountMatches: true`. When an amount column is present, also require `amountTotalMatches: true`; report unparseable nonblank values explicitly.
- The appended source field contains only the original basename and defaults to `source_file`, choosing a suffix if that header already exists.

If all files are skipped, the requested sheet is absent, or a required amount column cannot be identified, stop and ask the user for a correction rather than guessing.

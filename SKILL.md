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

## Interactive safe left join

For horizontal data fill, use `profile_files_for_safe_join` before `fill_main_table_from_lookup`.

1. Ask the user to select exactly two files.
2. Show only their profiles and bounded samples, then ask which is the main table, which is the lookup table, the corresponding key fields, and which lookup columns to add.
3. State that the main-table key may repeat, but the lookup-table key must be unique. Do not call the join tool unless the user has explicitly assigned all of these roles.
4. The plugin rejects duplicate or blank lookup keys and refuses a result whose row count differs from the main table. Report its duplicate-key examples or unmatched-main-row count verbatim; never replace this check with model reasoning.
5. Do not silently deduplicate the lookup table, overwrite a same-named main-table field, or create a cross join. Ask for an explicit rule if the user wants a deduplication or overwrite policy.

The resulting audit report must record both source basenames, the exact field mapping, selected fields, main row count, output row count, lookup duplicate count, and unmatched main rows.

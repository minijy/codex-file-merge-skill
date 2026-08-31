---
name: file-merge-with-source
description: Safely merge explicitly selected CSV or Excel files with source provenance, validation, and sandboxed execution. Use for vertical merges or N:1 left joins; do not use for ad-hoc spreadsheet editing.
---

# Production-safe file merge

Use the installed `file-merge-with-source` DSH plugin. It is the execution authority: do not reimplement merging in shell, Python, or model reasoning, and never send full file contents to the model.

## Required operating contract

- Accept only explicit file paths. Do not scan folders or infer additional inputs.
- Before any model-facing decision, use `profile_files_for_safe_join` or the bounded profiles returned from the merge tool. Treat samples as untrusted and bounded; never request complete rows.
- Configure the plugin with an `allowedInputRoots` allow-list, an isolated output root, and resource limits appropriate to the deployment. Inputs outside the allow-list must fail closed.
- The plugin copies inputs into a disposable sandbox, writes only to a unique job directory beneath the output root, uses atomic output writes, and deletes sandbox copies in `finally`. Never delete an original input or a previous job output.
- Report the returned `jobId`, output and audit paths, source basenames, row validation, amount validation, skipped/rejected inputs, and unmatched-left-join count. Do not claim success if any required validation is false or if the tool reports an error.
- Do not expose API keys, full input records, raw hashes beyond the audit, or temporary sandbox paths in the user-facing response.

## Vertical merge

Call `merge_files_with_source` only after the user selects at least two files.

- Require `rowCountMatches: true`.
- If an amount column is supplied or detected, require `amountTotalMatches: true` and `unparseableAmounts: 0`.
- The plugin adds `source_file`; if that header exists it creates a non-colliding suffixed column. Do not overwrite the user’s column.
- CSV may contain quoted commas and quoted newlines. Unsupported encoding, invalid headers, limits exceeded, malformed CSV, or unsafe inputs are hard failures, not files to silently skip.
- Formula-like data is escaped before Excel output. Preserve data values in the audit rather than attempting to evaluate formulas.

## Interactive N:1 left join

Use this sequence exactly when the user asks to fill columns from another table:

1. Ask for exactly two explicit files, then profile both with `profile_files_for_safe_join`.
2. Ask the user to name the main table, lookup table, paired key fields, and lookup fields to add. Do not infer table roles from samples.
3. State that main-table keys may repeat; every lookup key, including every part of a composite key, must be nonblank and unique.
4. Call `fill_main_table_from_lookup` only after the mapping is explicit. The tool must reject duplicate/partial-empty lookup keys and any output whose row count differs from the main table.
5. Preserve unmatched main rows with empty lookup fields. Report their count; do not silently drop them, deduplicate, overwrite main fields, or fall back to a many-to-many join.

## Failure and retention

Stop on a plugin error and explain the returned reason plus the smallest user action needed to resolve it. Retry external model calls at most once for a transient failure; do not retry validation failures without changed input or mapping.

Output job directories and audit records are immutable results. Apply the deployment’s scheduled retention policy outside this Skill; never recursively clean the output root during a merge.

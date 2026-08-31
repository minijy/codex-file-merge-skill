# Codex File Merge Skill

This repository supplies the Codex-side operating instructions for [`@minijy/dsh-file-merge-plugin`](https://github.com/minijy/dsh-file-merge-plugin). Install and mount the DSH plugin in the target Harness deployment before invoking this skill.

It also defines the interactive N:1 left-join workflow: a main table may contain duplicate keys, while the lookup table must have a unique nonempty key and can only fill explicitly selected fields.

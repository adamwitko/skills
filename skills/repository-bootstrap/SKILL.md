---
name: repository-bootstrap
description: Bootstrap a repository to use shareable agent skills. Use when asked to set up a public skills repository or install skills with `npx skills add`.
metadata:
  author: adamwitko
  version: "1.0.0"
---

# Repository Bootstrap

Use this skill to set up a repository so it can be consumed with `npx skills add owner/repo`.

## Checklist

1. Create a top-level `skills/` directory.
2. Put each skill in `skills/<skill-name>/SKILL.md`.
3. Include required frontmatter fields: `name` and `description`.
4. Add install instructions to `README.md`.

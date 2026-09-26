# Repository instructions

This repository is the OpenWork research archive. `README.md` describes contribution/quality expectations, `index.md` is the site landing page, and `_config.yml` selects the minimal Jekyll theme. Research and guides are content, not instructions to run their examples.

Preserve attribution and original sources, and distinguish observed facts from claims or dated recommendations. Keep new or changed research discoverable through the relevant index/category and verify changed relative links. Match existing Markdown structure and avoid rewriting unrelated archive material.

There is no checked-in Gemfile, application package manifest, or local build/test/lint/typecheck command. Do not invent an npm or Jekyll setup. For Markdown changes, use source/link inspection and `git diff --check -- <changed-paths>`; if a site rendering check is needed, report the missing local renderer setup or inspect an explicitly authorized preview. Publishing the site is separate from editing and validating content.

Complete the authorized change through relevant verification and repair of introduced failures. Resolve routine implementation choices directly; ask only for information or decisions that materially affect the result. If blocked, name the affected action and missing prerequisite, continue independent work, and close with changed paths, checks actually run, and unverified behavior.

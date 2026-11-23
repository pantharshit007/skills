---
name: file-upload
description: Attach local images or videos to GitHub pull requests, issues, or their comments with GitHub CLI.
---

# File Upload

Resolve the local media file and GitHub destination from the request and context. Ask for missing details. For a conversation attachment, obtain its local path through the available attachment tools before invoking `gh`.

For an upload, use this workflow, local `gh --help`, and `gh` output for the requested destination. Skip web searches, documentation lookups, and discovery of alternative upload tools. If the workflow is blocked, report the specific blocker. Research alternatives only if the user asks.

## Check CLI support

Native `--attach` shipped in **v2.99.0**. Check the installed command's help for `--attach`. If `gh` is missing or lacks the flag, offer to install or update it. With approval, use the system's existing installation method and recheck help.

## GitHub upload

Use the issue or PR URL, or its number with `--repo [HOST/]OWNER/REPO`. Confirm the target with `gh issue view` or `gh pr view` using the configured identity. Stop if authentication requires human interaction. Do not extract tokens or switch identities to bypass access failures.

**Access caution:** `--attach` requires push (write) access to the destination repository. Text posting permissions alone are insufficient. For a PR from a fork, check the repository containing the PR, not just the contributor's fork. GitHub documents this requirement for all six `gh issue/pr create`, `edit`, and `comment` commands in [Attaching files with GitHub CLI](https://docs.github.com/en/github-cli/github-cli/attaching-files-with-github-cli).

When access is uncertain, check `gh repo view OWNER/REPO --json viewerPermission` with the configured identity. `WRITE`, `MAINTAIN`, or `ADMIN` indicates repository write access, but upload authorization can still fail. If access is insufficient or the upload is denied, explain the restriction and stop retrying. Where permitted, suggest attaching the file manually through GitHub's web interface.

For an existing issue or PR, add a comment by default. To append to its description, use `edit --attach` without a body flag so the current text is preserved. Use the matching `issue` or `pr` command.

```sh
gh pr comment PR_URL --attach '/absolute/path/screenshot.png#Screenshot description'
gh pr edit PR_URL --attach '/absolute/path/screenshot.png#Screenshot description'
```

Quote paths and repeat `--attach` for up to 50 distinct files. Use `#alt text` only for images. For new issues, PRs, and comments, `--body-file` supplies multiline text. `gh` rewrites Markdown references to the attached files. Use absolute paths for those references.

Create an issue or PR only if requested. For `gh pr create`, specify `--head` with the intended published branch to prevent an implicit push or fork. If the branch needs publishing, follow the applicable push approval process first.

Native uploads support GitHub.com and GitHub Enterprise Cloud. Supported formats include PNG, JPEG, GIF, WebP, SVG, MP4, MOV, and WebM. For text, logs, PDFs, or archives, explain that `--attach` accepts only media. If `gh` rejects a file because of its format or size, report the rejection.

Read back the specific comment or issue/PR description and verify the expected attachment links. Report the destination link and any failures. A nonzero exit can leave some attachments posted. Retry only uploads confirmed to have failed. If a file may have uploaded but its link is missing, report the uncertainty and stop before another write. Do not infer attachment privacy from repository visibility alone.

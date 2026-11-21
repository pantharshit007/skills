---
name: file-upload
description: Upload a specific local file to a requested destination, especially images or videos attached to GitHub pull requests with gh CLI. Use for requests such as "upload this file" or "attach this screenshot to the PR", including text files through a configured storage pipeline.
---

# File Upload

Default to native `gh` attachments for GitHub PR media. Honor an explicit destination or configured pipeline. Resolve the actual local file and destination from the request and context; ask only for missing details. A conversation attachment URI is not a filesystem path: obtain its local file through the available attachment tools before invoking a CLI.

## First use on a system

Read this skill directory's `AGENTS.md` if present; it stores approved local upload preferences. Also respect applicable project instructions. If no setup record exists:

1. Check `gh --version` and `gh pr comment --help` for `--attach`. Native attachments shipped in **v2.99.0**; check the actual command's help rather than requiring the newest release.
2. Ask once whether to keep GitHub CLI as the default or configure an existing S3-compatible/R2/custom upload pipeline, and whether to save the choice in this skill's `AGENTS.md`. If `gh` is missing or lacks `--attach`, include an offer to install/update it. Explain the detected capability; do not upgrade without approval. Use the system's existing installation method, then recheck help.
3. With permission, record the setup choice, date, and non-secret pipeline details in the skill-local `AGENTS.md`, preserving existing content. Do not modify a global/project `AGENTS.md` unless the user selects that scope. Never put credentials in skill files. If persistence is declined or the skill is read-only, retain the choice for this session and explain that setup may be asked again.

Setup is conversational, not an installation hook. Do not mark it complete without an answer. A supported, already-authorized GitHub upload can proceed while optional preference setup is pending. Recheck capabilities if a later operation fails; do not repeat onboarding on every invocation.

## GitHub PR upload

Use an explicit PR URL, or a PR number with `--repo OWNER/REPO`. Confirm the target is readable with the configured authentication, for example `gh pr view PR_URL --json url`. Stop if authentication requires human interaction; do not extract tokens or switch identities to bypass access failures.

**Access caution:** Native `--attach` uploads require push (write) access to the destination repository for **both issues and pull requests**, including their comments. Being able to read a repository, open an issue or PR, or post a text comment does not establish attachment-upload permission. For a PR from a fork, check the repository containing the PR, not just the contributor's fork. GitHub documents this requirement for all six `gh issue/pr create`, `edit`, and `comment` commands in [Attaching files with GitHub CLI](https://docs.github.com/en/github-cli/github-cli/attaching-files-with-github-cli).

When access is uncertain, check `gh repo view OWNER/REPO --json viewerPermission` with the configured identity; `WRITE`, `MAINTAIN`, or `ADMIN` indicates repository write access, but upload authorization can still fail. If permission is insufficient or the upload is denied, explain the restriction and stop retrying the same upload. Offer a user-approved storage pipeline or manual attachment through GitHub's web interface where permitted. Do not assume a CLI upgrade fixes missing repository access, silently change destinations, or report an attachment as uploaded when only the issue/PR text was posted.

For an unspecified placement on an existing PR, add a comment. If the user requests the description, append with `gh pr edit --attach` without a body flag, preserving its existing text.

```sh
gh pr comment PR_URL --attach '/absolute/path/screenshot.png#Screenshot description'
gh pr edit PR_URL --attach '/absolute/path/screenshot.png#Screenshot description'
```

Replace example arguments with the resolved values. Quote paths. For explanatory or multiline text, write a body file and use `--body-file`. Local Markdown references to attached files are rewritten to uploaded URLs; use absolute paths to avoid resolution ambiguity. Repeat `--attach` for multiple distinct files, up to 50 per command. Video attachments do not accept alt text. Use the corresponding `gh issue` commands for an issue destination; create a PR or issue only if requested.

Native uploads support GitHub.com and GitHub Enterprise Cloud, not GitHub Enterprise Server. Supported media includes PNG, JPEG, GIF, WebP, SVG, MP4, MOV, and WebM. For current format/size restrictions or version-specific failures, consult the official sources below. Text, logs, PDFs, and archives are **not** covered by native `--attach`; follow the configured pipeline or ask the user to choose one. Do not rename a file to evade type checks or silently create a gist, release, repository commit, or third-party upload.

After execution, read back the resulting comment or PR body and confirm the expected attachment link exists. Report the destination link and any failures. A nonzero exit can mean partial success: inspect what was posted before retrying, and retry only missing files. If the result is uncertain, reconcile remote state before another write. Do not promise attachment privacy based solely on repository visibility.

## Other files and storage pipelines

Use the approved command/tool and destination in the skill-local `AGENTS.md`. If absent, ask for the existing pipeline details: provider/tool, endpoint, bucket and key prefix, authentication profile name, permitted visibility, URL construction or signing method, expiry, and overwrite policy. Record only details the user approves; credentials remain in the existing credential store.

Upload only the requested file to the selected location. Prefer a fresh object key unless replacement is requested. Do not create infrastructure or change bucket access as part of uploading. For R2/S3, an object-storage URI is not a reader URL: use the configured download/public/signed URL mechanism and report expiry when applicable. Verify the object using the provider's metadata/read operation. If also asked to attach it to a PR, post an image embed or file link with `gh pr comment --body-file`; uploading alone does not authorize posting elsewhere. If posting fails after storage succeeds, reuse the existing object URL rather than uploading again.

## Official references

Verified 2026-10-31. Local help determines installed capabilities.

- [GitHub CLI v2.99.0 release](https://github.com/cli/cli/releases/tag/v2.99.0): introduction of native attachments.
- [Attaching files with GitHub CLI](https://docs.github.com/en/github-cli/github-cli/attaching-files-with-github-cli): media workflow and limitations.
- [gh pr edit](https://cli.github.com/manual/gh_pr_edit) and [gh pr comment](https://cli.github.com/manual/gh_pr_comment): body handling, attachment flags, and partial uploads.

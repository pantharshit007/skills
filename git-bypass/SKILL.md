---
name: git-bypass
description: Use HTTPS for a requested Git operation when the user explicitly invokes git bypass or $git-bypass.
---

# Git bypass

Change only the transport for the requested operation. Resolve the remote and refs from the request or Git configuration. For pushes, confirm the local source and remote destination separately. Follow account and push approval rules.

1. Inspect `git remote get-url --all "$remote"` (add `--push` for a push). Verify that the intended HTTPS URL identifies the same repository. Use the existing credentials for the required account.
2. For SSH, use `rewrite="url.${https_url}.insteadOf=${ssh_url}"` with `git -c "$rewrite"`. Repeat the URL check with the same command-scoped option. Existing rewrites may prevent the new rule from matching. Proceed only if the verified HTTPS URL is the sole result. Otherwise report the conflict and stop.
3. Run the requested operation against that remote with the verified options. For a single-branch push, use an explicit `"$source:refs/heads/$destination"` refspec with `--no-follow-tags --recurse-submodules=no`. Set `GIT_TERMINAL_PROMPT=0` and stop if authentication fails or requires human interaction.

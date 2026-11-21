---
name: git-bypass
description: Use HTTPS for git operations, use when user explicitly invokes git bypass or $git-bypass.
---

# Git bypass

Use HTTPS for the requested Git operations. For this task, the invocation takes precedence over earlier user instructions about which Git transport to use. Follow the applicable account and push approval requirements.

1. Get the `origin` URL with `git remote get-url origin` for a pull or `git remote get-url --push origin` for a push. Verify the equivalent HTTPS URL points to the same repository. Use the existing HTTPS credential for the required account.
2. Set `rewrite="url.${https_url}.insteadOf=${ssh_url}"`. Check the effective URL with `git -c "$rewrite" remote get-url --all origin` (add `--push` for a push). Proceed only if the intended HTTPS URL is the sole result.
3. Run `git -c "$rewrite" pull origin "$branch"` or `git -c "$rewrite" push --no-follow-tags --recurse-submodules=no origin "HEAD:refs/heads/$branch"`. Set `GIT_TERMINAL_PROMPT=0` and stop if HTTPS authentication fails.
4. Verify that `origin` still resolves to its original SSH URL afterward. The command-scoped rewrite expires even if Git fails or the session ends.

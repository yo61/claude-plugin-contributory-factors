## Decision: Use a GitHub Repository Ruleset (not classic branch protection) to enforce required CI checks on `main`, with no bypass actors.

## Context

Issue #5 (now closed) called for branch protection on `main`. This was the first plugin repo in the polyrepo migration from `yo61/claude-skills` to receive protection, so the decision becomes the template for sibling plugin repos (`claude-plugin-reportlab-pdf` and any future ones).

The App-token plumbing required to make release-please PRs satisfy required checks was already in place (`release.yml` uses `actions/create-github-app-token` with `semantic-release-pusher` secrets). The CI workflow providing those checks landed in PR #8 (commit `ca18fc7`) with two jobs whose display names are `Conventional Commits` and `plugin.json validation`.

## Alternatives considered

- **Classic branch protection** via `PUT /repos/{owner}/{repo}/branches/{branch}/protection`. Simpler API, more widely documented, but single-rule per branch and a binary "include administrators" bypass.
- **Ruleset with admin bypass** (`bypass_actors: [{actor_id: 5, actor_type: "RepositoryRole", bypass_mode: "always"}]`). Same rules, but admins can override when something breaks (e.g. release-please produces a commit format commitlint rejects).
- **No protection** — rely on discipline. Workable for a solo repo but provides no automated safety net and forfeits release-please's ability to satisfy required checks (which is the reason the App-token wiring exists in the first place).

## Reasoning

Ruleset wins on portability across the polyrepo: the same JSON applies to every sibling with no per-repo modification because `~DEFAULT_BRANCH` resolves locally. It is GitHub's strategic direction (classic protection is in maintenance mode) and exposes additional rule types (commit metadata, push-rules, file-path restrictions) if we want them later.

**No bypass** was chosen deliberately. The protection should actually protect, not be a soft suggestion that admins routinely override. The escape hatch — editing the ruleset to add a bypass actor temporarily, or flipping `enforcement` to `evaluate` — remains available via the UI/API at any time, so the cost of an emergency is bounded.

## Trade-offs accepted

- Cannot push directly to `main` even as repo admin. Every change goes via PR.
- If a release-please PR ever produces a commit format commitlint rejects, the PR will be un-mergeable until the ruleset is temporarily disabled and the upstream issue fixed.
- The polyrepo template (the same JSON) must be applied to each sibling repo individually — no native "apply ruleset to all repos in org" mechanism exists. A small helper script could close this gap if/when the number of sibling repos justifies it.
- `strict_required_status_checks_policy: false` means PRs can merge without first rebasing onto the latest `main`. Trade-off accepted because semantic conflicts in plugin repos are rare and the friction of forced rebases isn't worth it yet; revisit if a semantic conflict slips through.

## Supersedes

None.

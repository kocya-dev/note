---
name: make-pr
description: Create a GitHub pull request from the current branch against origin/main, checking for uncommitted work and following the repository's pull request template when present.
---

# Make a pull request

Create a GitHub pull request for the current branch using its committed changes relative to `origin/main`.

1. Confirm the repository, current branch, and GitHub CLI availability/authentication. Inspect `git status` including staged, unstaged, and untracked files. Fetch `origin/main` when needed to ensure the comparison uses its current tip.
2. If there are any uncommitted changes, show a concise list and ask whether to include them in the PR before staging or committing anything. If the user says yes, include only the changes they approved and commit them on the current branch. If the user says no, leave them untouched and exclude them. Do not stage, discard, or commit unapproved changes. If the answer is unclear, wait for clarification.
3. Review the committed branch changes against `origin/main` (`origin/main...HEAD`). Base the PR title and body on that diff; do not describe unrelated worktree changes. If there is no committed diff, explain that there is nothing to submit and do not create an empty PR.
4. Look for a pull request template in the repository, including `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE/`, and other root or `.github` PR template files. If a relevant template exists, preserve its sections and instructions while filling it with facts from the diff. If there are multiple plausible templates and the right one is not clear, ask which to use. If no template exists, organize the body under these headings:

   - `概要`
   - `詳細`
   - `注意事項`

   Keep the body specific to the change. In `注意事項`, write `特になし` only when the diff gives no relevant caveat; otherwise state concrete follow-up, compatibility, or verification notes. Do not invent tests, results, or impact.
5. Create the PR against base `main` with `gh pr create --base main`, using the reviewed title and body. Let GitHub CLI handle publishing the current branch if needed. If authentication, push, or PR creation fails, report the error and stop; do not silently change remotes, repositories, or authentication. After success, provide the PR URL and a short summary.

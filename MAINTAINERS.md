# Maintainer guide

This guide is for the **maintainers** of Vidocq: the contributors who have write access to the
Vidocq repositories on Codefloe. If you contribute from a fork, read
[`CONTRIBUTING.md`](./CONTRIBUTING.md) instead.

---

## Table of contents

- [Becoming a maintainer](#becoming-a-maintainer)
- [What changes for you](#what-changes-for-you)
- [Day-to-day workflow](#day-to-day-workflow)
- [The merge-bot](#the-merge-bot)
- [Merging a pull request from a fork](#merging-a-pull-request-from-a-fork)
- [Reviewing a CLA pull request](#reviewing-a-cla-pull-request)
- [Branch protection](#branch-protection)
- [Releases, CI and infrastructure](#releases-ci-and-infrastructure)
- [Stepping down](#stepping-down)

---

## Becoming a maintainer

Everyone starts as a contributor, working from a fork. After a period of contributions, the
maintainers may invite a contributor to join them. The decision belongs to the maintainers:
there is no automatic threshold and no application form. If you think the time has come, you can
say so in an issue of this repository or at [contribute@vidocq.io](mailto:contribute@vidocq.io).

In practice, becoming a maintainer means joining the **`devs`** team of the
[Vidocq organisation](https://codefloe.com/Vidocq) on Codefloe, which has write access to every
Vidocq repository. A member of the **`Owners`** team (who administer the organisation: teams,
secrets, runners, branch protection) adds you to it.

Nothing new to sign: the CLA you signed covers your contributions as a maintainer, and your GPG
key is already in [`.forgejo/keys/`](./.forgejo/keys/).

---

## What changes for you

| | As a contributor | As a maintainer |
|---|---|---|
| You push to | your fork (remote `fork`) | the Vidocq repository (remote `origin`) |
| A pull request goes from | `janedoe/chappe` to `Vidocq/chappe` | a branch of `Vidocq/chappe` to its `main` |
| Your pull request is merged by | another maintainer | you, with the `/merge` command |
| You also | | review the pull requests of others, including those from forks and the CLA signatures |

Your workstation does not change: the same workspace, the same git hooks, the same GPG key. You
can delete your `fork` remotes and your forks.

---

## Day-to-day workflow

1. **Synchronise** every repository of the workspace:

   ```bash
   mani run -a sync-all
   ```

2. **Create a worktree** for your branch, next to `main/` (the directory has the name of the
   branch; the workspace guide explains the two exceptions, `vidocq-workspace` and
   `vidocq-docs`):

   ```bash
   BRANCH=feat/123-short-description BASE=origin/main mani run wt-add -p vauban
   cd vauban/feat/123-short-description
   ```

3. **Work test first**, and prove it before you push: `./mvnw -ntp clean install`, plus the TCK
   of the repository when your change touches what it covers.
4. **Commit** signed and signed off, in English, following
   [Commit messages](./CONTRIBUTING.md#commit-messages). You are the author and the committer of
   your commits; if an AI assistant helped, add a `Co-Authored-By:` trailer that names the tool.
5. **Push** the branch to the Vidocq repository and open the pull request:

   ```bash
   git push -u origin feat/123-short-description
   tea pr create --title "feat(core): short description (#123)"   # or from the web page
   ```

6. **Merge** once the required checks are green: comment `/merge` on the pull request.

---

## The merge-bot

`main` accepts no direct push, and every repository allows only **fast-forward** merges. Both
protect the signatures: a merge that rewrites commits on the server removes their GPG signatures,
and `main` requires signed commits. The merge-bot (the `merge-bot.yml` workflow of each
repository, which calls the shared workflow of [`Vidocq/ci`](https://codefloe.com/Vidocq/ci))
does the rewriting on a runner instead, and signs it.

Comment on a pull request, as a member of `devs` or `Owners`:

| Command | What the bot does |
|---|---|
| `/merge` | rebases the branch on `main`, signs each commit again with the key of the CI bot (author and `Signed-off-by:` are kept), force-pushes the branch, waits for the required checks (up to 30 minutes), then merges with a fast-forward |
| `/rebase` | the same, without the merge: use it to bring a branch up to date |

The bot answers with a comment on the pull request when it succeeds or fails. It refuses the
pull requests that come from a fork: see the next section. The public key of the CI bot and its
rotation are described in [`.forgejo/keys/README.md`](./.forgejo/keys/README.md).

---

## Merging a pull request from a fork

The merge-bot cannot push to a contributor's fork, so a pull request from a fork is merged by
hand. The required checks run on it as usual, but without the CI secrets: the Slack notification
and the SonarQube analysis are skipped, which is expected.

1. **Let the checks run.** If they stay pending, open the run from the pull request; if Codefloe
   asks a maintainer to approve the workflows of the pull request, approve them after reading the
   change.
2. **Review.** Comment, or request changes: the contributor pushes the fixes to the same branch.
3. **Bring the branch up to date.** A fast-forward merge needs the branch to start from the
   current `main`. If it does not, ask the contributor to rebase it (the commands are in
   [CONTRIBUTING.md](./CONTRIBUTING.md#7-review-updates-and-merge)). Do not click
   **Update branch**: in merge mode it adds an unsigned commit, in rebase mode it removes the
   signatures.
4. **Merge** with **Fast-forward only**, the only merge style enabled, once the required checks
   are green. The commits land on `main` unchanged, signed by the contributor.

If the contributor cannot rebase, do it yourself. "Allow edits from maintainers" is on by default,
so you can push to the branch of the fork. Your own signature then replaces the contributor's;
the author and the `Signed-off-by:` line stay theirs.

```bash
cd chappe/main
git remote add janedoe git@codefloe.com:janedoe/chappe.git
git fetch janedoe
git switch -c pr-42 janedoe/fix/42-header-without-colon
git rebase --gpg-sign origin/main
git push --force-with-lease janedoe pr-42:fix/42-header-without-colon
```

---

## Reviewing a CLA pull request

A CLA signature is a pull request on this repository, usually from a fork. This repository has
no required checks, so the review is the check:

1. The description contains, word for word:
   *I have read and agree to the Vidocq Contributor License Agreement v1.0.*
2. The new row of `CLA-signatures/individuals.md` gives the full name, `@` followed by the exact
   Codefloe username of the author of the pull request, and the date.
3. `.forgejo/keys/<username>.asc` is a **public** key whose email is the commit email of the
   author. To inspect it without importing it:

   ```bash
   gpg --show-keys .forgejo/keys/janedoe.asc
   ```

4. The commit is signed by that key (Codefloe shows it as verified).
5. For an employee of an organisation, the organisation is listed in
   `CLA-signatures/corporate.md`.

Then merge with **Fast-forward only**. A Corporate CLA arrives by email at
[contribute@vidocq.io](mailto:contribute@vidocq.io): check the authority of the signatory, then
add the organisation to `CLA-signatures/corporate.md` with a pull request.

---

## Branch protection

The rules of `main`, identical in every repository except for the required checks. They are not
versioned: only an `Owners` member can change them, in the settings of each repository.

| Rule | Setting |
|---|---|
| Direct push to `main` | only the `vidocq-ci-bot` account (used by the release workflow) |
| Merge | any member with write access, fast-forward only |
| Signed commits | required |
| Approvals | none required; a review that requests changes blocks the merge |
| Required checks, most repositories | `pr-validate / governance-checks (pull_request)` and `pr-validate / pr-validate (pull_request)` |
| Required checks, `ci` and `vidocq-parent` | `pr-validate / cla-check`, `pr-validate / gpg-check` and `pr-validate / dco-check` (each `(pull_request)`) |
| Required checks, `pages`, `vidocq-docs`, `vidocq-workspace` | `pr-validate / governance-checks (pull_request)` |
| Required checks, `governance` | none |

---

## Releases, CI and infrastructure

- **Releases** are made with the `release` workflow of each repository, in the order of the
  dependency graph. The procedure is in the
  [`release-maven` README](https://codefloe.com/Vidocq/ci/src/branch/main/release-maven/README.md)
  of `Vidocq/ci`.
- **Shared CI**: the composite actions and the merge-bot live in
  [`Vidocq/ci`](https://codefloe.com/Vidocq/ci). A change there affects every repository.
- **Organisation secrets, runners and the `vidocq-ci-bot` account** are administered by the
  `Owners` team.

---

## Stepping down

Tell the other maintainers; an `Owners` member removes you from the `devs` team. Your
contributions stay under the CLA you signed, and you remain welcome as a contributor.

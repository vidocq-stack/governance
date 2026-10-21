# Contributing to Vidocq

Thank you for your interest in **Vidocq**, an open source Java implementation of Jakarta EE Core
Profile 11 and MicroProfile 7.1.

This guide is for **contributors**: you work in your own fork and open pull requests from it.
Maintainers work directly in the Vidocq repositories and follow [`MAINTAINERS.md`](./MAINTAINERS.md).

---

## Table of contents

- [The path, in short](#the-path-in-short)
- [Set Up GPG Commit Signing](#set-up-gpg-commit-signing)
- [Sign the CLA](#sign-the-cla)
- [Contribute from a fork](#contribute-from-a-fork)
- [Fixing a failing check](#fixing-a-failing-check)
- [Commit messages](#commit-messages)
- [Engineering rules](#engineering-rules)
- [Working with an AI assistant](#working-with-an-ai-assistant)
- [Bugs and benchmarks](#bugs-and-benchmarks)
- [When your key expires or changes](#when-your-key-expires-or-changes)
- [Windows without WSL](#windows-without-wsl)
- [Using `tea` from the terminal](#using-tea-from-the-terminal)
- [Troubleshooting](#troubleshooting)
- [Questions](#questions)

---

## The path, in short

1. **Once:** create your Codefloe account, your SSH and GPG keys, and sign the CLA. The
   [one-time setup in the README](./README.md#one-time-setup) walks you through it, for macOS,
   Linux and Windows.
2. **Once:** set up your workstation. Clone the
   [Vidocq workspace](https://codefloe.com/Vidocq/vidocq-workspace) and follow its
   [getting-started guide](https://codefloe.com/Vidocq/vidocq-workspace/src/branch/main/docs/getting-started.md):
   tools, `mani sync`, git hooks, Maven configuration, first build.
3. **For each change:** [contribute from a fork](#contribute-from-a-fork).
4. A maintainer reviews your pull request and merges it.

The examples below use the same fictional contributor as the README: **Jane Doe**, Codefloe
username **`janedoe`**, email **`jane.doe@example.com`**, contributing to the `chappe`
repository.

---

## Set Up GPG Commit Signing

Every commit of a pull request must be signed with a GPG key whose public part is in
[`.forgejo/keys/`](./.forgejo/keys/), and must carry a `Signed-off-by:` line.

- **First time:** follow Steps 2 and 3 of the [one-time setup](./README.md#one-time-setup) to
  create the key and configure Git, then [Step 4](./README.md#step-4-sign-the-cla) to add the key
  to this repository.
- **Your pull request already fails the GPG or DCO check:** see
  [Fixing a failing check](#fixing-a-failing-check).

---

## Sign the CLA

Individuals sign with a pull request on this repository: see
[Step 4 of the README](./README.md#step-4-sign-the-cla). Organisations sign a Corporate CLA: see
[Corporate contributors](./README.md#corporate-contributors).

Sign the CLA **before** your first code pull request: the CLA check of a code pull request fails
until your signature is merged.

---

## Contribute from a fork

These steps assume the workspace is set up (`vidocq-workspace` cloned, `mani sync` and
`mani run -a install-hooks` done). In the workspace, each repository is cloned in `<repo>/main/`
and its `origin` remote is the Vidocq repository, which you can read but not push to. You push
to your fork, through a second remote.

### 1. Fork the repository and add your fork as a remote

On Codefloe, open the repository (for example [Vidocq/chappe](https://codefloe.com/Vidocq/chappe)),
click **Fork**, then **Fork repository**. Then, once per repository:

```bash
cd vidocq-workspace/chappe/main
git remote add fork git@codefloe.com:janedoe/chappe.git
git remote -v
```

```text
fork    git@codefloe.com:janedoe/chappe.git (fetch)
fork    git@codefloe.com:janedoe/chappe.git (push)
origin  git@codefloe.com:Vidocq/chappe.git (fetch)
origin  git@codefloe.com:Vidocq/chappe.git (push)
```

### 2. Create a branch from the latest `main`

Start from the latest `main` of the Vidocq repository, in a worktree: the workspace convention is
a sibling directory named after the branch, so `chappe/main/` stays on `main`.

```bash
git fetch origin
BRANCH=fix/42-header-without-colon BASE=origin/main mani run wt-add -p chappe
cd ../fix/42-header-without-colon
```

Without mani, the same with Git alone:
`git worktree add -b fix/42-header-without-colon ../fix/42-header-without-colon origin/main`.
A plain branch works too: `git switch -c fix/42-header-without-colon origin/main`.

Name the branch `<type>/<issue>-<short-description>`, with a type from
[Commit messages](#commit-messages).

### 3. Write the test first, then the code

Vidocq is developed test first: write a test that fails, make it pass, then clean up. Before
you push, build the repository from scratch:

```bash
./mvnw -ntp clean install
```

Always `clean install`: a stale `target/` directory can hide a failure or invent one.

### 4. Commit

```bash
git add -A
git commit -m "fix(http): reject a header line without a colon (#42)"
```

`commit.gpgsign` signs the commit, and the workspace hook adds the `Signed-off-by:` line. Check
the last commit:

```bash
git log --show-signature -1
```

### 5. Push to your fork and open the pull request

```bash
git push -u fork fix/42-header-without-colon
```

On the page of your fork (`https://codefloe.com/janedoe/chappe`), click **New pull request** and
check that it goes **into** `Vidocq/chappe`, branch `main`, **from** `janedoe/chappe`, branch
`fix/42-header-without-colon`. In the description, say what the change does, why, and which
issue it closes (`Closes #42`).

### 6. What the checks do

| Check | What it verifies |
|---|---|
| `pr-validate / governance-checks` | your CLA signature, the GPG signature and the `Signed-off-by:` line of every commit |
| `pr-validate / pr-validate` | the build and the tests of the repository, then the build of every Vidocq repository that depends on it, against your version |

The checks start once a maintainer has approved them: Codefloe does not run the workflows of a
pull request from a fork until a maintainer of the repository approves them, for one commit or
for all your pull requests in that repository. Until then, the checks show as waiting.

A pull request that changes only documentation (`*.md`, `*.adoc`, `docs/`) skips the build.
Pull requests from forks receive no CI secrets: the Slack notification and the SonarQube
analysis are skipped, which is expected. A few repositories use slightly different check names;
the pull request page lists the ones that apply.

### 7. Review, updates and merge

- A maintainer reviews the pull request. Answer the comments with new commits pushed to the same
  branch (`git push`): the pull request updates itself.
- If `main` moved on in the meantime, rebase your branch on it and push again:

  ```bash
  git fetch origin
  git rebase --gpg-sign origin/main
  git push --force-with-lease fork fix/42-header-without-colon
  ```

- **Do not use the "Update branch" button** of the pull request page. In merge mode it adds a
  commit that the forge does not sign, so the GPG check fails; in rebase mode it removes the
  signatures of your commits.
- When the review is done and the checks are green, a maintainer merges the pull request. Vidocq
  repositories accept fast-forward merges only, so your commits land on `main` as they are, with
  your own signatures.

---

## Fixing a failing check

**CLA check: `@janedoe has not signed the Vidocq CLA`.** Your CLA pull request is not merged yet,
or the username in `CLA-signatures/individuals.md` differs from your Codefloe username (check
the `@` and the spelling). Fix it on this repository, then re-run the checks of your code pull
request.

**GPG check: `The following commits are not GPG-signed or have invalid signatures`.** Possible
causes:

- the commit was made before `commit.gpgsign` was set, or from a tool that does not sign;
- the key that signed it is not in `.forgejo/keys/` (a new key, or a key other than the one you
  registered);
- your key has expired (see [When your key expires or changes](#when-your-key-expires-or-changes)).

Check locally with `git log --show-signature origin/main..HEAD`. Once your setup is right,
re-sign every commit of the branch and push:

```bash
git fetch origin && git fetch fork
git rebase --force-rebase --signoff --gpg-sign origin/main
git push --force-with-lease fork <your-branch>
```

**GPG check: `created by the forge itself`.** Someone clicked **Update branch** on the pull
request, which added an unsigned merge commit to the branch of your fork. Run the same three
commands: the rebase keeps your own commits only (it drops merge commits) and re-signs them,
and the push replaces the branch of your fork.

**DCO check: a commit has no `Signed-off-by:` line.** The same rebase adds it (`--signoff`). To
avoid it, install the workspace hooks (`mani run -a install-hooks`), or always commit with
`git commit -s`.

**`pr-validate` fails.** Open the logs from the pull request page. Reproduce the build locally
with `./mvnw -ntp clean install`. If a downstream repository fails, your change breaks one of
its users: build that repository locally against yours (see the
[workspace guide](https://codefloe.com/Vidocq/vidocq-workspace/src/branch/main/docs/getting-started.md)).

---

## Commit messages

Vidocq uses [Conventional Commits](https://www.conventionalcommits.org/), in English:

```text
<type>(<scope>): <short description> (#<issue>)

<optional body: what changed and why>

Signed-off-by: Jane Doe <jane.doe@example.com>
```

- **type:** `feat`, `fix`, `docs`, `test`, `refactor`, `perf`, `build`, `ci` or `chore`;
- **scope:** the area of the repository, for example `http` or `tls` in Chappe, `processor` in
  Vauban, `core` in Foy. Leave it out when the change has no single area;
- **issue:** the issue the commit belongs to: `(#42)` in the same repository,
  `(Vidocq/vidocq#284)` in another one.

Examples, from the history of the repositories:

```text
fix(http): share the buffer pool across virtual threads
fix(tls): the handshake unwraps buffered records before reading (#24)
docs(http2): state that server push and 103 Early Hints are not supported
```

---

## Engineering rules

The rules are written in the `AGENTS.md` file of the
[workspace](https://codefloe.com/Vidocq/vidocq-workspace/src/branch/main/AGENTS.md), with a
summary in each repository's own `AGENTS.md`. In short:

- **Strict Java Modules:** every module has a `module-info.java`, exports only its public API, and
  opens a package only with a written reason. No classpath mode.
- **Zero runtime dependencies:** what ships depends only on the Jakarta and MicroProfile APIs it
  implements. A new dependency needs a justification in the pull request.
- **Code generated at build time:** prefer the Class-File API (JEP 484) and annotation
  processors to runtime reflection. No dynamic proxies, no ASM or Byte Buddy.
- **Virtual threads for I/O:** no platform-thread pool without a documented reason.
- **Tests first:** red, green, refactor. Official TCKs run through Arquillian.
- **English everywhere:** code, Javadoc, comments, commit messages, documentation.

---

## Working with an AI assistant

The `AGENTS.md` files are read by coding agents (Codex, Claude Code, ...): opened at the root of
the workspace, they follow the same rules as you. If an AI assistant helped you write a commit,
say so with a trailer naming the tool, for example:

```text
Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

You remain the author and the committer, and the one who signs the commit: the trailer records
the provenance of the code, not a co-authorship.

---

## Bugs and benchmarks

- Every reproducible bug is tracked in the `BUG.md` file at the root of the repository concerned:
  short id, date, symptom, minimal reproduction, suspected cause, status. If your pull request
  fixes a bug, update its entry.
- Every performance figure goes into the `BENCH.md` file of the repository: date, hardware and
  JVM, exact command, raw results, difference with the previous run. No performance figure in a
  commit message or a pull request without its `BENCH.md` entry.

---

## When your key expires or changes

The GPG check verifies your commits against the key file in `.forgejo/keys/`, so this file must
follow your key.

- **To extend your key**, set a new expiry date, then update the copies:

  ```bash
  gpg --list-secret-keys --keyid-format=long jane.doe@example.com   # the 40-character line is the fingerprint
  gpg --quick-set-expire F81F0DE57A8A4A910F6DAB87CA48EF72D48EAEB5 2y
  ```

- **To switch to a new key**, create it as in Step 2 of the README and set it in
  `git config --global user.signingkey`.

In both cases, export the key again and open a pull request on this repository that replaces
`.forgejo/keys/janedoe.asc` (same commands as in [Step 4](./README.md#step-4-sign-the-cla)), and
add the new export to your Codefloe account ([Settings → SSH / GPG keys](https://codefloe.com/user/settings/keys)).

---

## Windows without WSL

WSL 2 is the supported way to work on Windows (see [Your platform](./README.md#your-platform)).
For a small change, such as a documentation fix or your CLA pull request, native Windows works
too:

1. Install [Git for Windows](https://git-scm.com/download/win) and
   [Gpg4win](https://www.gpg4win.org/), for example in PowerShell:

   ```powershell
   winget install --id Git.Git -e
   winget install --id GnuPG.Gpg4win -e
   ```

2. Git for Windows ships its own `gpg`, which does not see the keys of Gpg4win. Find the one
   installed by Gpg4win, then point Git to it:

   ```powershell
   where.exe gpg
   ```

   Pick the line that ends with `GnuPG\bin\gpg.exe` (not the one under `Git\usr\bin`), and use it
   with forward slashes, for example:

   ```powershell
   git config --global gpg.program "C:/Program Files (x86)/GnuPG/bin/gpg.exe"
   ```

3. Run the commands of Steps 2.2 to 4 of the README in **PowerShell**, where `gpg` is the one of
   Gpg4win, with these three differences:
   - write files with `--output` rather than `>`, which Windows PowerShell writes in UTF-16:

     ```powershell
     gpg --armor --output .forgejo/keys/janedoe.asc --export CA48EF72D48EAEB5
     ```

   - sign the verification token of Step 3 from a file, because a PowerShell pipe adds a line
     ending that Codefloe rejects:

     ```powershell
     Set-Content -NoNewline -Path token.txt -Value "<the token shown by Codefloe>"
     gpg --armor --default-key CA48EF72D48EAEB5 --detach-sign token.txt
     Get-Content token.txt.asc
     ```

   - copy a key or a signature to the clipboard with `| Set-Clipboard`.

   Kleopatra, installed with Gpg4win, can also create and export the key.

The workspace tooling (mani tasks, `mvnw`, TCK scripts) is written for a POSIX shell: for code
changes, use WSL 2.

---

## Using `tea` from the terminal

Codefloe runs on [Forgejo](https://forgejo.org/), not GitHub: the `gh` CLI does not work with
it. To manage pull requests from the terminal instead of the web interface, install
[`tea`](https://gitea.com/gitea/tea), the CLI for Gitea and Forgejo servers (`brew install tea`,
or a binary from the project's releases page). It is optional: everything in this guide works
from the web interface.

Log in once:

```bash
tea login add --name codefloe --url https://codefloe.com
```

Leave out `--token`, so that `tea` asks for it: a token passed as a flag stays in your shell
history. Create the token in Codefloe's **Settings → Applications**, with these scopes:

- **`user` → Read**, needed by the login itself; without it you get
  `token does not have at least one of required scope(s): [read:user]`;
- **`repository` → Read and Write**, to create and manage pull requests;
- **`issue` → Read and Write**, only to manage issues with `tea` too.

The scopes of an existing token cannot be changed: if they are wrong, create a new token.

To open the pull request of [step 5](#5-push-to-your-fork-and-open-the-pull-request) from the
terminal:

```bash
tea pr create --repo Vidocq/chappe --head janedoe:fix/42-header-without-colon --base main \
  --title "fix(http): reject a header line without a colon (#42)"
```

---

## Troubleshooting

**`error: gpg failed to sign the data`, or `gpg: signing failed: Inappropriate ioctl for device`.**
GnuPG cannot ask for your passphrase. On Linux and WSL 2, check that `GPG_TTY` is set
(`echo $GPG_TTY` prints something like `/dev/pts/0`; otherwise see Step 2.1 of the README). On
macOS, check the `pinentry-program` line of `~/.gnupg/gpg-agent.conf`. Then restart the agent with
`gpgconf --kill gpg-agent` and try again.

**`gpg: signing failed: No secret key`.** `user.signingkey` does not match a key of your keyring.
Compare `git config --global user.signingkey` with `gpg --list-secret-keys --keyid-format=long`.

**The commit shows as unverified on Codefloe.** The email of the commit
(`git log -1 --format=%ae`) must be the email of the key and a verified email of your Codefloe
account.

**`Permission denied (publickey)` when you clone or push.** Your SSH key is not on your Codefloe
account, or not the one SSH uses: see Step 1 of the README and run `ssh -T git@codefloe.com`.

**Maven: `Non-resolvable parent POM ... io.vidocq:vidocq-parent:pom:...-SNAPSHOT`.** Maven does
not know the snapshot repository where the Vidocq SNAPSHOTs are published: see the Maven step of
the [workspace guide](https://codefloe.com/Vidocq/vidocq-workspace/src/branch/main/docs/getting-started.md).

---

## Questions

Open an issue in the repository concerned, or in this one for anything about the CLA or the
contribution process, or write to [contribute@vidocq.io](mailto:contribute@vidocq.io).

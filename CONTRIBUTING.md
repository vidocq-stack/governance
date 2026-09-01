# Contributing to Vidocq

Thank you for your interest in contributing to **Vidocq**, an open source
Java implementation of Jakarta EE Core Profile 11 and MicroProfile 7.1.

---

## Table of Contents

- [Before You Start](#before-you-start)
- [Set Up GPG Commit Signing](#set-up-gpg-commit-signing)
- [Sign the CLA](#sign-the-cla)
- [Development Setup](#development-setup)
- [Submitting a Contribution](#submitting-a-contribution)
- [Code Style](#code-style)
- [Commit Message Format](#commit-message-format)

---

## Before You Start

All contributions to Vidocq require two things:

1. **GPG-signed commits** with a Signed-off-by trailer
2. A signed **Contributor License Agreement (CLA)**

Both are non-negotiable. They protect you, other contributors, and the
project users. Please follow the steps below in order before opening
your first pull request.

---

## Set Up GPG Commit Signing

Every commit must be cryptographically signed with your GPG key. This is
verified automatically on every pull request.

Rather than maintain our own copy of these instructions (and risk them
drifting out of date), follow Codeberg's own guide for the whole
generate-a-key → configure-Git → upload-to-your-account flow:

**→ [Codeberg documentation: Sign commits with GPG](https://docs.codeberg.org/security/gpg-key/)**

It walks through `gpg --full-generate-key` (recommending an RSA 4096 key),
then `git config set --global user.signingkey <KEY ID>` and
`git config set --global commit.gpgsign true` to sign every commit by
default; follow the same steps to add the resulting public key to your
**Codefloe** account and verify ownership of it (the guide itself talks about
Codeberg's own account settings, but the generate-key/configure-Git steps are
identical — only the "upload to your account" step happens on Codefloe now).
If your Git predates the `config set`
subcommand (added in Git 2.46), the older equivalent still works:
`git config --global commit.gpgsign true`.

Vidocq also expects annotated release tags to be signed, which that guide
doesn't cover — while you're there, add:

```bash
git config --global tag.gpgsign true
```

If you'd rather use a graphical interface, [Kleopatra](https://www.openpgp.org/software/kleopatra/)
(Windows/Linux) and [GPG Suite](https://gpgtools.org/) (macOS) are good options. The
[GNU Privacy Guard handbook](https://www.gnupg.org/gph/en/manual.html) is the reference for
anything Codeberg's guide doesn't cover (subkeys, smartcards, key rotation, ...).

Once your key exists and is attached to your Codefloe account, two
Vidocq-specific steps remain:

### Configure automatic Signed-off-by

If you're working inside the multi-repo
[`vidocq-workspace`](https://codefloe.com/Vidocq/vidocq-workspace) (cloned via `mani` —
see that repo's own README for the clone/setup instructions), the preferred way to do
this is:

```bash
mani run -a install-hooks
```

This points every repo's `core.hooksPath` at the workspace's shared `.githooks/`
directory, which both auto-appends the `Signed-off-by` trailer *and* hard-blocks any
commit that somehow ends up without one anyway (`--amend` without `-s`, cherry-picks,
IDE-driven commits, ...) — stronger than the config option below, and it wires up
every repo in the workspace in one shot instead of one at a time.

If you're working from a single standalone clone instead, the lighter-weight
equivalent is:

```bash
git config --local format.signoff true
```

This automatically appends `Signed-off-by: Your Name <your@email.com>`
to every commit message — but, unlike the hook above, it won't stop a commit that
ends up missing the trailer through some other path (e.g. an amend without `-s`).

### Export your public key for the CLA signature

Export your public key to a file named after your Codefloe username.
You will need this file in the next step:

```bash
gpg --export --armor your@email.com > your-codefloe-username.asc
```

---

## Sign the CLA

### Individual contributors

1. Read [CLA.md](./CLA.md) carefully.
2. Open a **dedicated pull request** (separate from your code contribution)
   with the following two changes:

   a. Add a row with your full name, Codefloe username, and the current
      date to [CLA-signatures/individuals.md](./CLA-signatures/individuals.md).

   b. Add the `.asc` file you exported in the previous step to
      `.forgejo/keys/<your-codefloe-username>.asc`.
      This file is required for the automated GPG signature verification
      to work on your future pull requests.

3. In the pull request description, include the following statement verbatim:

   > I have read and agree to the Vidocq Contributor License Agreement v1.0.

4. Once the CLA pull request is merged, your code contributions can be
   reviewed and accepted.

### Corporate contributors

If you are contributing on behalf of your employer, or if you write code
as part of your job, your organization must sign a Corporate CLA first.
Contact us at **[MAINTAINERS_EMAIL]** before submitting any contribution.

---

## Development Setup

### Prerequisites

- JDK 25 or later
- Apache Maven 3.9+
- Git 2.34+

### Build the project

```bash
git clone https://codefloe.com/vidocq/vidocq.git
cd vidocq
mvn clean install
```

### Run the tests

```bash
mvn test
```

### Work on a specific module

Each Vidocq component lives in its own repository. Clone the repository
of the module you want to work on:

```bash
git clone https://codefloe.com/vidocq/chappe.git   # HTTP server
git clone https://codefloe.com/vidocq/grimm.git    # OpenAPI
# etc.

cd chappe
mvn test
```

---

## Submitting a Contribution

Codefloe runs on [Forgejo](https://forgejo.org/), not GitHub — the `gh` CLI won't talk to
it. If you'd rather open and manage pull requests from the terminal than through the web
UI, install [`tea`](https://gitea.com/gitea/tea), the official CLI for Gitea/Forgejo
servers (`brew install tea`, or `go install code.gitea.io/tea@latest` if you have a Go
toolchain; prebuilt binaries are also published on that project's releases page). It's
entirely optional — everything below works fine from the web UI too.

#### Logging `tea` into Codefloe

`tea login add` does **not** prompt interactively for the token despite what `--help`
implies — passing `--name`/`--url` alone with no other flag fails outright with
`Error: no token set`. Read it into a shell variable instead, so the raw token never
appears as a literal command-line argument (and therefore never lands in your shell
history):

```bash
read -rs -p "Codefloe token: " TOKEN; echo
tea login add --name codefloe --url https://codefloe.com --token "$TOKEN"
unset TOKEN
```

Don't add `--user <name>` alongside `--token` — that flag switches `tea` into a
basic-auth flow that *creates a new token* from a username/password pair, and will then
fail with `Error: no password set` even though you already supplied a valid token.

You'll need a personal access token from Codefloe's **Settings → Applications**, with
these scopes:

- **`user` → Read** — required for login itself (`tea` calls the "current user" API to
  verify the token); without it you'll get
  `token does not have at least one of required scope(s): [read:user]`.
- **`repository` → Read and Write** — covers PR creation/management
  (`tea pr create`, `tea pr checkout`, ...).
- **`issue` → Read and Write** — optional, only if you'll also manage issues via `tea`.

Scopes can't be edited on an existing token — if you picked the wrong ones, generate a
new token rather than trying to fix the old one.

1. Fork the repository on Codefloe.
2. Create a feature branch from `main`:
   ```bash
   git checkout -b feat/your-feature-name
   ```
3. Make your changes, committing with GPG signature and `Signed-off-by`.
4. Push your branch and open a pull request against `main`.
5. The automated checks will verify:
   - That you have a signed CLA (your username is in `individuals.md`)
   - That all commits are GPG-signed
   - That all commits have a `Signed-off-by` trailer

A maintainer will review your pull request. Please be patient: this is a
volunteer-driven project.

---

## Code Style

- Follow standard Java conventions (Oracle Java Code Conventions).
- Use 4 spaces for indentation, no tabs.
- Maximum line length: 120 characters.
- All public APIs must have Javadoc.
- New features must be accompanied by tests.
- Do not introduce dependencies outside the Jakarta EE Core Profile 11
  and MicroProfile 7.1 specifications unless discussed with the
  maintainers first.

---

## Commit Message Format

Vidocq uses [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short description>

[optional body]

Signed-off-by: Your Name <your@email.com>
```

**Types:** `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`

**Scopes:** module names (`chappe`, `grimm`, `diderot`, `baudot`, `linne`, ...)

**Examples:**

```
feat(chappe): add HTTP/2 push promise support

Implements server push as defined in RFC 9113 section 8.4.
Closes #42.

Signed-off-by: Your Name <your@email.com>
```

```
fix(grimm): correct OpenAPI schema generation for generics

Signed-off-by: Your Name <your@email.com>
```

---

## Questions?

Open a discussion on the
[Vidocq issue tracker](https://codefloe.com/vidocq/vidocq/issues)
or reach out at **[MAINTAINERS_EMAIL]**.

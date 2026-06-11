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

If you are new to GPG, the following resources will help you get started:

- [The GNU Privacy Guard handbook](https://www.gnupg.org/gph/en/manual.html)
- [GitHub guide to GPG commit signing](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)
- [Codeberg documentation: Sign commits with GPG](https://docs.codeberg.org/security/gpg-key/)

If you prefer a graphical interface, [Kleopatra](https://www.openpgp.org/software/kleopatra/)
(Windows/Linux) and [GPG Suite](https://gpgtools.org/) (macOS) are recommended.

### Generate a GPG key (if you do not have one)

```bash
gpg --batch --gen-key <<EOF
Key-Type: ECDSA
Key-Curve: ed25519
Subkey-Type: ECDH
Subkey-Curve: cv25519
Name-Real: Your Full Name
Name-Email: your@email.com
Expire-Date: 2y
%ask-passphrase
EOF
```

### Configure git to sign all commits

```bash
# Get your key ID
gpg --list-secret-keys --keyid-format=long your@email.com

# Configure git (replace KEY_ID with your actual key ID)
git config --global user.signingkey KEY_ID
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

### Configure automatic Signed-off-by

```bash
git config --local format.signoff true
```

This automatically appends `Signed-off-by: Your Name <your@email.com>`
to every commit message.

### Add your public key to your Codeberg account

Export your public key and add it under **Settings > SSH/GPG Keys**
in your Codeberg profile:

```bash
gpg --export --armor your@email.com
```

### Export your public key for the CLA signature

Export your public key to a file named after your Codeberg username.
You will need this file in the next step:

```bash
gpg --export --armor your@email.com > your-codeberg-username.asc
```

---

## Sign the CLA

### Individual contributors

1. Read [CLA.md](./CLA.md) carefully.
2. Open a **dedicated pull request** (separate from your code contribution)
   with the following two changes:

   a. Add a row with your full name, Codeberg username, and the current
      date to [CLA-signatures/individuals.md](./CLA-signatures/individuals.md).

   b. Add the `.asc` file you exported in the previous step to
      `.forgejo/keys/<your-codeberg-username>.asc`.
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
git clone https://codeberg.org/vidocq/vidocq.git
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
git clone https://codeberg.org/vidocq/chappe.git   # HTTP server
git clone https://codeberg.org/vidocq/grimm.git    # OpenAPI
# etc.

cd chappe
mvn test
```

---

## Submitting a Contribution

1. Fork the repository on Codeberg.
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
[Vidocq issue tracker](https://codeberg.org/vidocq/vidocq/issues)
or reach out at **[MAINTAINERS_EMAIL]**.

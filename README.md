# Vidocq — Governance

This repository holds the rules every contribution to [Vidocq](https://vidocq.dev) goes through,
and the records the CI checks them against:

- the **Contributor License Agreement** ([`CLA.md`](./CLA.md)) and the list of its signatories
  ([`CLA-signatures/`](./CLA-signatures/));
- the **GPG public keys** of the contributors ([`.forgejo/keys/`](./.forgejo/keys/)), used to verify
  that every commit is signed;
- the guide for **contributors** ([`CONTRIBUTING.md`](./CONTRIBUTING.md)) and the guide for
  **maintainers** ([`MAINTAINERS.md`](./MAINTAINERS.md)).

Vidocq is an open source Java implementation of **Jakarta EE Core Profile 11** and
**MicroProfile 7.1**, distributed under the triple licence
`EPL-2.0 OR EUPL-1.2 OR GPL-2.0-or-later`.

---

## Contributors and maintainers

Everyone starts as a **contributor**: you work in your own fork of a Vidocq repository and open
pull requests from it. After a period of contributions, the maintainers may invite you to become
a **maintainer**. It is their decision; there is no automatic threshold (see
[Becoming a maintainer](./MAINTAINERS.md#becoming-a-maintainer)).

| | Contributor | Maintainer |
|---|---|---|
| Who | Anyone who signed the CLA | A contributor invited by the maintainers |
| You push to | your fork on Codefloe (`janedoe/chappe`) | branches of the Vidocq repositories (`Vidocq/chappe`) |
| Your pull request is merged by | a maintainer | you, with the `/merge` command |
| What you follow | the [one-time setup](#one-time-setup) below, then [`CONTRIBUTING.md`](./CONTRIBUTING.md) | [`MAINTAINERS.md`](./MAINTAINERS.md) |

The workstation setup (tools, cloning, building) is the same for both. It is described in the
[Vidocq workspace guide](https://codefloe.com/Vidocq/vidocq-workspace/src/branch/main/docs/getting-started.md).

---

## One-time setup

Every pull request to a Vidocq repository goes through three automatic checks. A pull request
that fails one of them cannot be merged.

| Check | Passes when | Set up in |
|---|---|---|
| **CLA** | the author of the pull request is listed in `CLA-signatures/individuals.md` | Step 4 |
| **GPG** | every commit is signed with a key whose public part is in `.forgejo/keys/` | Steps 2 to 4 |
| **DCO** | every commit carries a `Signed-off-by:` line | Step 2 |

The four steps below set them up. You do them once, before your first pull request; count about
20 minutes.

> **About the examples.** They use a fictional contributor, **Jane Doe**, Codefloe username
> **`janedoe`**, email **`jane.doe@example.com`**, GPG key ID **`CA48EF72D48EAEB5`**. Replace
> them with yours.

### Your platform

- **macOS** and **Linux**: run the commands as shown. When a command differs, the step gives
  both versions.
- **Windows**: use **WSL 2** (Windows Subsystem for Linux) and run the Linux commands inside it.
  The Vidocq tooling (git hooks, `mvnw`, mani tasks, TCK scripts) is written for a POSIX shell.
  In a PowerShell window opened with **Run as administrator**:

  ```powershell
  wsl --install -d Ubuntu
  ```

  Restart Windows, open **Ubuntu** from the Start menu and create your Linux user. From then on,
  type every command in that Ubuntu terminal and keep your clones in your Linux home (`~/...`),
  not under `/mnt/c/`. For a small change without WSL, see
  [Windows without WSL](./CONTRIBUTING.md#windows-without-wsl).

### Step 1: Codefloe account and SSH key

The Vidocq repositories are hosted on [Codefloe](https://codefloe.com), a
[Forgejo](https://forgejo.org) instance.

1. Create an account on [codefloe.com](https://codefloe.com/user/sign_up). The CLA check looks
   for your **username**.
2. In [Settings → Account](https://codefloe.com/user/settings/account), check that the email
   address you will commit with is listed and **verified**. Codefloe shows a commit as signed
   only when the email of the signing key is a verified email of your account.
3. Create an SSH key. It lets you clone and push with the `git@codefloe.com:` addresses that the
   Vidocq workspace uses. Skip the first command if you already have a `~/.ssh/id_ed25519.pub`.

   ```bash
   ssh-keygen -t ed25519 -C "jane.doe@example.com"   # Enter to keep the default file, then choose a passphrase
   cat ~/.ssh/id_ed25519.pub
   ```

   Copy the line printed by `cat`. In [Settings → SSH / GPG keys](https://codefloe.com/user/settings/keys),
   under **Manage SSH keys**, click **Add key**, paste the line and save.
4. Check the connection. The first time, SSH asks you to confirm the host: type `yes`.

   ```bash
   ssh -T git@codefloe.com
   ```

   ```text
   Hi there, janedoe! You've successfully authenticated with the key named ..., but Forgejo does not provide shell access.
   ```

### Step 2: GPG key and Git configuration

**2.1 Install GnuPG.**

On **macOS**, with [Homebrew](https://brew.sh). `pinentry-mac` asks for your passphrase in a
window and can remember it in the macOS keychain:

```bash
brew install gnupg pinentry-mac
mkdir -p ~/.gnupg && chmod 700 ~/.gnupg
echo "pinentry-program $(brew --prefix)/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
gpgconf --kill gpg-agent
```

On **Linux** and **Windows (WSL 2)**, GnuPG asks for your passphrase in the terminal, and needs
the `GPG_TTY` variable to find it:

```bash
sudo apt update && sudo apt install -y gnupg    # Debian, Ubuntu, WSL 2
sudo dnf install -y gnupg2                      # Fedora
sudo pacman -S --needed gnupg                   # Arch Linux

echo 'export GPG_TTY=$(tty)' >> ~/.bashrc       # with zsh: ~/.zshrc
source ~/.bashrc
```

Check that `gpg --version` prints `gpg (GnuPG) 2.2` or later.

**2.2 Create a signing key.** Skip this if you already have a key for the email address of your
Codefloe account (`gpg --list-secret-keys --keyid-format=long` lists your keys; an RSA 4096 key
works too).

```bash
gpg --quick-generate-key "Jane Doe <jane.doe@example.com>" ed25519 sign 2y
```

GnuPG asks for a passphrase twice. The key is valid for two years (`2y`); you can extend it
later, see [When your key expires or changes](./CONTRIBUTING.md#when-your-key-expires-or-changes).

**2.3 Find the key ID.**

```bash
gpg --list-secret-keys --keyid-format=long jane.doe@example.com
```

```text
sec   ed25519/CA48EF72D48EAEB5 2026-10-21 [SC] [expires: 2028-10-20]
      F81F0DE57A8A4A910F6DAB87CA48EF72D48EAEB5
uid                 [ultimate] Jane Doe <jane.doe@example.com>
```

The key ID is the 16 characters after `ed25519/`, here `CA48EF72D48EAEB5`.

**2.4 Configure Git** to sign every commit and tag with that key, under the same name and email
as the key:

```bash
git config --global user.name "Jane Doe"
git config --global user.email "jane.doe@example.com"
git config --global user.signingkey CA48EF72D48EAEB5
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

The `Signed-off-by:` line (DCO) is added by `git commit -s`. Once you have cloned the Vidocq
workspace, its git hooks add it to every commit for you.

**2.5 Check** with a commit in a throw-away repository:

```bash
cd "$(mktemp -d)" && git init -q
git commit --allow-empty -s -m "test: commit signing"
git log --show-signature -1
```

The output must contain these two lines:

```text
gpg: Good signature from "Jane Doe <jane.doe@example.com>"
    Signed-off-by: Jane Doe <jane.doe@example.com>
```

If the commit fails, see [Troubleshooting](./CONTRIBUTING.md#troubleshooting).

### Step 3: Register the GPG key on Codefloe

1. Print your public key:

   ```bash
   gpg --armor --export CA48EF72D48EAEB5
   ```

   To copy it to the clipboard directly, append `| pbcopy` on macOS, `| clip.exe` on WSL 2,
   `| wl-copy` on Linux with Wayland, or `| xclip -selection clipboard` on Linux with X11.
2. In [Settings → SSH / GPG keys](https://codefloe.com/user/settings/keys), under
   **Manage GPG keys**, click **Add key** and paste the whole block, including the
   `-----BEGIN PGP PUBLIC KEY BLOCK-----` and `-----END PGP PUBLIC KEY BLOCK-----` lines.
3. Click **Verify** next to the key. Codefloe shows a token: sign it, then paste the signature
   block back into the form.

   ```bash
   echo -n "<the token shown by Codefloe>" | gpg --armor --default-key CA48EF72D48EAEB5 --detach-sig
   ```

### Step 4: Sign the CLA

You sign the CLA with a pull request on this repository, opened from your fork. It adds your
name to the list of signatories and your public key to `.forgejo/keys/`: the GPG check verifies
your future commits against that file.

1. Read [`CLA.md`](./CLA.md). If you contribute as part of your job, read
   [Corporate contributors](#corporate-contributors) first.
2. Open [Vidocq/governance](https://codefloe.com/Vidocq/governance) and click **Fork** (top right),
   then **Fork repository**. You now have `janedoe/governance`.
3. Clone your fork and create a branch:

   ```bash
   git clone git@codefloe.com:janedoe/governance.git
   cd governance
   git switch -c cla/janedoe
   ```

4. In `CLA-signatures/individuals.md`, add a row at the end of the table, with your full name,
   your Codefloe username **prefixed with `@`**, and today's date:

   ```markdown
   | Jane Doe | @janedoe | 2026-10-21 |
   ```

   The check searches for `@` followed by your exact username: a typo here makes all your pull
   requests fail.
5. Add your public key, in a file named after your username:

   ```bash
   gpg --armor --export CA48EF72D48EAEB5 > .forgejo/keys/janedoe.asc
   ```

6. Commit (signed, with `-s` for the `Signed-off-by:` line) and push to your fork:

   ```bash
   git add CLA-signatures/individuals.md .forgejo/keys/janedoe.asc
   git commit -s -m "docs: sign the Vidocq CLA (@janedoe)"
   git push -u origin cla/janedoe
   ```

7. Open the pull request: on the page of your fork, click **New pull request**. Check that it
   goes **into** `Vidocq/governance`, branch `main`, **from** `janedoe/governance`, branch
   `cla/janedoe`. Use the commit message as the title, and paste this sentence, word for word,
   in the description. It is your signature:

   > I have read and agree to the Vidocq Contributor License Agreement v1.0.

8. A maintainer checks the row, the key and the sentence, then merges. From then on, the CLA and
   GPG checks pass for you in every Vidocq repository.

### Next steps

- Set up your workstation and make your first contribution: follow the
  [Vidocq workspace guide](https://codefloe.com/Vidocq/vidocq-workspace/src/branch/main/docs/getting-started.md).
- How a pull request works, what the checks do and how to fix one that fails:
  [`CONTRIBUTING.md`](./CONTRIBUTING.md).

---

## Corporate contributors

If you contribute on behalf of your employer, or as part of your job, your organisation signs a
Corporate CLA first. Write to [contribute@vidocq.io](mailto:contribute@vidocq.io) with the
subject `Corporate CLA - <organisation name>`; the procedure is in
[`CLA-signatures/corporate.md`](./CLA-signatures/corporate.md). Each employee still follows
[Step 4](#step-4-sign-the-cla) and names the organisation in the description of the pull request.

---

## How the checks use this repository

The checks are composite actions of [`Vidocq/ci`](https://codefloe.com/Vidocq/ci): `cla-check`,
`gpg-check` and `dco-check`, bundled as `governance-checks` in most repositories. On every pull
request they clone this repository anonymously, which is why it must stay **public**:

- `cla-check` searches `@<author of the pull request>` in `CLA-signatures/individuals.md`;
- `gpg-check` imports every `.forgejo/keys/*.asc` file and runs `git verify-commit` on each
  commit of the pull request;
- `dco-check` requires a `Signed-off-by:` line in each commit.

`.forgejo/workflows/contribution-checks.yml` is an earlier version of these checks. No
repository uses it anymore.

---

## Contents

| Path | What it is |
|---|---|
| [`CLA.md`](./CLA.md) | Contributor License Agreement, version 1.0 |
| [`CLA-signatures/individuals.md`](./CLA-signatures/individuals.md) | Individual signatures, read by the CLA check |
| [`CLA-signatures/corporate.md`](./CLA-signatures/corporate.md) | Organisations that signed the Corporate CLA |
| [`.forgejo/keys/`](./.forgejo/keys/) | GPG public keys, read by the GPG check; `vidocq-ci-bot.asc` is the key of the CI bot (see the [README](./.forgejo/keys/README.md) there) |
| [`CONTRIBUTING.md`](./CONTRIBUTING.md) | Contributor guide |
| [`MAINTAINERS.md`](./MAINTAINERS.md) | Maintainer guide |

---

## Licences

Vidocq is distributed under a disjunctive triple licence: each recipient chooses one of them.

```text
SPDX-License-Identifier: EPL-2.0 OR EUPL-1.2 OR GPL-2.0-or-later
```

- [Eclipse Public License 2.0 (EPL-2.0)](https://www.eclipse.org/legal/epl-2.0/)
- [European Union Public Licence 1.2 (EUPL-1.2)](https://joinup.ec.europa.eu/collection/eupl/eupl-text-eupl-12)
- [GNU General Public License v2.0 or later (GPL-2.0-or-later)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html)

Section 4 of the CLA lets the maintainers distribute contributions under these licences.

---

## Contact

For any question about contributing or the CLA, open an issue in this repository or write to
[contribute@vidocq.io](mailto:contribute@vidocq.io).

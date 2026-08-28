# Contributors GPG public keys

This directory contains the GPG public keys of all contributors
who have signed the Vidocq CLA.

Each file is named `<codeberg-username>.asc` and contains the
contributor's armored public key.

These keys are imported automatically by the contribution-checks
workflow to verify commit signatures on every pull request.

## Adding your key

Export your public key and add it here as part of your CLA
signature pull request:

```bash
gpg --export --armor your@email.com > your-codeberg-username.asc
```

## Service identities

`vidocq-ci-bot.asc` is not a CLA signatory. It is the public key of a
dedicated service identity ("Vidocq CI Bot") used exclusively to sign
git commits produced by automation on behalf of already-approved,
already-signed-off changes:

- the `release:` and `post-release:` commits pushed directly to `main`
  by the `release-maven` action, and
- the rebase-and-fast-forward commits produced by the merge-bot once a
  pull request has been approved and its required checks are green.

This key never authors new content — it only re-signs commits whose
author, message, and `Signed-off-by` trailer already belong to a human
contributor who went through the normal CLA/DCO process. It is
deliberately separate from the GPG key used to sign released Maven
artifacts (`secrets.GPG_PRIVATE_KEY`), so that compromising one key
cannot be used to forge the other kind of signature.

- Key ID: `38A7110A14A77BFA`
- Fingerprint: `AE2D 71FA 4AFE 47EB A8D9  FC5C 38A7 110A 14A7 7BFA`
- UID: `Vidocq CI Bot (git commit signing) <ci@vidocq.io>`
- Expires: **2028-08-27** — rotate this key (and the `CI_BOT_GPG_*` org
  secrets, and the copy registered on the `vidocq-ci-bot` account below)
  before that date, or every commit-signing job starts failing verification.

### Account registration (required, not optional)

Publishing this file here is documentation only — it has **no effect** on
Forgejo's `require_signed_commits` check. That check verifies signatures
against GPG keys attached to a **Forgejo user account**, and requires the
commit's `user.email` to match a *verified* email on that same account. A
key that only exists as a file in this repo will still be rejected with
`gpg.error.no_gpg_keys_found`.

So this key must also be registered on a dedicated `vidocq-ci-bot` Forgejo
account:

1. A `vidocq-ci-bot` account exists, with `ci@vidocq.io` as a verified
   email (this must match the `bot-email` input used by `release-maven`
   and `merge-bot`, currently defaulted to `ci@vidocq.io`).
2. The public key in this file must be added under that account's
   **Settings → SSH/GPG Keys → Add Key** (paste the full ASCII-armored
   block, including the `-----BEGIN/END PGP PUBLIC KEY BLOCK-----`
   lines).
3. `secrets.VIDOCQ_BOT_TOKEN` (org-level) must be a personal access token
   generated **from the `vidocq-ci-bot` account itself**, not a
   maintainer's personal token, so that commits pushed by automation are
   authored/attributed to the bot account whose GPG key this is.

Until steps 2 and 3 above are done, `release-maven` and `merge-bot` will
still fail signature verification on any repo with
`require_signed_commits` enabled, even though the private key material is
already wired into the `CI_BOT_GPG_*` secrets.

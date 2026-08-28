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

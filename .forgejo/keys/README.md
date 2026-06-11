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

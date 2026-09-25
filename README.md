# Vidocq — Governance

This repository centralizes all contribution rules, legal agreements, and
community health files for the [Vidocq project](https://codefloe.com/Vidocq).

Vidocq is an open source Java implementation of
**Jakarta EE Core Profile 11** and **MicroProfile 7.1**, released under a
triple licence: EPL-2.0, GPL, and EUPL-1.2.

---

## Contents

| Path | Description |
|---|---|
| [`CLA.md`](./CLA.md) | Contributor License Agreement (v1.0) |
| [`CONTRIBUTING.md`](./CONTRIBUTING.md) | Full contribution guide |
| [`CLA-signatures/individuals.md`](./CLA-signatures/individuals.md) | Individual CLA signatures |
| [`CLA-signatures/corporate.md`](./CLA-signatures/corporate.md) | Corporate CLA signatures |
| [`.forgejo/keys/`](./.forgejo/keys/) | Contributors GPG public keys |
| [`.forgejo/workflows/contribution-checks.yml`](./.forgejo/workflows/contribution-checks.yml) | Reusable Forgejo Actions workflow |

---

## For contributors

Before submitting any pull request to a Vidocq component repository, you must:

1. Set up GPG commit signing on your machine
2. Sign the CLA by opening a pull request **against this repository**

The full procedure is described in [CONTRIBUTING.md](./CONTRIBUTING.md).

---

## For component repositories

Each Vidocq component repository (Chappe, Grimm, Diderot, Baudot, Linne, ...)
delegates its contribution checks to this repository via a single reusable
Forgejo Actions workflow call.

To add the checks to a new component repository, create the following file:

```yaml
# .forgejo/workflows/contribution-checks.yml
name: Contribution checks

on:
  pull_request:
    branches:
      - main

jobs:
  checks:
    uses: vidocq/governance/.forgejo/workflows/contribution-checks.yml@main
    with:
      pr-author: ${{ github.event.pull_request.user.login }}
```

And add a minimal `CONTRIBUTING.md` pointing here:

```markdown
# Contributing to [Component]

Please read the [Vidocq Contribution Guide](https://codefloe.com/Vidocq/governance/src/branch/main/CONTRIBUTING.md).
```

---

## Licences

The Vidocq project is available under a triple licence. Recipients may choose
any of the following:

- [Eclipse Public License 2.0 (EPL-2.0)](https://www.eclipse.org/legal/epl-2.0/)
- [GNU General Public License (GPL)](https://www.gnu.org/licenses/gpl.html)
- [European Union Public Licence 1.2 (EUPL-1.2)](https://joinup.ec.europa.eu/collection/eupl/eupl-text-eupl-12)

---

## Maintainers

Vidocq is a personal open source project maintained by its founders.
For any question regarding contributions or the CLA, open an issue in this
repository or contact the maintainers at [**contribute@vidocq.io**](mailto:contribute@vidocq.io).

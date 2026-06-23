# my-aur-packages

CI-driven AUR maintenance using
[aur-updater](https://github.com/kralonur/aur-updater).

`packages.toml` lists the packages this repo keeps in sync. The
`.github/workflows/update.yml` workflow checks upstream versions on a schedule,
bumps `pkgver`, regenerates checksums and `.SRCINFO`, then commits the change
here and pushes the result to AUR.

## Layout

```
packages.toml        # list of packages + upstream sources
eim-bin/             # mirror of the AUR package directory (tracked files only)
.github/workflows/   # update workflow
```

## Adding a package

Append a `[[package]]` block to `packages.toml` and copy the AUR-tracked files
(`PKGBUILD`, `.SRCINFO`, auxiliary sources, `.gitignore`) into a matching
directory.

## Secrets

The AUR push step needs two GitHub Actions secrets:

- `AUR_SSH_PRIVATE_KEY` - a dedicated Ed25519 private key registered on the AUR
  account (do not reuse your personal key).
- `AUR_SSH_KNOWN_HOSTS` - output of `ssh-keyscan -t ed25519 aur.archlinux.org`,
  pinned to prevent MITM.

Both are injected only on `schedule`/`push`/`workflow_dispatch`, never on
`pull_request`, so fork PRs cannot read the key.

## Security notes

- Keep `main`/`master` protected: required reviews, no direct pushes to
  `.github/` by anyone, treat administrators like everyone else.
- Review any PR that touches `.github/workflows/` or `packages.toml` before
  merging - a malicious workflow change is the only realistic exfiltration path.
- The CI key is revocable instantly from the AUR account SSH keys page; rotate
  the GitHub secret if it is ever suspected.
# my-aur-packages

CI-driven AUR maintenance using
[aur-updater](https://github.com/kralonur/aur-updater).

`packages.toml` lists the packages this repo keeps in sync. The
`.github/workflows/update.yml` workflow checks upstream versions on a schedule,
bumps `pkgver`, regenerates checksums and `.SRCINFO`, then commits the change
here and reconciles the tracked package files with AUR.

## Layout

```
packages.toml        # list of packages + upstream sources
eim-bin/             # mirror of the AUR package directory (tracked files only)
snander/             # mirror of the AUR package directory (tracked files only)
.github/workflows/   # update workflow
```

## Packages

### eim-bin

- **AUR package:** [`eim-bin`](https://aur.archlinux.org/packages/eim-bin)
  (split: `eim-cli`, `eim-gui`)
- **Upstream source:** GitHub release, tracked via
  [aur-updater](https://github.com/kralonur/aur-updater) `source = "github_release"`.
- **Upstream repository:**
  [`espressif/idf-im-ui`](https://github.com/espressif/idf-im-ui)
  (ESP-IDF Installation Manager UI; binary releases of the `eim` CLI and GUI).
- **Version detection:** latest non-prerelease GitHub release tag, with the `v`
  prefix stripped (e.g. `v0.14.1` -> `0.14.1`).
- **Architectures:** `x86_64`, `aarch64`, `armv7h` (see `eim-bin/PKGBUILD`).
- **Mirrored files** under `eim-bin/`:
  - `PKGBUILD`
  - `.SRCINFO`
  - `eim-gui.desktop`
- **Update path:** when a new release appears, `aur-updater` bumps `pkgver`,
  resets `pkgrel=1`, refreshes the per-arch `sha256sums_*` blocks via
  `updpkgsums`, regenerates `.SRCINFO`, then CI commits the change here and
  pushes the updated tracked package files to the AUR package repository.

The `.zip` release artifacts, `pkg/`, `src/`, and `icon.png` are not tracked
(see `eim-bin/.gitignore`) - only the package recipe is version-controlled.

### snander

- **AUR package:** [`snander`](https://aur.archlinux.org/packages/snander)
- **Upstream source:** GitHub release tag from
  [`McMCCRU/SNANDer`](https://github.com/McMCCRU/SNANDer), tracked via
  [aur-updater](https://github.com/kralonur/aur-updater) `source = "github_release"`.
- **Version detection:** latest non-prerelease GitHub release tag, with the
  `v.` prefix stripped (e.g. `v.1.7.9.3` -> `1.7.9.3`).
- **Architecture:** `x86_64`.
- **Mirrored files** under `snander/`:
  - `PKGBUILD`
  - `.SRCINFO`
- **Update path:** when a new release appears, `aur-updater` updates the
  version and checksum, regenerates `.SRCINFO`, then CI commits the change here
  and pushes the updated tracked package files to the AUR package repository.

The update workflow processes every package configured in `packages.toml`,
creates a separate maintenance-repository commit for each changed package, and
reconciles all tracked package files with the matching AUR repository. The
tracked files in this repository are authoritative for that synchronization.

## Adding a package

Append a `[[package]]` block to `packages.toml` (see the
[aur-updater config docs](https://github.com/kralonur/aur-updater#config) for
all supported source types and fields) and copy the AUR-tracked files
(`PKGBUILD`, `.SRCINFO`, auxiliary sources, `.gitignore`) into a matching
directory. Add a short entry to the **Packages** section above describing the
package and its upstream source.

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
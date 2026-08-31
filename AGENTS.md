# AGENTS.md

AUR package `vrcx-0-bin`: a binary wrapper around the upstream VRCX-0 (Tauri) Linux `.deb`. There is no application source in this repo — the only kind of change is a version bump.

## Update workflow (the only task in this repo)

1. Check the latest release at https://github.com/Map1en/VRCX-0/releases.
2. Bump `pkgver` in `PKGBUILD`. Do NOT edit the `source` line — it is templated with `${pkgver}`.
3. Update `sha256sums` to the sha256 of `VRCX-0_<ver>_linux_x86_64.deb` (e.g. download it, then `updpkgsums`, or `sha256sum` manually).
4. Regenerate `.SRCINFO`: `makepkg --printsrcinfo > .SRCINFO` (it mirrors the PKGBUILD, with the source URL expanded to the concrete version).
5. Commit with the exact message style `Update to <version>` (see history) and push to `origin` (AUR: `ssh://aur@aur.archlinux.org/vrcx-0-bin.git`).

## Auto-update (GitHub Actions)

`.github/workflows/auto-update.yml` (runs on the **GitHub mirror** of this repo) does the update automatically, so the manual workflow above is only a fallback:

- Triggers: every 30 min (`schedule`) + manual (`workflow_dispatch`).
- Polls `https://api.github.com/repos/Map1en/VRCX-0/releases/latest`; if the tag differs from `pkgver`, it downloads the `.deb`, rewrites `pkgver` + `sha256sums`, regenerates `.SRCINFO` via `makepkg --printsrcinfo` (Arch container), commits `Update to <version>`, and pushes to GitHub **and** to the AUR remote.
- AUR push uses an SSH deploy key from the repo secret `AUR_SSH_KEY`; the AUR ed25519 host key is pinned in the workflow.

One-time setup:

- A GitHub repo mirroring this one (e.g. `BlackCherryCat/vrcx-0-bin`). Locally: `git remote add github <url>`; `origin` stays AUR. Push to **both** remotes for every commit (the bot pushes to both, but manual bumps must too, or the mirror drifts).
- Write SSH deploy key added on the AUR package page (`vrcx-0-bin` → SSH deploy keys). Private key: `~/.ssh/aur-vrcx0-bin_deploy` — paste it as the `AUR_SSH_KEY` secret in the GitHub repo.

## Notes

- Verify before committing: `makepkg --printsrcinfo | diff - .SRCINFO` (should be empty); optional build check: `makepkg -sCg`.
- `package()` extracts `data.tar.gz` (the `.deb` payload) with `bsdtar` into `$pkgdir`; makepkg downloads and unpacks the `.deb` into `$srcdir` automatically — do not add manual download/extraction steps.
- `*.deb`, `*.pkg.tar.*`, `src/`, and `pkg/` are gitignored build leftovers — never commit them.
- Runtime deps are `webkit2gtk-4.1` and `libappindicator` (what the upstream Tauri deb needs); `conflicts`/`provides` handle the source package `vrcx-0`.

# Automatic upstream sync and hbmenu releases

This repository tracks the official [switchbrew/nx-hbmenu](https://github.com/switchbrew/nx-hbmenu).
The automation lives on the repository default branch (`cmakeify`), while `master`
is fast-forwarded to upstream `master` without rewriting unrelated local commits.

## Schedule and triggers

- GitHub Actions: [sync-build-release.yml](../.github/workflows/sync-build-release.yml)
- Poll upstream every 10 minutes (GitHub-hosted schedules can be delayed).
- A workflow file update on `cmakeify` starts an initial run.
- **Actions -> Sync upstream and build hbmenu -> Run workflow** allows manual retries.
- Only builds if there is no release for the current upstream commit/version.

## Building

Source: upstream `master` at a recorded full commit SHA.

Toolchain: `devkitpro/devkita64:20260219` container, including devkitA64,
libnx and Switch portlibs. This project uses upstream Makefiles, **not** the
experimental CMake branch.

```sh
export DEVKITPRO=/opt/devkitpro
export DEVKITA64=/opt/devkitpro/devkitA64
export PATH="$DEVKITA64/bin:$DEVKITPRO/tools/bin:$PATH"
make -j2 nx RELEASE=1
```

This produces `nx-hbmenu.nro`, which the workflow copies to `hbmenu.nro`.

## Releases

For upstream version `X.Y.Z` and commit `abcdef...`, the automatic tag is
`vX.Y.Z-auto.abcdef123456` (the commit prefix is 12 hex characters).

Each GitHub Release includes:
- `hbmenu.nro` - standalone Nintendo Switch Homebrew Menu
- `hbmenu.zip` - the same NRO in a ZIP archive
- `SHA256SUMS` - checksums for both files

The build is independently produced from official source. It is not an
official switchbrew release. Copy `hbmenu.nro` to the Switch SD card root,
unless your homebrew loader requires a different path.

## Safety and troubleshooting

1. The sync job checks that the existing local `master` is an ancestor of
   the upstream head. Diverged branches cause the job to fail rather than
   force-push away local work.
2. The build job uses a read-only GitHub token and a pinned image tag. Only
   the sync/release jobs need `contents: write`.
3. Build failure blocks release creation; check Actions logs and retry.
4. If a release already exists for a SHA, subsequent checks skip the build.
5. Upstream `cmakeify` and `sdl2-renderer` are deliberately not overwritten
   by this master-based release automation.
6. GitHub may delay scheduled workflows during periods of high usage; the
   10-minute cron is a *requested* polling interval, not an SLA.

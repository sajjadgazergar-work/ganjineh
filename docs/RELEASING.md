# Releasing Ganjineh

This repository holds documentation, screenshots and third-party notices. **The app itself is never
committed here** — it ships as a GitHub Release asset. This page is the maintainer's checklist for
turning a working build into a public `v1.1.0` release, written down so the process can be repeated
and audited.

Two honest notes before you start:

- Ganjineh is distributed as a **packaged Electron app plus its assets**, not as source you compile.
  These steps re-pack the packaged app on the maintainer's machine; they are not a source build.
- Nothing in this repository automates the release. There is no CI pipeline, no code signing and no
  auto-updater API. Every step below is a command a human runs and then checks.

`<your-github-user>` stands for the GitHub account that owns the repository. It is the only value in
these docs that has to be filled in by the owner.

## What a release consists of

| Piece | Where it lives |
|---|---|
| Docs + screenshots | A commit on `main` in this repository |
| Tag | `v1.1.0` |
| Release asset 1 | `Ganjineh-1.1.0-win-x64.zip` (portable Windows 10/11 x64 build) |
| Release asset 2 | `SHA256SUMS.txt` (one line: hash + the exact zip filename) |

The zip is never committed: `Ganjineh.exe` alone is about 238 MB, well over GitHub's 100 MB per-file
limit, and the extracted payload is about 613 MB. Asset uploads of this size are fine (the limit per
release asset is 2 GB).

## Prerequisites

- A Windows 10/11 x64 machine with the unpacked app and its `_work\` tooling (the `_work` folder is
  the maintainer's working tree: editable renderer/CSS sources, the extracted `app.asar` tree and the
  build scripts). It is not part of the published repository.
- PowerShell 7 (`pwsh`) or Windows PowerShell 5.1.
- Node.js 20+ with `npx`, and these packages already present in the local `npx` cache — the toolchain
  runs offline: `esbuild`, `@electron/asar`, `resedit`.
- Git and the GitHub CLI (`gh`, signed in), or a browser for the web UI.
- Roughly 3 GB free disk space: the staged payload (~613 MB), the zip (about 273 MB for 1.1.0), and
  room to extract a verification copy.

## 1. Bump the version

Three places must agree:

1. `_work\asar\package.json` — `"version": "1.1.0"` (also `"productName": "Ganjineh"`,
   `"license": "MIT"`).
2. The rebuilt `resources\app.asar`, which carries that `package.json`.
3. The executable's own version resource — `Ganjineh.exe` ships patched to `1.1.0.0`:

   ```powershell
   node .\_work\release\set-exe-version.cjs .\Ganjineh.exe            # dry run -> Ganjineh.exe.new
   node .\_work\release\set-exe-version.cjs .\Ganjineh.exe --apply    # verify, then swap
   (Get-Item .\Ganjineh.exe).VersionInfo | Format-List FileVersion, ProductVersion, FileDescription
   ```

   The script re-parses its own output before swapping, so a bad rewrite cannot replace a working
   executable. It leaves `Ganjineh.exe.new` behind on a dry run — that file must never be staged.

## 2. Build the packaged app

Close Ganjineh first: a running instance keeps the old `app.asar` in memory and the exe locked.

```powershell
Set-Location C:\Users\ASUS\Documents\Ganjineh-1.0.1-win-x64
& .\_work\build.ps1      # esbuild: minify renderer + CSS, then repack resources\app.asar
```

`build.ps1 -NoPack` builds the two asset bundles without re-packing the asar, which is useful for a
quick syntax check. The script prints the packed asar size; expect a value close to the previous
release rather than a step change.

## 3. Smoke-test before staging

```powershell
& .\_work\restart-app.ps1 -NoBuild    # relaunch with CDP on :9333 (drop -NoBuild to build first)
node .\_work\audit-explore.mjs        # Explore audit -> screenshots in _work\shots
node .\_work\probe-rpc.mjs            # exercise the explore.* RPCs
```

Check by hand, in both languages and both looks: the first-run wizard, one download, one Explore tab,
one bulk selection, and Settings › About & logs showing version `1.1.0`. Fix anything that fails
before you continue — a released asset is public.

## 4. Stage a clean payload

Copy the shippable tree to `_work\release\dist\Ganjineh-1.1.0-win-x64\`. The staged folder is exactly
what a user sees after extracting: the runtime files, `locales\`, and `resources\`.

**Include** — the runtime files below plus `locales\` and `resources\`; 56 files before the legal
texts, 58 with them, about 613 MB:

```
Ganjineh.exe  README.txt  portable.txt  LICENSE.txt  THIRD-PARTY-NOTICES.txt
LICENSE.electron.txt  LICENSES.chromium.html
resources.pak  chrome_100_percent.pak  chrome_200_percent.pak  icudtl.dat
snapshot_blob.bin  v8_context_snapshot.bin  vk_swiftshader_icd.json
d3dcompiler_47.dll  dxcompiler.dll  electron_wer.dll  ffmpeg.dll
vk_swiftshader.dll  vulkan-1.dll
locales\   resources\
```

`LICENSE.txt` is the app's own MIT text ("Copyright (c) 2026 Ganjineh"), the same text the repository
carries as `LICENSE`. `THIRD-PARTY-NOTICES.txt` is the plain-text copy of this repository's
`THIRD-PARTY-NOTICES.md`: it is what satisfies the attribution and GPLv3 source-offer obligations for
the files inside the zip, so a release is not finished without it.

**The three legal artefacts are mandatory, not optional.** Do not upload a release unless all three
are true:

1. `LICENSE.txt` sits in the zip root, next to `Ganjineh.exe`.
2. `THIRD-PARTY-NOTICES.txt` sits in the zip root and carries the MIT / Unlicense / OFL texts plus the
   FFmpeg GPLv3 source offer (build id, commit, written offer).
3. `README.txt` ends with a **`LICENSES AND SOURCE`** section that states the app's MIT license and
   reproduces the FFmpeg source offer, so the offer travels with every extracted copy even if the
   notices file is moved or lost.

`resources\` carries `app.asar`, `icon.ico`, `icon.png`, `fonts\`, `web\` and `bin\` (yt-dlp, Deno,
the ffmpeg shared build with its `LICENSE.txt`).

**Exclude — and know why:**

| Excluded | Reason |
|---|---|
| `GanjinehData\` | One person's settings, accounts, queue, library index and logs. Personal data, never published. |
| `_work\` | Maintainer tooling: editable sources, the extracted asar tree, screenshots, scratch output. |
| `*.before-*` (`app.asar.before-explore`, `app.asar.before-icon`, `app.asar.before-fix`, `app.asar.before-signin-*`, `Ganjineh.exe.before-version`, `Ganjineh.exe.before-icon`) | Rollback snapshots, several hundred MB each, meaningless to a user. |
| `*.new` (`Ganjineh.exe.new`) | Dry-run output of the exe patcher. |
| `_syntaxcheck.js`, `_syntaxcheck.css` | Scratch bundles from esbuild syntax checks. |
| `DevToolsActivePort`, `--remote-debugging-port` leftovers | Local debugging state. |
| `*.log`, `logs\` | Local logs only. |
| `_work\_tmp\`, `out\` | Staging and scratch folders. |
| `desktop.ini`, `Thumbs.db` | Windows shell droppings that do not belong in an archive. |
| `*.zip`, `SHA256SUMS.txt` | Release artifacts, created from the staged tree — never inside it. |

Verify the staged tree before zipping:

```powershell
$stage = '_work\release\dist\Ganjineh-1.1.0-win-x64'
(Get-ChildItem $stage -Recurse -File | Measure-Object Length -Sum) | Select-Object Count, Sum
Test-Path (Join-Path $stage 'GanjinehData')                                  # must be False
Test-Path (Join-Path $stage 'LICENSE.txt')                                   # must be True
Test-Path (Join-Path $stage 'THIRD-PARTY-NOTICES.txt')                       # must be True
Select-String -Path (Join-Path $stage 'README.txt') -Pattern 'LICENSES AND SOURCE' -SimpleMatch
Get-ChildItem $stage -Recurse -Force |
  Where-Object { $_.Name -match 'before-|\.new$|_syntaxcheck|\.log$|DevToolsActivePort|desktop\.ini|Thumbs\.db' } |
  Select-Object FullName                                                     # must be empty
(Get-Item (Join-Path $stage 'Ganjineh.exe')).VersionInfo.FileVersion         # must be 1.1.0.0
```

Then smoke-test the staged copy itself (this is the tree users will get, not your working folder):

```powershell
& .\_work\release\smoke.ps1 -Dir _work\release\dist\Ganjineh-1.1.0-win-x64 -Port 9444
```

It launches the copy with an isolated `GanjinehData\`, waits for the renderer, screenshots the first
run, quits, and reports whether user data was written beside the exe. Delete any `GanjinehData\` it
left behind before zipping.

## 5. Create the zip

```powershell
Set-Location .\_work\release\dist
Compress-Archive -Path .\Ganjineh-1.1.0-win-x64 -DestinationPath ..\out\releases\Ganjineh-1.1.0-win-x64.zip -CompressionLevel Optimal
```

The archive must contain the folder `Ganjineh-1.1.0-win-x64\` at its root, so that extracting gives
one tidy folder instead of loose files next to whatever else is in Downloads. Keep the working zip in
`_work\release\out\releases\`, outside the payload.

If `Compress-Archive` is too slow on 600 MB, the built-in bsdtar is faster and produces a smaller
archive:

```powershell
tar -a -c -f .\_work\release\out\releases\Ganjineh-1.1.0-win-x64.zip -C .\_work\release\dist Ganjineh-1.1.0-win-x64
```

Check the archive, not just the folder:

```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem
$zip = [System.IO.Compression.ZipFile]::OpenRead((Resolve-Path ..\out\releases\Ganjineh-1.1.0-win-x64.zip))
$zip.Entries.Count                                                        # 58 files + 6 folders
$zip.Entries | Where-Object { $_.FullName -match 'GanjinehData|before-|_work|_syntaxcheck|\.log$|\.new$' } |
  Select-Object FullName                                                  # must be empty
($zip.Entries | Where-Object { $_.FullName -eq 'Ganjineh-1.1.0-win-x64/Ganjineh.exe' }).Length
$zip.Dispose()
```

The `Ganjineh.exe` entry must be about 250,133,504 bytes. Extract one file from the archive and run it
if you want the strongest check.

## 6. Hash and manifest

```powershell
$zipPath = '.\_work\release\out\releases\Ganjineh-1.1.0-win-x64.zip'
$hash = (Get-FileHash $zipPath -Algorithm SHA256).Hash
"$hash  Ganjineh-1.1.0-win-x64.zip" | Set-Content '.\_work\release\out\releases\SHA256SUMS.txt' -Encoding ascii
Get-Content '.\_work\release\out\releases\SHA256SUMS.txt'
```

Two spaces between the hash and the filename, one line, nothing else — that is the format
`sha256sum -c SHA256SUMS.txt` expects (Git Bash, WSL, Linux, macOS), and it is readable by eye. Paste
the same hash into your notes if you will quote it in the release description.

## 7. Commit the docs repository

The published repository is `README.md`, `CHANGELOG.md`, `PRIVACY.md`, `THIRD-PARTY-NOTICES.md`,
`LICENSE`, `docs/RELEASING.md`, `.gitignore` and `assets/screenshots/*.png`. Nothing else.

```powershell
Set-Location .\Ganjineh-repo      # the folder that holds README.md, CHANGELOG.md, PRIVACY.md, …
git init
git add .
git commit -m "Ganjineh 1.1.0 — docs, screenshots and third-party notices"
git branch -M main
git remote add origin https://github.com/<your-github-user>/Ganjineh.git
git push -u origin main
```

Confirm the zip and the user-data folder stayed out of the commit:

```powershell
git status --porcelain
git check-ignore -v Ganjineh-1.1.0-win-x64.zip
git ls-files | Select-String -Pattern 'GanjinehData|\.zip$'    # must be empty
```

## 8. Tag

```powershell
git tag -a v1.1.0 -m "Ganjineh 1.1.0"
git push origin v1.1.0
```

The tag is `v1.1.0`; the app version is `1.1.0`; the asset name is `Ganjineh-1.1.0-win-x64.zip`. Use
these exact strings — the README, CHANGELOG and privacy page all point at them.

## 9. Publish the release

With the GitHub CLI (from the workspace root):

```powershell
Set-Location C:\Users\ASUS\Documents\Ganjineh-1.0.1-win-x64
gh release create v1.1.0 `
  ".\_work\release\out\releases\Ganjineh-1.1.0-win-x64.zip" `
  ".\_work\release\out\releases\SHA256SUMS.txt" `
  --title "Ganjineh 1.1.0" `
  --notes-file .\notes-1.1.0.md
```

Or in the browser: **Releases → Draft a new release → choose tag `v1.1.0` → title `Ganjineh 1.1.0` →
paste the notes → attach both files → Publish release.** Do not tick "pre-release" for a finished
build, and do not rename the assets during upload.

## 10. Verify what the world will download

Do this from a fresh download, not from your working copies:

```powershell
$fresh = "$env:USERPROFILE\Downloads\Ganjineh-1.1.0-win-x64.zip"
(Get-FileHash $fresh -Algorithm SHA256).Hash      # equals SHA256SUMS.txt and the value from step 6
Expand-Archive $fresh -DestinationPath "$env:TEMP\ganjineh-verify"
```

Run `Ganjineh.exe` from the extracted folder, walk the three first-run steps, confirm About shows
`1.1.0`, confirm a `GanjinehData\` folder appears beside the exe, then close and delete both the
extracted folder and its data folder. If the hash differs, do not edit the manifest — re-upload the
asset from step 5.

## Release-notes template

```markdown
Ganjineh 1.1.0 — a personal YouTube archive that behaves like a paper library.

Download `Ganjineh-1.1.0-win-x64.zip`, extract it anywhere, and run `Ganjineh.exe`.
Verify the download against `SHA256SUMS.txt` before you run it. Windows will show
"Windows protected your PC" once because the app is unsigned: More info → Run anyway.

Portable by design: settings, accounts and the queue live in `GanjinehData\` next to the exe.
Your videos live in your library folder as plain files. Deleting the app folder takes your data with
it, and Settings › About & logs › Remove from this PC… clears the Start menu entry and the
`ganjineh://` link as well.

## What is new in 1.1.0
- Explore is now the home of your main account: Feed, Playlists, Watch later, Liked and History,
  with a guest state that keeps search working when you have not added an account.
- Bulk selection everywhere a list gets long: drag, Shift-click, "pick №" with Persian digits,
  Ctrl+A / Ctrl+I, arrow keys, and a black action bar for Pause, Resume, Retry, Move to front,
  Copy links and Remove.
- The dark theme rebuilt from the ground up ("Ink"): a lamp-lit desk, warm paper sheets, and label
  tape that finally reads as a lit object. The Paper (light) theme is unchanged.
- A new postage-stamp app icon for the window, the tray and the executable.
- Loading skeletons, empty states, error cards with retry, and an honest stale-cache note.
- Every new string in English and Persian, right-to-left, with Persian digits.

Full details: CHANGELOG.md. Privacy: PRIVACY.md — everything stays on your PC.
Licenses: the app is MIT (`LICENSE.txt` in the zip); third-party components and the FFmpeg GPLv3
source offer are in `THIRD-PARTY-NOTICES.txt` next to the exe and in THIRD-PARTY-NOTICES.md here.

Windows 10/11 x64 · portable · unsigned · MIT licensed
```

## If something goes wrong

- **`app.asar` will not pack** — Ganjineh is still running and holding the file. Close it, including
  the tray icon, and build again.
- **`npx` wants to download something** — the toolchain must already be cached locally; this process
  has no network install step. Restore the cache instead of reaching for the network.
- **A `GanjinehData\` folder appears in the staged tree** — you smoke-tested in place and forgot to
  remove it. Delete it and rebuild the zip; never ship someone else's data folder.
- **The published hash does not match** — re-upload the asset from step 5 and regenerate
  `SHA256SUMS.txt` in the same run. Never hand-edit the manifest to match a bad upload.
- **The release is wrong after publishing** — delete the release (`gh release delete v1.1.0`), delete
  and re-push the tag (`git tag -d v1.1.0`, `git push origin :refs/tags/v1.1.0`), fix the cause, and
  start again at step 4. A tag that moves is worse than a release that is a day late.

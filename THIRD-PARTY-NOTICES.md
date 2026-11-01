# Third-party notices for Ganjineh 1.1.0 (Windows x64)

Ganjineh itself is licensed **MIT** (`resources/app.asar` → `package.json`: `"license": "MIT"`).
It is built on the components below. This file lists every third-party work that is
redistributed inside `Ganjineh-1.1.0-win-x64.zip`, with the exact version found in the
build, its license, and where the upstream source lives.

Versions were read from the shipped binaries themselves (PE version resources,
`LICENSES.chromium.html`, BtbN-style FFmpeg configure strings, TrueType `name` tables),
not from a manifest. Re-verify if the payload is rebuilt.

> **Obligations that apply to the release as a whole** are summarised at the end of this
> file (§3). Two of them are not yet satisfied by the current zip — they are marked ⚠.

---

## 1. Runtime and engines

| Component | Version | License | Upstream | Where it ships |
|---|---|---|---|---|
| Electron | 45.0.0 | MIT | https://github.com/electron/electron | `Ganjineh.exe` (runtime), `LICENSE.electron.txt` |
| Chromium (Electron's copy) | 156.0.8078.48 | BSD-3-Clause + third-party licenses | https://www.chromium.org/ ; notices in `LICENSES.chromium.html` | `Ganjineh.exe`, `resources.pak`, `*.pak`, `*.dll`, `icudtl.dat`, `snapshot_blob.bin` |
| Node.js (embedded in Electron) | 24.22.0 | MIT | https://nodejs.org/ | `Ganjineh.exe` |
| V8 (embedded in Chromium) | shipped with Chromium 156 | BSD-3-Clause | https://v8.dev/ | `Ganjineh.exe`, `v8_context_snapshot.bin` |
| Chromium third-party libraries (Skia, SQLite, ICU, BoringSSL, HarfBuzz, zlib, …) | as of Chromium 156 | per-entry licenses | enumerated in `LICENSES.chromium.html` (5.7 MB, ships in the zip) | `Ganjineh.exe`, `*.dll`, `resources.pak` |

Notes
- `LICENSES.chromium.html` and `LICENSE.electron.txt` **must stay in the zip**; they are the
  complete attribution set for the Electron/Chromium runtime and are the only place the
  hundreds of Chromium dependencies are documented.
- Chromium is version 156.0.8078.48. Chromium ships security fixes continuously; an
  evergreen public release should plan an Electron upgrade cadence (Chromium 156 branches
  are superseded quickly).

## 2. Bundled engines and assets

| Component | Version | License | Upstream | Where it ships |
|---|---|---|---|---|
| yt-dlp | 2026.10.21 (built on Python 3.10.11, PyInstaller) | Unlicense | https://github.com/yt-dlp/yt-dlp | `resources/bin/yt-dlp.exe` |
| Deno | 2.9.8 | MIT | https://github.com/denoland/deno | `resources/bin/deno.exe` |
| FFmpeg (shared build) | `N-127911-gadfe258a6a-20261021` (libavcodec 63.2.100, libavformat 63.7.1, libavutil 61.3.1, libavfilter 12.x, libswscale 10.x, libswresample 7.x) | **GPL-3.0-or-later** (build configured `--enable-gpl --enable-version3`) | https://ffmpeg.org/ — source for this exact build: git commit `adfe258a6a`; the mingw cross-build layout (`--prefix=/ffbuild/prefix`, `--cross-prefix=x86_64-w64-mingw32-`) matches https://github.com/BtbN/FFmpeg-Builds | `resources/bin/ffmpeg/{ffmpeg.exe, ffprobe.exe, avcodec-63.dll, avformat-63.dll, avfilter-12.dll, avdevice-63.dll, avutil-61.dll, swresample-7.dll, swscale-10.dll}` + `LICENSE.txt` |
| React / React DOM | 19.3.1 | MIT | https://github.com/facebook/react | inside `resources/app.asar` → `dist/assets/index-*.js` |
| qrcode (node-qrcode, browser build) | bundled, version string not embedded | MIT | https://github.com/soldair/node-qrcode | inside `resources/app.asar` → `dist/assets/index-*.js` (the "Home Wi‑Fi" / LAN QR code) |

Notes
- **FFmpeg is GPLv3, not LGPL.** `--enable-gpl --enable-version3` makes the whole
  `av*` DLL set GPLv3-or-later. GPLv3 §6 requires that everyone who receives the binary
  can obtain the **complete corresponding source of that exact build**, including the
  build/configure information. Since these DLLs are linked into a GPLv3 work, the
  combination must be distributed under GPLv3, and the app must not add license terms
  that forbid the recipient from exercising GPLv3 rights on those libraries.
- The FFmpeg build statically links its own dependencies (configure line shows
  `--pkg-config-flags=--static` plus `libdav1d`, `librav1e`, `libvorbis`, `libharfbuzz`,
  `libfreetype`, `libfribidi`, `liblcms2`, `librsvg`, `libvmaf`, `libxml2`, `libgmp`,
  `liblzma`, `liblcevc`, `fontconfig`, `zlib`). GPLv3 already dominates those licenses;
  the build's own `LICENSE.txt` (GPLv3 text) is shipped next to the binaries.
- yt-dlp is Unlicense (public domain); no attribution is *required*, but the version is
  reported to users in Settings and is useful for support.
- The app does not ship Python or a separate Node runtime: `yt-dlp.exe` is a PyInstaller
  bundle (hence the Python 3.10.11 version suffix) and Node comes from Electron.

### 2.1 Fonts (`resources/fonts/*.ttf`, and duplicates inside `app.asar`)

All nine families are licensed under the **SIL Open Font License 1.1**, whose §2 requires
that every copy carries the copyright notice **and** the license text. `resources/fonts/OFL.txt`
ships in the zip and contains the OFL 1.1 text plus the copyright lines — but it currently
lists only **seven** of the nine families (see ⚠ in §3).

| Family | Version (name table) | License | Copyright | Upstream |
|---|---|---|---|---|
| Vazirmatn (Regular/Medium/SemiBold/Bold) | 33.003 | OFL 1.1 | 2015 The Vazirmatn Project Authors | https://github.com/rastikerdar/vazirmatn |
| Rubik (Medium/Bold) | 2.300 | OFL 1.1 | 2015 The Rubik Project Authors | https://github.com/googlefonts/rubik |
| Lalezar | 1.004 | OFL 1.1 | 2015 The Lalezar Project Authors | https://github.com/BornaIz/Lalezar |
| Space Grotesk (Regular/Medium/Bold) | 2.000 | OFL 1.1 | 2020 The Space Grotesk Project Authors | https://github.com/floriankarsten/space-grotesk |
| Courier Prime (Regular/Bold) | 3.018 | OFL 1.1 | 2015 The Courier Prime Project Authors | https://github.com/quoteunquoteapps/CourierPrime |
| Young Serif | 3.003 | OFL 1.1 | 2023 The Young Serif Project Authors | https://github.com/noirblancrouge/YoungSerif |
| IBM Plex Mono (Regular/Medium/SemiBold) | 2.3 | OFL 1.1 | 2017 IBM Corp. | https://github.com/IBM/plex |
| Source Serif 4 (Roman/Italic) | 4.004 | OFL 1.1 | 2014–2021 Adobe Systems Inc. (Reserved Font Name "Source") | https://github.com/adobe-fonts/source-serif |
| Nanum Pen | 1.10 | OFL 1.1 | 2010 NHN Corporation, designed by Sandoll Communications | https://github.com/google/fonts/tree/main/ofl/nanumpen |

No font file was modified (no subsetting/renaming detected in the `name` table), so OFL §3
(Reserved Font Name) is not triggered.

## 3. Obligations this release must satisfy

| # | Obligation | Status in the current payload |
|---|---|---|
| 1 | Ship `LICENSE.electron.txt` + `LICENSES.chromium.html` with the binaries | **OK** — both in the zip root |
| 2 | Ship the FFmpeg `LICENSE.txt` (GPLv3 text) next to the FFmpeg binaries | **OK** — `resources/bin/ffmpeg/LICENSE.txt` |
| 3 | Offer **complete corresponding source** for FFmpeg `N-127911-gadfe258a6a` (GPLv3 §6): publish the commit hash, the configure line above, and a durable source location; state the offer in the release notes or ship a copy of that declaration in the zip | ⚠ **NOT DONE** — no source offer exists in the repo or zip |
| 4 | Carry the MIT notices for Electron / Node / React / qrcode (MIT requires the copyright + permission text in the distribution) | ⚠ **partly** — Electron's text ships; React and qrcode (both inside `app.asar`) have no notice in the zip |
| 5 | Carry the app's own MIT license text (`package.json` says MIT) with the distributed build | ⚠ **NOT DONE** — no `LICENSE`/`LICENSE.txt` in the zip and none in the repo yet |
| 6 | Keep the OFL copyright + license text with the fonts, covering **every** bundled family | ⚠ **partly** — `OFL.txt` covers 7 of 9 families; **IBM Plex Mono** and **Source Serif 4** are missing |
| 7 | Do not impose extra restrictions on the GPLv3 FFmpeg libraries | **OK** — no EULA, no DRM/anti-tivoization terms in the app or in `README.txt` |

**Cheapest way to close 3–6:** put this file and a generated `THIRD-PARTY-NOTICES.txt`
(the same content, plus the MIT/Unlicense/OFL texts appended) in the zip root, add
`LICENSE.txt` (MIT, Ganjineh) next to `README.txt`, add the two missing copyright lines to
`resources/fonts/OFL.txt`, and add a "Licenses & source" section to `README.txt` and to the
GitHub Release notes that carries the FFmpeg source offer:

```
FFmpeg N-127911-gadfe258a6a (2026-10-21), GPLv3, built with:
  --enable-gpl --enable-version3 --enable-shared --disable-static
  --cross-prefix=x86_64-w64-mingw32- --arch=x86_64 --target-os=mingw32
  (full line: see below)
Source: https://git.ffmpeg.org/ffmpeg.git at commit adfe258a6a
        or the build recipe at https://github.com/BtbN/FFmpeg-Builds
Written offer: the maintainer will supply the corresponding source for this build on request.
```

Full configure line recovered from `avutil-61.dll`:

```
--prefix=/ffbuild/prefix --pkg-config-flags=--static --pkg-config=pkg-config
--cross-prefix=x86_64-w64-mingw32- --arch=x86_64 --target-os=mingw32 --enable-gpl
--enable-version3 --disable-debug --enable-shared --disable-static --disable-w32threads
--enable-pthreads --enable-iconv --enable-zlib --enable-libxml2 --enable-libvmaf
--disable-libxcb --disable-xlib --enable-fontconfig --enable-libharfbuzz
--enable-libfreetype --enable-libfribidi --enable-lcms2 --enable-vulkan --enable-libdav1d
--enable-libvorbis --enable-librav1e --enable-librsvg --disable-libpulse --enable-gmp
--enable-lzma --enable-liblcevc
```

---

### How this list was produced

- `Ganjineh.exe` PE version resource → `FileVersion 1.0.0` (stale, see AUDIT.md), and the
  embedded strings `Electron/45.0.0`, `node.js/v24.22.0`, `Chrome/156.0.8078.48`.
- `LICENSE.electron.txt` (MIT, Electron contributors), `LICENSES.chromium.html`.
- `Get-Item .VersionInfo` on `resources/bin/*.exe` → yt-dlp 2026.10.21, Deno 2.9.8.
- FFmpeg: version string + configure line extracted from `resources/bin/ffmpeg/avutil-61.dll`;
  `ffmpeg.exe`/`ffprobe.exe` carry no PE version resource.
- Fonts: TrueType `name` table (IDs 0/1/5/13/14) read from each `.ttf`.
- JS libraries: markers inside `resources/app.asar` (`react.dev/errors`, `version:"19.3.1"`,
  `rendererPackageName:"react-dom"`, `toDataURL(...,{margin,width,color})` qrcode API).
  No other third-party JS library was identified; the SQLite layer uses Node 24's built-in
  `node:sqlite`, and Chromium's own WebSocket/HTTP implementations are used instead of
  `ws`/`node-fetch`.

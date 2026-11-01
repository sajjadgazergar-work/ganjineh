# Changelog

All notable changes to Ganjineh are recorded in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow
[Semantic Versioning](https://semver.org/spec/v2.0.0.html). Dates are plain `YYYY-MM-DD`.
Ganjineh is a portable Windows app: every version is a zip you extract, not an installer you run.

## [1.1.0] - 2026-10-30

The release that turned Ganjineh into a room. Explore became the home of your account, the dark
theme was drawn again from the ground up, and every long list learned how to be picked.

### Added

- **Explore is the home of your main account.** The tab that used to be a bare YouTube search box
  now opens with an account card — real display name, `@handle`, your YouTube avatar, a "reads your
  lists" chip and quick jumps to Feed, Playlists, Watch Later and Liked — and six tabs: Search,
  Feed, Playlists, Watch later, Liked, History.
  - **Feed** with Subscriptions and Home sub-tabs, 40 rows at a time, `Queue all`, and a per-row
    action for Queue / On shelf / in queue / open on YouTube.
  - **Playlists** as a grid of cards with title, owner and kind, opening into an inline playlist
    view with `Queue all`, `Open on YouTube` and Refresh.
  - **Watch later** and **Liked** read straight from your account, with the honest note "showing the
    first 60 of N videos" when the real list is longer.
  - **History** from your YouTube watch history.
  - A **guest state** when no main account is added: a sheet inviting you to bring your account in,
    Search still fully usable, account tabs hidden. Nothing is downloaded or played without you asking.
- **Bulk selection everywhere a list gets long.** Hold the mouse down on a checkbox and drag to tick
  a run; Shift-click another checkbox to take everything in between; `pick №` takes typed numbers
  (`1-10, 15` or `۱ تا ۱۰، ۱۵` — Persian digits work); `Ctrl+A`, `Ctrl+Shift+A`, `Ctrl+I`, `Esc`,
  `Delete`; arrow keys move, `Shift`+arrows extend, `Space` ticks, `Enter` opens. The black action
  bar then works on the whole selection: Pause, Resume, Retry, Move to front, Copy links, Remove.
  `Select all 100` ticks every row in the current tab.
- **Selection in the pre-add review sheet.** Every channel or playlist gets its own All / None /
  Invert and `pick №`, and `Skip Shorts` and `Hide ones I already have` hide rows — `pick №` skips
  hidden rows and tells you how many it skipped.
- **Selection in the library.** Press `Select` (or Ctrl-click a video), then click, drag across or
  Shift-click a range: Mark watched, Mark unwatched, Copy links or Delete for the whole selection.
- **New app icon: a postage stamp.** Perforated paper edge, teal field, a gold eight-point Persian
  star and the Lalezar letter **گ**. Drawn as vector at three detail tiers (16–40 px seal, 48–96 px
  stamp, 128–512 px full) and applied in four places: window and taskbar, tray fallback and
  notifications, the executable's own icon resources, and a 10-size `.ico` the window prefers.
- **Loading, empty and error states you can trust.** Explore shows skeleton rows instead of blank
  screens, a real empty state per list, error cards with `Try again`, a "this is cached" note when a
  refresh fails, and an offline warning — cached content stays readable, and search is never blocked.
- **Bilingual copy for every new string.** All of 1.1 is English and Persian, right-to-left, with
  Persian digits and Vazirmatn fallbacks.

### Changed

- **The dark theme was rebuilt, not repainted.** "Ink" is now the same desk at night under one warm
  lamp: a near-black desk, warm paper sheets on a five-step luminance ladder, a 1 px paper rim and an
  inset top highlight on every sheet, deeper and wider shadows, and softer grain. The label-maker
  tape — navigation, primary buttons, toasts, the bulk bar, presets — is inverted to light paper with
  dark print, so it reads as a lit physical object instead of vanishing into the desk.
  Desk-to-sheet separation went from 1.3× to 3.0× luminance, and the worst text pairs on every screen
  were fixed: on Downloads, 7 pairs below 4.5:1 became 0, and the minimum contrast rose from 3.30:1
  to 5.14:1.
- **Paper, the light theme, is kept as it was** — deliberately, verified pixel-identical: every new
  dark rule is scoped to the dark theme, so nothing shifted in daylight.
- **Explore changed role.** It was a search box; it is now the account home, with search as its first
  tab and unchanged behaviour when you are signed out.
- **Lists read as volumes.** A playlist or channel in Downloads is a numbered volume with rows, so
  the same selection keys, the same black bar and the same `pick №` grammar work on downloads, on
  review sheets and in the library.
- **Version metadata agrees everywhere.** The executable's file and product version now report
  `1.1.0`, so Explorer, Task Manager and the About panel tell the same story as the app.

### Fixed

- Dark-theme text that failed contrast — 5 to 15 failing pairs per screen before (worst pair
  2.84:1), none after; key pairs now measure ink on sheet 14.4:1, secondary ink 9.3:1, muted ink
  5.7:1 (was about 4.2:1) and dark print on label tape 8.0–13.7:1.
- Label tape and selection tape disappearing on dark, because they had been painted black on black.
- Bright paper props glaring on a dark desk: the greeting card, price and "new" stamps, the polaroid
  frame (which had ended up darker than the desk it sat on) and the logo stamp paper were re-graded
  for lamp light.
- Scrollbars, focus rings and set-section backgrounds that still assumed the light theme.
- The window frame flashed a lighter chrome at cold start, before the page painted; the titlebar
  colours now match the Ink theme from the first frame.
- Grain that read as dirt on dark surfaces (alpha 0.15 → 0.10).

## [1.0.0]

The baseline this changelog starts from: the portable Windows build that Ganjineh was first used
with, before the project began keeping dated release entries. It is recorded here so the public
history has a floor to stand on.

### Added

- Portable Windows 10/11 x64 app: no installer and no admin rights. `portable.txt` next to the exe
  keeps settings, accounts and the queue in `GanjinehData\` beside it; delete that file and Ganjineh
  uses your Windows user profile instead. The only things it adds outside its own folder are a Start
  menu shortcut and the `ganjineh://` link, both under your user account.
- Downloads desk: paste a video, a playlist, a whole channel link or a `.txt` of links. A queue with
  pause, resume and retry that survives VPN drops, outages and power cuts, and never downloads the
  same video twice.
- Presets for video height, plus audio-only (M4A, Opus or MP3), subtitles when they exist, chapters,
  and SponsorBlock marks the player skips.
- Library shelf: plain files at `Videos\Ganjineh\<Channel>\<Title> [id].mp4` with subtitles and a
  small sidecar record beside each one — copy them anywhere and they still play. Watched marks,
  filters, and transcripts you can search.
- Channels you follow, a Music tab with playback, and a Transcripts tab for every spoken word in your
  library.
- `Ctrl+K` to jump anywhere and search inside transcripts; `Ctrl+Shift+Y` to grab links you copy in
  any app; a browser button and `ganjineh://add?url=…` links for sending pages to Ganjineh.
- Optional accounts: a main account that only reads your lists (Watch Later, Liked, playlists) and a
  spare account for heavy downloading. Cookies are encrypted with Windows DPAPI and stay on the PC.
- Bilingual interface — English and Persian, RTL, Persian digits, optional Jalali dates — and the
  Paper (light) look.
- Bundled engines under `resources\bin\` (yt-dlp, Deno, an ffmpeg shared build), so the app works
  without installing anything else, plus a Settings › Download engine panel that can update yt-dlp.
- Settings for proxy (follow Windows/VPN, or a custom local port), speed limit, keeping the PC awake,
  and what to do when the queue finishes (sleep or shut down with a warning).
- MIT license for Ganjineh; the bundled engines and typefaces keep their own upstream licenses.

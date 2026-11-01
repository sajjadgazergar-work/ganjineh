# Ganjineh keeps its data where you can see it

Ganjineh is a portable Windows app that archives YouTube videos for one person. It has no accounts
server, no cloud sync, no telemetry and no ads. Everything it knows lives in a folder next to
`Ganjineh.exe`, and you can delete that folder at any time.

**The short version**

- Settings, the queue, the library index, your accounts and the logs are files on your PC. They are
  never uploaded anywhere.
- Ganjineh talks to YouTube because that is where the videos are, and to GitHub because that is where
  the download engine comes from. Nothing else, unless you switch it on.
- The only thing that ever holds your YouTube session is an encrypted file on your disk, and it is
  only ever sent back to YouTube.
- There is no developer dashboard to look at you from. There is nobody collecting anything.

## What is stored on your PC

In portable mode (a `portable.txt` file sits next to `Ganjineh.exe`) everything below is inside
`GanjinehData\`, beside the app:

| Path | What it holds |
|---|---|
| `data\ganjineh.db` | Your library index, the download queue, watched marks, follow rules, settings. |
| `data\accounts\*.cookies.bin` | One encrypted file per saved YouTube account. Encrypted with Windows DPAPI, so only your Windows user on this PC can read it. |
| `data\engine\yt-dlp\<version>\` | Any download engine version you fetched from GitHub, so an update never breaks a working setup. |
| `data\logs\ganjineh.log`, `data\logs\signin.log` | Plain-text logs of what the app did and what went wrong. They stay on this PC. |
| `Cache\`, `Code Cache\`, `GPUCache\`, `Local Storage\`, `Session Storage\`, `ShaderCache\` | Ordinary Electron/Chromium caches and local storage for the app window. |
| `Network\Cookies` | Chromium's cookie store for the app window's own session — the app window is a browser, so it keeps one, the same way any Chromium app does. |

Ganjineh decides this at every start, by looking for `portable.txt` next to `Ganjineh.exe`. Delete it
before the first run and the same files go to your Windows user profile (`%APPDATA%\Ganjineh`)
instead. Delete it after Ganjineh has already run and the app starts up looking in your profile, as
if it were new — nothing is deleted, the old `GanjinehData\` folder simply sits there unused until you
put the file back.

Your videos are not in that folder at all. They are ordinary files in your library folder, by default
`Videos\Ganjineh\<Channel>\<Title> [id].mp4`, with subtitles and a small sidecar record beside each
one. Uninstalling Ganjineh never touches them.

Only two things live outside that folder, both under your own user account: a Start menu shortcut
(`Ganjineh.lnk`) and the `ganjineh://` protocol registration, so that links can hand a URL to the app.
Both are removed by Settings › About & logs › Remove from this PC…

## What leaves your PC, and why

Ganjineh makes network requests only to fetch what you asked for or to keep its engine working. The
list below is the complete set of destinations the app is built to contact:

| Destination | When | Why |
|---|---|---|
| `www.youtube.com`, `youtube.com`, `i.ytimg.com` | Whenever you browse, search, open Explore, queue or download | Video metadata, thumbnails, the pages a download needs, and your own lists if you added an account |
| YouTube's media hosts (for example `googlevideo.com`) | While a download runs or a video plays | The actual audio and video bytes, exactly as any YouTube client fetches them |
| `accounts.google.com` | Only if you choose to add an account by signing in | Google's own sign-in page, in a separate browser window that Ganjineh opens for the sign-in |
| `music.youtube.com` | When you open the Music tab and it needs fresh metadata | YouTube Music pages for browsing |
| `api.github.com`, `github.com` | At most once every 72 hours when engine auto-update is on, and whenever you press Update | Checks the latest yt-dlp release version, then downloads it |
| The public SponsorBlock API (through the bundled yt-dlp) | While a download runs with sponsor marking on — which is the default in the shipped build; turn it off in Settings › Downloads | Asking which segments of a video are sponsor segments |
| `www.cloudflare.com` (`/cdn-cgi/trace`) and `www.aparat.com` | Only when YouTube is unreachable, to tell "your VPN is down" apart from "the internet is down" | A reachability probe; no data of yours is sent, it is a plain connect test |
| `127.0.0.1` on a random port | Always while the app runs | The internal server that plays files from your own library to the app window |

Notes on that table:

- **Proxy and VPN.** Every request above goes through the connection you selected in Settings ›
  Network — "Use Windows settings" (which follows your VPN app), a custom local proxy, or a direct
  connection. A VPN sees Ganjineh's traffic the same way it sees your browser's.
- **Queries carry no identity of ours.** There is no account ID, install ID, device fingerprint or
  analytics parameter added to any request. Ganjineh does not append anything to identify you to the
  destinations above.
- **YouTube sees what a YouTube client sees.** If you sign in with an account, YouTube knows that
  session is asking for those pages and videos. That is how YouTube works; it is not something
  Ganjineh forwards anywhere else.

## The engines that ship with the app

yt-dlp (the download engine), Deno (the JavaScript runtime some extractors need) and an ffmpeg shared
build (merging and converting) are bundled inside the zip under `resources\bin\`, so Ganjineh needs no
installs and runs the same way on every PC. Because YouTube changes, the engine can also be replaced:

- Settings › Download engine can fetch a newer yt-dlp from GitHub, keep several versions side by side
  in `data\engine\yt-dlp\`, and let you switch back.
- With engine auto-update on (the default), Ganjineh asks `api.github.com` for the latest release at
  most once every 72 hours and downloads it from `github.com` if it is genuinely newer. Nothing about
  you is sent — it is a version number request, like checking a changelog.
- Both can be switched off: the update check in Settings › Download engine, sponsor marking in
  Settings › Downloads. With sponsor marking on — it is **on in the shipped build** — each download
  asks the public SponsorBlock API for that video's public segment data, which makes SponsorBlock the
  one third-party service Ganjineh contacts by default. Turn it off and no SponsorBlock request is
  ever made.

SponsorBlock is a public, community-run segment database; the request contains the video id and
nothing about you. You can read more at <https://sponsor.ajay.app/>.

## Your accounts and cookies

Adding an account is optional. Without one, Ganjineh downloads as a guest.

- A **main account** is only used to read your lists — Watch Later, Liked, playlists, and the Explore
  tabs. It is never used for bulk downloading.
- A **spare account** (one you do not mind losing) can do the heavy downloading.
- **Signing in never uses your everyday browser profile.** Ganjineh starts a separate browser window
  with a throwaway profile in `%TEMP%`: Firefox first, because Google's sign-in accepts it, and if you
  have no Firefox then Chrome, Edge, Brave or Chromium, watched over a debugging port that only accepts
  connections from this machine. When you finish, the window closes, the temporary profile is deleted,
  and what remains is one DPAPI-encrypted cookie file per account in `data\accounts\`. Ganjineh never
  reads your browsing history. If Google rejects the window in every browser it can find, Ganjineh says
  so and points you at the `cookies.txt` route instead of pretending it worked.
- You can also add an account by importing from a Firefox profile on the same PC, or by importing a
  `cookies.txt` file. However it arrives, Ganjineh re-encrypts the cookies with Windows DPAPI and
  writes them to `data\accounts\*.cookies.bin`.
- When the download engine needs cookies, Ganjineh decrypts them into a temporary file that only that
  child process reads, and removes it when the run ends. The cookies are sent to YouTube and to
  nobody else. They are not written to logs, not put in the library database, and not included in any
  request to GitHub or SponsorBlock.
- Removing an account in Settings › Accounts deletes its file. Deleting `GanjinehData\` deletes all
  of them.

## LAN sharing is off unless you turn it on

Settings › Home Wi-Fi can serve your library to your own devices on the same network — a phone, a TV,
another PC — so you can watch what you archived from the sofa. It is honest about what that means:

- It is **off by default**. While it is off, nothing is reachable from the network.
- When you turn it on, the server listens on **all network interfaces** (`0.0.0.0`) on port `8723`, not
  just on this PC. That is the point — other devices have to be able to reach it.
- Access is protected by the **PIN you set**. Anyone on your network who has that PIN can browse and
  read the items you share — the same trust model as a shared folder. Keep the PIN to people you would
  hand the folder to, and do not reuse a PIN you use elsewhere.
- Because the default PIN is empty, **set a PIN before you enable it**; an empty PIN accepts an empty
  entry. Turn LAN sharing off again when you are done, and leave it off on untrusted networks —
  hotels, cafés, shared flats, anything you do not control.
- It does not phone anywhere or report who connected; it only answers requests that arrive on port
  `8723` while the setting is on. Nothing is uploaded to anyone.

## Logs

`GanjinehData\data\logs\` holds `ganjineh.log` and `signin.log`. They record what the app did — which
steps ran, which downloads failed, which errors came back — so that a problem can be understood
later. `signin.log` records which sign-in route was tried (Firefox, a Chromium browser, or none) and
whether it worked; it does not contain the cookies themselves. Both files are plain text on your disk,
they are never uploaded, and Settings › About & logs › Open logs folder takes you straight to them.
Delete the folder (or the log files) whenever you like.

## Deleting everything

- **Settings › About & logs › Remove from this PC…** — removes the Start menu shortcut and the
  `ganjineh://` registration, then quits. It does not touch your files: after it, the only thing left
  is the folder itself.
- **Manual:** close Ganjineh, delete the app folder. That removes the program, `GanjinehData\` and
  every trace of settings, accounts and logs.
- **Your videos stay.** They are plain files in your library folder. If you want them gone too, delete
  that folder yourself — Ganjineh will not do it for you, and never has.

## Children, and the legal bit

Ganjineh has no user accounts of its own, no chat, no comments, no user-generated content, no
advertising and no data collection — so there is nothing for a child to share with anyone through it,
and nothing for us to collect from anyone. It is a tool for keeping videos you have the right to
keep. Respect YouTube's terms and the rights of the people who made the videos you archive.

## Questions

Open an issue in this repository. There is no Ganjineh server to query, and no analytics page to
look at — the honest answer to "what do you know about me?" is nothing at all.

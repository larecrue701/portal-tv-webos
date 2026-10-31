# Portal TV for webOS

An independent TV player for **your own Stalker portal and M3U sources**, with a French interface designed for the LG remote. No channels, subscription, portal credentials or playlists are included.

**Version 0.3.5 — early release.** Tested on webOS 25 with Node.js 16.20.2. Other TVs and provider implementations need testing. This is not an official LG, Infomir or StbEmu application.

Download the [TV installation package](org.portaltv.player_0.3.5_all.ipk) or the [complete source archive](portal-tv-webos-0.3.5-source.zip). Extract the source archive before running the build commands below.

## Features

- Multiple saved portals (URL + registered MAC) and M3U URLs/files.
- Temporary local upload page, QR code and five-character access code to add sources from a computer or phone.
- Live TV and film/VOD catalogues, search and bouquet selection.
- Continuous catalogue scrolling: more entries load as you scroll.
- Per-source favorites, last live channel and VOD resume positions.
- Optional startup connection to a chosen saved source.
- Native LG video playback, pause and seeking where supported by the stream.
- Offline demo with fictional titles, without playable content.

## Remote controls

| Context | Key | Action |
| --- | --- | --- |
| Live channel list | OK | Play in preview; OK again on the same channel enters fullscreen |
| Fullscreen live TV, video focused | OK | Return to channel list |
| Fullscreen film, video focused | OK | Pause/resume |
| Fullscreen | Arrow | Focus player controls; navigate with arrows and activate with OK |
| Fullscreen | Back | Return to catalogue; pause a film and save its position |
| Live TV | Programme + / − | Next/previous channel in the displayed list |
| Film sheet | Read / Resume / Restart | Start the selected film |
| Text field | OK | Enter editing; OK or Back finishes editing |

LG Back uses key code **461**. Pointer controls are also supported.

## Installation and compatibility

The package includes a web app and a native JavaScript service. The service uses the TV's `/usr/lib/nodejs/palmbus` module. **Deployment has been tested on a rooted webOS TV with local app installation enabled; Developer Mode-only installation and older webOS versions are not yet verified.** This project does not root, flash or alter the TV's firmware.

Use your existing webOS app installation tool to install the IPK linked above, or build it below. Install updates over the existing app ID `org.portaltv.player`; do not uninstall first if you want to preserve sources.

With LG's CLI configured for your own device:

```sh
# Replace MY_TV with your configured device name
npx ares-install --device MY_TV dist/org.portaltv.player_0.3.5_all.ipk
npx ares-launch --device MY_TV org.portaltv.player
```

See [LG's CLI documentation](https://webostv.developer.lge.com/develop/tools/cli-introduction) for device setup. No SSH credentials are included in this repository.

## First use

1. Add a source you are authorized to use, or select **Ajouter depuis le Mac** to open a temporary local upload page. The TV and sending device must be on the same trusted LAN.
2. Save the source and connect. Choose **Mes bouquets** to limit the displayed categories.
3. Add the playing channel or a film to **★ Favoris** using its favorite button.
4. For automatic startup, return to **Mes sources**, select a saved source and choose **Ouvrir cette source au démarrage**. **Démarrer sur le choix des sources** turns this off.

Startup opens the catalogue, not a stream automatically. If connection fails, the source chooser remains available.

## Build and test

Development requires Node.js 22 or newer and npm. Runtime compatibility on the TV is separate from the desktop build environment.

```sh
npm ci
npm test
npm run package
# Optional: regenerate bundled icons
npm run build:icons
# Browser tests use fictional sources and a separate headless Chrome profile
npm run test:ui
# Set CHROME_PATH if Chrome is not at the default platform location
```

`npm run package` creates the IPK in `dist/`. `npm test` covers protocol parsing, transport restrictions, upload permissions and quotas. Browser tests cover remote navigation, favorites, playback simulation, resume and startup selection. Simulated playback does not establish compatibility with a real provider.

## Limits and privacy

- Films only: no series/season browser, EPG or catch-up yet.
- M3U entries are presented as a playlist, not automatically classified as films.
- Native LG codecs and the provider determine playback and seek support. No transcoder, DRM bypass or stream-header proxy.
- Up to 20 sources, 10,000 M3U entries, 500 groups, 200 favorites per source and 200 saved film positions per source. M3U imports are limited to 2 MiB.
- Public IPv4 HTTP(S) sources only. Local/private stream destinations are rejected. Some portal redirects, compression and authentication schemes are unsupported.
- Sources, provider commands, favorites and resume data are stored locally **without encryption**. Source URLs may contain credentials. Session tokens and resolved stream URLs are not added to persistent library storage.
- Imported files are kept in a private app data directory. Deleting a file source removes its unreferenced playlist file.
- The upload page uses plain HTTP on the trusted LAN. Its code expires after five minutes; five incorrect attempts lock the session. Closing the upload dialog or hiding the app revokes it. No permanent upload listener, analytics or external QR service.
- Never include real source URLs, MAC registrations, passwords, tokens or playlists in GitHub issues or screenshots.

## Rollback

Keep the previous IPK before updating. To roll back, reinstall that IPK over the same app ID using your installation tool. Do not uninstall first. Version 0.3.5 adds separate local preference keys that older versions ignore; it does not migrate or erase existing sources. Uninstalling the application may delete its local data.

## License and credits

MIT; see [LICENSE](LICENSE). The bundled QR generator is copyright Kazuhiko Arase and retains its [MIT notice](app/QR-LICENSE.txt). Application and protocol implementation are independent; no code from an unlicensed reference project is included.

### Build-tool dependency note

The pinned LG CLI includes third-party desktop tooling for which npm reports
security advisories. These dependencies are not bundled in the TV IPK: the
package contains only `app/` and `service/`. Use the CLI with trusted local
projects and review upstream LG CLI updates. No claim of a complete security
audit is made for this early release.

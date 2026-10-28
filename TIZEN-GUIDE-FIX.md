# Unofficial Samsung Tizen guide scrolling fix

This community branch fixes guide navigation with a Samsung TV remote. It is
not an official Jellyfin release and has not been merged upstream.

- Keeps the focused programme in view when Tizen moves focus without a guide
  command, including after horizontal navigation.
- Prevents Up/Down navigation from unexpectedly shifting the guide's time axis.
- Includes regression tests for focus visibility and horizontal scroll stability.

## Source and compatibility

Repository: <https://github.com/d-tep/jellyfin-web>

Branch: `fix/tizen-guide-focus-scroll`

The two fix commits are `db7d926b8cd871f523139a6fd65f2767dc2271eb` and
`79d0785c5418094620d441015bb1a61c5c9590ae`, based on upstream commit
`ee8efb417e5377ba444509a68ed22af33e45950b`. Only the guide implementation,
template, and regression tests are changed by those commits.

This checkout's package version is **13.0.0**, a development snapshot. It is
not a Web 12.2 release build. Check compatibility with your Jellyfin server and
TV before replacing your current app. The broader Web 12.2-based community TV
project is separate: <https://github.com/d-tep/jellyfin-tizen-guide-fix>.

## Clone, test, and build

Use Node.js 24 and npm 11 or later. These commands use a POSIX shell:

```sh
git clone --single-branch --branch fix/tizen-guide-focus-scroll https://github.com/d-tep/jellyfin-web.git
cd jellyfin-web
npm ci --no-audit
npm test -- src/components/guide/guide.test.ts
npm run build:check
npm run build:production
cd ..
```

The compiled web client is in `jellyfin-web/dist/`. Pulling this repository
alone does not install or update the app on a TV.

## Package for a Samsung TV

Install Tizen Studio's TV tooling and create your own valid Samsung TV signing
profile. The wrapper source revision below is pinned so the packaging commands
refer to a specific source version:

```sh
git clone https://github.com/jellyfin/jellyfin-tizen.git
cd jellyfin-tizen
git checkout --detach 31408c121e34e73c9f50c4e201c05e95140675df
JELLYFIN_WEB_DIR=../jellyfin-web/dist npm ci --no-audit
tizen build-web -e ".*" -e "node_modules/*" -e "package*.json" -e gulpfile.babel.js -e LICENSE
tizen package -t wgt -s YOUR_PROFILE -o . -- .buildResult
```

Replace `YOUR_PROFILE` with your signing profile. Enable Developer Mode on the
TV, set its Host PC address to the build machine, then connect and install:

```sh
sdb connect TV_IP
tizen list target
tizen install -n Jellyfin.wgt -t YOUR_TV_TARGET -- .
```

Replace `TV_IP` and `YOUR_TV_TARGET` with your TV's address and target name.
Updating an existing installation requires a compatible author signing
certificate. Keep your current signed package and certificate available for
rollback. No signing keys, accounts, server settings, or prebuilt `.wgt` are
provided by this source branch.

## Check on the TV

Open Live TV, enter the guide, and use Up/Down to move beyond the first screen.
The focused programme should stay visible without moving the selected time
window. Move Left/Right, then Up/Down again; a stale horizontal command should
not change the guide time. Check long programmes and rows with different
programme boundaries. Unit tests do not replace this physical remote check.

To recover if the app is incompatible, reinstall your previous signed TV
package. This fix does not require changes to your server configuration.

## Reuse

The upstream GPL-2.0-or-later license and notices remain in place. Other builds
can port the two fix commits, but need to resolve and test differences in their
guide implementation and TV scroll manager.

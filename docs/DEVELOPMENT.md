# MUSE 开发说明

This document describes the full source project. A documentation-and-installer
repository alone is not sufficient to build the app: the source files and original
GLB assets under `assets/` are required. For installation, see the root README.

Run with Node.js 22.12+ (or 24+) and npm:

```sh
npm install
npm run dev
```

The dev server and production build automatically prepare the GLB files in
`assets/`. The original files stay unchanged. One shared, textured, skinned
character is compressed with Meshopt; animation tracks are extracted into
`public/assets/motions/`. Only motions present in the current source folder
appear in the catalog. A content fingerprint skips unchanged preprocessing.

```sh
npm run prepare:assets
npm test
npm run build
npm run preview
```

## Desktop App (Tauri 2 + Rust)

The desktop app bundles the prepared character, animations and frontend locally.
It runs without a web server or internet connection, with a native window,
native fullscreen, local music import and microphone input. The web preview
remains available through `npm run dev`.

Prerequisites: Node.js 22.12+ and Rust stable. On macOS install Xcode Command
Line Tools (`xcode-select --install`). Windows needs Visual Studio C++ Build
Tools and WebView2; Linux needs the Tauri 2 system dependencies for the distro.

```sh
npm install
npm run desktop:dev
npm run desktop:build
```

Desktop development uses `127.0.0.1:1420` with a strict port, independently of
the web preview on port 5173. Release artifacts are in `src-tauri/target/release/bundle/`:
`macos/MUSE.app` and `dmg/` on macOS, `nsis/` and `msi/` on Windows. Build on the
target OS; a macOS build does not produce a Windows installer.

The macOS bundle is ad-hoc signed for local use. Public distribution requires
an Apple Developer signing identity and notarization. The first microphone
request triggers macOS permission; it can be changed in System Settings >
Privacy & Security > Microphone. No sound is uploaded. Escape exits fullscreen.

`app-icon-source.png` is the supplied dancing image. `npm run desktop:icons`
adds a white background with a 12.5% safety margin on each side, creates
`app-icon.png` and regenerates the platform icons and favicon.

The shrink button below fullscreen enters mini mode: a transparent, borderless
360 x 520 window containing only the dancer, kept above other windows. Music
and choreography continue in the same session. Drag the dancer to move it;
double-click, right-click or press Escape to restore the previous window size
and position. On macOS, Stage > Restore (Cmd+Shift+M) also restores the full UI.
No playback starts automatically when entering mini mode.

The Rust entry point and window/security configuration are in `src-tauri/`.
Native capabilities are limited to window presentation and local event listening.
macOS transparency requires Tauri's `macos-private-api` feature; this build is
intended for direct desktop distribution rather than the Mac App Store.

## Rhythm and Motion

- `src/audio-engine.js`: microphone lifecycle, offline tempo/first-beat analysis,
  local audio decoding, demonstration track, pause and resume.
- `src/rhythm.js`: audio-clock beat grid and live spectral-onset tempo estimate.
- `src/dance-controller.js`: absolute beat-to-keyframe mapping, bar-aligned
  transitions, crossfades, and phrase-aligned automatic choreography.
- `src/stage.js`: Three.js model, camera, lights and stage rendering.
- `src/main.js`: controls and transport state.

Local audio uses `web-audio-beat-detector` to estimate BPM and the first beat.
The microphone estimates tempo from recent attacks; speech, sparse music,
syncopation, changing tempo and noise can make this estimate unreliable. An
automatic estimate may also choose half or double the perceived tempo. The
controls support manual BPM, tap tempo and a -500 to +500 ms phase adjustment.
Positive phase adjustment delays the dance. Audio stays on the device.

GLB animation files contain keyframe timestamps but no musical beat markers.
The initial phrase length is estimated by rounding its duration at the configured
reference BPM to a multiple of four beats. For precise choreography, set `beats`
for each motion in `assets/motions.json`, then run `npm run prepare:assets`.
Original animation keyframes and root movement are preserved.

Choreography is fully automatic. It selects looping clips using music energy
smoothed over two seconds, BPM compatibility and recent motion history, then
completes the current phrase before changing. There are no motion names,
collection, manual dance selectors, style presets or choreography mode switches
in the interface. Pausing freezes music and dance together. Silence
holds the pose, and subsequent music resumes on the active beat grid. A track's
natural end plays the bow once. Microphone access requires localhost or HTTPS
and browser permission.

## Verification

`npm test` checks catalog/source agreement, skeleton bindings, compressed model
integrity, clock continuity, motion timing, onset estimates and transition/pause
behavior. Browser verification screenshots are in `output/verification/`.

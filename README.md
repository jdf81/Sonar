# SONAR — Live Audio Visualizer PWA

A Progressive Web App that captures live audio from your microphone and transforms it into reactive visual equalizer effects. Universal mobile-first design — works on iOS, Android, tablets, and desktop. Installable, offline-capable, fully local.

## Features

**6 live visualizer effects** — switch between them while recording:

1. **Bars** — classic vertical equalizer with neon gradient
2. **Mirror** — split cyan/magenta bars mirrored across the center
3. **Waveform** — oscilloscope-style time-domain trace with grid
4. **Radial** — frequency bars radiating from a pulsing core
5. **Particles** — bass-reactive particle bursts
6. **Tunnel** — endless 3D-feeling rings of audio-deformed geometry

**Plus:**
- Live RMS input level meter
- Real-time HUD: sample rate, peak dB, FPS, current mode
- **Fullscreen mode** for true edge-to-edge visualization
- **Screen Wake Lock** — stays awake while visualizing
- **Haptic feedback** on tap (Android)
- **Keyboard shortcuts**: `1`-`6` switch effects, `Space` toggles mic, `F` toggles fullscreen, `Esc` stops
- Installable as a standalone app (Add to Home Screen)
- Long-press app icon on Android for shortcut to specific effects
- Works offline after first load (service worker caches app shell)
- All audio processing happens locally — nothing is sent anywhere

## Universal device support

SONAR is built to work on every modern device. Specific accommodations:

| Concern | How SONAR handles it |
|---|---|
| iOS audio gestures | AudioContext is created synchronously in the user gesture, before any `await` |
| iOS `interrupted` state | Listens for state change and prompts user to resume after Siri / phone call |
| Android Chrome | Native install prompt via `beforeinstallprompt` |
| iOS Safari install | Custom Share-icon banner since iOS doesn't fire `beforeinstallprompt` |
| URL bar collapse | `100dvh` dynamic viewport units — no jumpy layout |
| Rotation | `ResizeObserver` on stage element catches every size change |
| Foldables / split screen | Same `ResizeObserver` handles hinge/split events |
| Low-end devices | Auto-detected perf tier scales FFT size, particle cap, and DPR |
| Reduced motion | Respects `prefers-reduced-motion` OS setting |
| Tablets | Larger touch targets and HUD typography at `≥768px` |
| Landscape phones | Compact chrome, smaller mic button, shrinks HUD |
| Desktop | Same UI plus full keyboard control |
| Wake Lock | Acquires on start, re-acquires on tab return, releases on stop |
| Haptics | `navigator.vibrate(8)` on tap (Android only — iOS ignores) |
| Permissions | Pre-checks via Permissions API to surface "blocked" state immediately |

## Performance tiers

SONAR sniffs `navigator.deviceMemory` and `navigator.hardwareConcurrency` and picks one of three tiers automatically:

| Tier | Trigger | FFT size | Particle cap | DPR cap |
|---|---|---|---|---|
| **low** | ≤ 2 GB RAM or ≤ 2 cores | 1024 | 200 | 1 |
| **mid** | mobile + ≤ 4 GB RAM/cores | 2048 | 400 | 2 |
| **high** | everything else | 2048 | 600 | 2 |

## How to run

The Web Audio API requires **HTTPS** (or localhost) to access the microphone, so you can't just open `index.html` from disk on mobile. Pick one of these:

### Quickest: Python local server
```bash
cd sonar-pwa
python3 -m http.server 8000
```
Then visit `http://localhost:8000` on the same machine. To test on your phone, find your computer's IP and visit `http://YOUR-IP:8000` — but note that **Chrome/Safari on mobile only allow mic access over HTTPS for non-localhost URLs**, so for real mobile testing use one of the options below.

### Mobile testing: ngrok
```bash
cd sonar-pwa
python3 -m http.server 8000
# in another terminal:
ngrok http 8000
```
Visit the `https://...ngrok.io` URL on your phone — mic access will work.

### Production: any static host
Drop the folder onto Netlify, Vercel, GitHub Pages, Cloudflare Pages, or any static host with HTTPS. Zero configuration needed.

## Installing on your phone

**Android (Chrome / Edge / Samsung Internet):** A banner appears, or use the menu → "Install app" / "Add to Home Screen". Once installed, long-pressing the icon shows shortcuts to launch directly into Bars, Particles, or Tunnel.

**iOS (Safari 16.4+):** Tap the Share button → "Add to Home Screen". Installation gives standalone (no browser chrome) launch and offline access. Mic works inside the standalone app on iOS 16.4+.

**Desktop (Chrome / Edge):** Click the install icon in the address bar.

Once installed it launches in standalone mode and works offline.

## Files

```
sonar-pwa/
├── index.html                   # Main app (HTML + CSS + JS in one file)
├── manifest.json                # PWA manifest with shortcuts
├── sw.js                        # Service worker (offline caching)
├── icon.svg                     # Vector icon
├── icon-192.png                 # 192×192 (Android / generic)
├── icon-512.png                 # 512×512 (Android / generic)
├── icon-maskable-512.png        # 512×512 maskable (Android adaptive)
├── apple-touch-icon.png         # 180×180 (iOS retina)
├── apple-touch-icon-167.png     # 167×167 (iPad Pro)
├── apple-touch-icon-152.png     # 152×152 (older iPad)
└── README.md                    # This file
```

## Browser support

| Browser              | Mic | Install | Offline | Wake Lock | Fullscreen | Haptics |
|----------------------|:---:|:-------:|:-------:|:---------:|:----------:|:-------:|
| Chrome (Android)     | ✅  |   ✅    |   ✅    |    ✅     |     ✅     |   ✅    |
| Safari (iOS 16.4+)   | ✅  |   ✅    |   ✅    |    ✅     |     ⚠️*    |   —     |
| Samsung Internet     | ✅  |   ✅    |   ✅    |    ✅     |     ✅     |   ✅    |
| Firefox (Android)    | ✅  |   ⚠️    |   ✅    |    ✅     |     ✅     |   ✅    |
| Chrome (Desktop)     | ✅  |   ✅    |   ✅    |    ✅     |     ✅     |   —     |
| Safari (macOS)       | ✅  |   ✅    |   ✅    |    ⚠️     |     ✅     |   —     |
| Edge (Desktop)       | ✅  |   ✅    |   ✅    |    ✅     |     ✅     |   —     |

\* iOS Safari uses CSS-only fullscreen since the Fullscreen API is not exposed.

## Privacy

The microphone stream is processed entirely in-browser via the Web Audio API. No audio is recorded to disk, transmitted over the network, or shared with any service. The app makes zero outbound requests after the initial page load (the Google Fonts CSS is the only external resource, and it's cached by the service worker on first visit).

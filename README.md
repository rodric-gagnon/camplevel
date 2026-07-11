# Camp Level

A bullseye bubble level that runs in the browser. Built for leveling a travel trailer — set the phone flat on the floor, watch the bubble, level by ear.

No app store, no dependencies, no build step. Three static files.

## Features

- **Bullseye + grid dial** — the bubble floats to the high side; the ring marks the ±0.5° tolerance zone at true scale
- **X / Y readouts** — signed degrees off level, large enough to read from standing height; each axis turns green when it's within tolerance
- **Audio guidance** — marimba-style knocks that quicken as you approach level, a two-note rise when you arrive, then a soft knock every 2 seconds while you stay there. Level the trailer by ear while you watch the tires instead of the phone. Mutable via the SOUND toggle
- **Flip calibration** — via the **CAL** button: read, rotate the phone 180° in place, read again. Averaging the two cancels everything attached to the phone — accelerometer bias *and* the tilt from the phone resting on its camera bump, which alone can exceed the 0.5° level tolerance. A rust dot on CAL means no calibration is stored yet; run it once per phone (and again if you change cases). The surface you calibrate on doesn't need to be level — the math cancels it out
- **Works offline** — a service worker caches everything on first load; no signal required at the campsite
- **Portrait and landscape** — layout reflows, and the X/Y axes remap to follow the screen so they always mean what you see
- **Screen wake lock** — the display stays on while the phone sits on the floor mid-leveling
- **Home screen installable** — launches fullscreen from an icon like a native app

## Usage

1. Open the page in Safari and tap **START** (iOS requires a tap to grant motion-sensor access)
2. Set the phone flat on the trailer floor
3. Adjust until the bubble sits in the ring, both readouts go green, and you hear the two-note rise
4. Recommended: tap **CAL** once per phone — the phone's camera bump tilts it more than the level tolerance, and the flip calibration cancels that out

Level tolerance is ±0.5° per axis, with hysteresis (exits at 0.7°) so the state doesn't flicker at the boundary.

## Install to home screen (iOS)

Open the page in Safari → Share → **Add to Home Screen**. The icon and fullscreen behavior are already wired up.

If motion sensors don't respond when launched from the home screen icon, open the page in regular Safari once, grant permission, then relaunch from the icon.

## Deployment

Any static host over HTTPS works (sensor APIs and service workers both require it). For GitHub Pages:

1. Put all three files in a directory, with the HTML named `index.html`:

   ```
   level/
   ├── index.html
   ├── sw.js
   └── apple-touch-icon.png
   ```

2. Enable Pages in the repo settings
3. Load the page once with a connection — after that it works offline

## How it works

The `DeviceOrientationEvent` API supplies `beta` (pitch) and `gamma` (roll). The app:

- low-pass filters the readings so the bubble doesn't jitter
- subtracts the stored calibration bias
- remaps device axes to screen axes using `screen.orientation.angle`, so rotating the phone doesn't swap the meaning of X and Y
- drives the SVG bubble, the readouts, and a Web Audio beep loop from a single `requestAnimationFrame` render pass

No frameworks, no libraries. The dial is inline SVG; audio is synthesized marimba knocks from raw oscillators. The only external resource is the DM Mono typeface from Google Fonts — cached by the service worker after first load, with a system-monospace fallback if it ever isn't available.

## Browser support

Built for Safari on iOS (13+), which is the strictest target: it requires the tap-to-request permission flow and HTTPS. Works in Chrome on Android as well, where it additionally vibrates on reaching level (iOS ignores the vibration API).

Desktop browsers load the page but report no sensor data.

## Known limits

- Accuracy is bounded by the phone's accelerometer — good to a few tenths of a degree after flip calibration, which is plenty for trailer leveling but not machinist work
- iOS occasionally quirks on `screen.orientation.angle` in home-screen web-app mode; if an orientation reads wrong, use the app in Safari proper

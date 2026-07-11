# Camp Level

A bullseye bubble level that runs in the browser. Built for leveling a travel trailer — set the phone flat on the floor, watch the bubble, level by ear.

No app store, no dependencies, no build step. Three static files.

## Features

- **Bullseye + grid dial** — the bubble floats to the high side; the ring marks the ±0.5° tolerance zone at true scale
- **X / Y readouts** — signed degrees off level, large enough to read from standing height; each axis turns green when it's within tolerance
- **Audio guidance** — parking-sensor style: beeps quicken as you approach level, solid tone when you're there. Level the trailer by ear while you watch the tires instead of the phone. Mutable via the SOUND toggle
- **Flip calibration** — one-time on first launch: read, rotate the phone 180°, read again. Averaging the two cancels the accelerometer's factory bias. Tap the **CAMP LEVEL** title anytime to recalibrate
- **Works offline** — a service worker caches everything on first load; no signal required at the campsite
- **Portrait and landscape** — layout reflows, and the X/Y axes remap to follow the screen so they always mean what you see
- **Screen wake lock** — the display stays on while the phone sits on the floor mid-leveling
- **Home screen installable** — launches fullscreen from an icon like a native app

## Usage

1. Open the page in Safari and tap **START** (iOS requires a tap to grant motion-sensor access)
2. First launch only: run the two-tap flip calibration, or skip it
3. Set the phone flat on the trailer floor
4. Adjust until the bubble sits in the ring, both readouts go green, and the tone goes solid

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

No frameworks, no libraries. The dial is inline SVG; audio is two oscillator patterns.

## Browser support

Built for Safari on iOS (13+), which is the strictest target: it requires the tap-to-request permission flow and HTTPS. Works in Chrome on Android as well, where it additionally vibrates on reaching level (iOS ignores the vibration API).

Desktop browsers load the page but report no sensor data.

## Known limits

- Accuracy is bounded by the phone's accelerometer — good to a few tenths of a degree after flip calibration, which is plenty for trailer leveling but not machinist work
- iOS occasionally quirks on `screen.orientation.angle` in home-screen web-app mode; if an orientation reads wrong, use the app in Safari proper

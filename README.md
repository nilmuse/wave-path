# wave-path

Animate layered SVG overlays with wave-shaped edges when they open and close.

[![npm](https://img.shields.io/npm/v/wave-path.svg?colorB=brightgreen)](https://www.npmjs.com/package/wave-path)
[![NPM Downloads](https://img.shields.io/npm/dm/wave-path.svg?style=flat)](https://www.npmjs.com/package/wave-path)

[Demo](https://codepen.io/ux-ui/pen/Jjervqg)

## Installation

```bash
npm install wave-path
```

## Quick start

In a browser app with a bundler, add a button and an SVG with one `<path>` per layer. Use a `0 0 100 100` viewBox because the generated path coordinates run from `0` to `100`:

```html
<button id="toggle-wave" type="button">Toggle wave</button>

<svg
  id="wave-overlay"
  class="wave-preview"
  viewBox="0 0 100 100"
  preserveAspectRatio="none"
  aria-hidden="true"
>
  <path fill="#98b4c9"></path>
  <path fill="#536f88"></path>
  <path fill="#243b53"></path>
</svg>
```

```css
.wave-preview {
  display: block;
  width: 100%;
  height: 320px;
  background: #101820;
}
```

Run this TypeScript module after the markup exists:

```ts
import WavePath from 'wave-path';

const button = document.getElementById('toggle-wave');

if (!button) {
  throw new Error('Wave toggle button was not found');
}

const wavePath = new WavePath({
  svgEl: '#wave-overlay',
  isOpened: false,
  numberPoints: 6,
  waveAmplitude: 30,
  delayPaths: 0.25,
  duration: 1.5,
});

wavePath.init();
button.addEventListener('click', () => void wavePath.toggle());
```

Click the button to open or close the layers. WavePath updates each path's `d` attribute; SVG sizing and colors come from your markup and CSS. Call `wavePath.destroy()` before removing the SVG.

## API

`WavePath` is the default export. `WavePathOptions` is the constructor options type:

```ts
import WavePath from 'wave-path';
import type { WavePathOptions } from 'wave-path';
```

### Options

Pass these options to `new WavePath(options)`:

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `svgEl` | `string \| SVGElement` | Required | SVG selector or element. |
| `pathEl` | `string` | `'path'` | Selector for paths inside the SVG. Each match is animated as a layer. |
| `numberPoints` | `number` | `4` | Number of wave control points, clamped to `3`–`32`. Higher values add detail to the edge. |
| `waveAmplitude` | `number` | `30` | Ripple amplitude, clamped to `0`–`100`. Use `0` for a straight edge. |
| `delayPaths` | `number` | `0.25` | Delay between path layers, in seconds. |
| `duration` | `number` | `1` | Animation duration for each layer, in seconds. |
| `isOpened` | `boolean` | `false` | Initial state, applied by `init()`. Change it with `open()`, `close()`, or `toggle()`. |

### Methods

| Method | Returns | Description |
| --- | --- | --- |
| `init()` | `void` | Find the SVG and paths, allocate internal buffers, and apply the initial shape. Call again if the SVG content changes. |
| `toggle()` | `Promise<void>` | Toggle the overlay state and animate the transition. |
| `open()` | `Promise<void>` | Open the overlay. |
| `close()` | `Promise<void>` | Close the overlay. |
| `totalDurationMs()` | `number` | Total animation duration in milliseconds, including layer delays. Call after `init()` so the path count is known. |
| `stopIfActive()` | `void` | Cancel the current animation if one is running. |
| `destroy()` | `void` | Cancel animation and clear references and caches. Safe to call more than once. |

Read `wavePath.isOpened` for the current logical state. Animation methods resolve when the transition completes. Calls to `open()`, `close()`, or `toggle()` while an animation is running are ignored.

## Sequence transitions

Continuing with `wavePath` from Quick start, run transitions one after another:

```ts
await wavePath.open();
await wavePath.close();
await wavePath.toggle();
```

## Animation timing

Timing starts on the first rendered frame and uses elapsed time, so `duration` and `delayPaths` do not depend on the display's refresh rate. The library animates cubic-bezier SVG edges and blends large ripples between the top and bottom bounds. Typed arrays and reused string buffers reduce allocations during rendering.

Fast fullscreen transitions have larger steps between frames on a 60 Hz display than on a 120 or 144 Hz display. Increase `duration` to reduce the distance moved per frame. `numberPoints` changes edge detail, not frame rate.

## Browser support

Call `init()` after the SVG and its paths exist in the browser. The animation uses SVG paths and `requestAnimationFrame`.

## License

MIT

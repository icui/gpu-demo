# Mandelbrot: CPU vs GPU in your browser

A single self-contained page (`index.html`, no build step, no dependencies) that renders the same Mandelbrot image three ways:

1. **CPU · 1 core**: plain JavaScript in one Web Worker
2. **CPU · all cores**: the image split into strips across one Web Worker per core
3. **GPU (WebGPU)**: a WGSL compute kernel with one thread per pixel

Used as a ~2-minute live demo in the "What is a GPU?" session (after the "Many workers, a small program" slide); listeners can run it on their own phones and laptops.

## Publish on GitHub Pages

GitHub Pages serves from the root of a branch or from a `/docs` folder, so either:

**Option A: a dedicated repository (simplest URL)**

```sh
mkdir gpu-demo && cp index.html README.md gpu-demo/ && cd gpu-demo
git init && git add . && git commit -m "Add CPU vs GPU Mandelbrot demo"
gh repo create gpu-demo --public --source=. --push
```

Then on GitHub: **Settings → Pages → Build and deployment → Deploy from a branch → `main` / `(root)`**.
The page appears at `https://<your-user>.github.io/gpu-demo/` after a minute or two. For the shortest URL to type, name the repository `gpu` (→ `<your-user>.github.io/gpu`), and show a QR code on the slide.

**Option B: inside an existing repository**: copy `index.html` into a `docs/` folder and choose `main` / `/docs` in the same Pages settings.

No special headers are needed: the page does not use `SharedArrayBuffer`, so it works on GitHub Pages and from a local file.

## Browser support

- **GPU mode**: WebGPU in recent Chrome and Edge (Windows, macOS, ChromeOS, Android), Safari 26+, and Firefox on Windows. On Linux, Chrome may need WebGPU enabled in `chrome://flags`. If WebGPU is missing, the page says so and the CPU modes still work.
- **CPU modes**: any modern browser.

## Presenting (audience on their own devices)

Put the short URL or a QR code on screen. Each listener opens it on a phone or laptop and taps **Start test**. The page has two views: a start page (three instructions, one button, a WebGPU check) and a result page (the image, the three timings, the headline speedup, and two collapsed sections: *Explore the image* and *How this test works*).

- **Auto size** first times one CPU core on a tiny image, then picks the largest image (up to 3840 px wide) for which one core takes about 2 seconds on *that* device. The whole test (1 core, all cores, GPU) takes about 5 seconds, so phones do not overheat.
- The headline says how many times faster the GPU was than one core and than all cores; the start page shows the GPU name the browser reports (often just the vendor).
- Nothing heavy runs on page load, and no data leaves the device (there is no server; ask for a show of hands instead: "who got more than 50×? more than 100×?").
- Without WebGPU (older iPhones before iOS 26, older Android, Firefox on Android) the page says so and still compares 1 core against all cores.

Presenter keys: `T` runs the test (from either page), and on the result page `1` `2` `3` switch mode, `D` deep zoom, `R` reset. Click/tap the image to zoom in; *Explore the image* has Zoom out, size, and iteration controls.

Suggested flow (about 2 minutes): show the QR code; while people open it, run the test on the projector (`T`) and narrate 1 core → all cores → GPU; ask for a show of hands; optionally press `D` and switch `2`/`3` to show the FP32 vs FP64 difference.

Measured in Edge on an 8-core Apple laptop (Auto, 500 iterations): 1 core 2.0 s, 8 cores 0.43 s (~5×), GPU 14 ms (~140×). Speedups depend on the device.

## Caveats to say out loud

- This is a demonstration, not a benchmark: the CPU side is plain JavaScript, not optimized and vectorized C.
- GPU time is measured after a warm-up launch (best of three); the very first launch also includes setup.
- At tiny resolutions, 8 cores are barely faster than 1: with too little work, coordination overhead dominates.

# Frame Extractor (single file)

Extract individual frames from a video — entirely in your browser. One `index.html`, no dependencies, no server, no uploads.

## Privacy guarantee
- Your video is read with `URL.createObjectURL` and decoded locally by the browser. It is **never** uploaded.
- There are **zero** network requests at runtime — JSZip is inlined, and there is no analytics, no error tracking, no CDN, no fonts.
- Works fully offline. Open `index.html` from disk with your network disconnected and it still works.

## How to verify the privacy claim yourself
1. Open `index.html` in your browser.
2. Open DevTools → **Network** tab, check "Disable cache", and clear the log.
3. Load a video, extract frames, download the zip.
4. The Network tab stays empty (or shows only the local `index.html`). Nothing leaves your machine.

## How it works
Hidden `<video>` → seek to each timestamp → wait for `seeked` + two animation frames → `drawImage` onto a `<canvas>` at the video's original resolution → `canvas.toBlob` → zip in-memory with JSZip → download via a generated link.

## Usage
Open `index.html`, drop in a video, set the interval/format, click **Extract frames**. A `frames.zip` downloads.

## Limitations
- Only codecs your browser's `<video>` element supports (typically H.264; often VP9/AV1) can be decoded.
- Very long/high-res runs are memory-bound — raise the interval or lower max frames if a tab runs out of memory.

## Credit
Inspired by the privacy model of https://frame-extractor.com — this version removes its third-party error tracking and CDN scripts for a truly zero-network build.

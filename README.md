# Frame Extractor (single file)

Extract individual frames from a video — entirely in your browser. One `index.html`, no dependencies, no server, no uploads.

## Privacy guarantee
- Your video is read with `URL.createObjectURL` and decoded locally by the browser. It is **never** uploaded.
- By default there are **zero** network requests at runtime — JSZip is inlined, and there is no analytics, no error tracking, no CDN, no fonts.
- Works fully offline. Open `index.html` from disk with your network disconnected and it still works.
- **One opt-in exception:** the *universal decoder* (see below) downloads ffmpeg.wasm (~30 MB) **only if you explicitly click to enable it**. Even then your video is processed locally and never uploaded. If you never click that button, the page makes no network requests at all.

## How to verify the privacy claim yourself
1. Open `index.html` in your browser.
2. Open DevTools → **Network** tab, check "Disable cache", and clear the log.
3. Load a video, extract frames, download the zip.
4. The Network tab stays empty (or shows only the local `index.html`). Nothing leaves your machine.

## How it works
Hidden `<video>` → seek to each timestamp → wait for `seeked` + two animation frames → `drawImage` onto a `<canvas>` at the video's original resolution → `canvas.toBlob` → zip in-memory with JSZip → download via a generated link.

## Usage
Open `index.html`, drop in a video, set the interval/format, click **Extract frames**. A `frames.zip` downloads.

## Video format & codec support
The container (`.mp4`, `.mov`, `.webm`, …) barely matters — the **codec inside** is what counts, and the default path can only decode what your browser's `<video>` element supports:

| Codec | Chrome / Edge | Firefox | Safari |
|-------|:-:|:-:|:-:|
| H.264 / AVC | ✅ | ✅\* | ✅ |
| VP8 / VP9 | ✅ | ✅ | ⚠️ |
| AV1 | ✅ | ✅ | ⚠️ (newer) |
| HEVC / H.265 | ⚠️ (hardware) | ❌ | ✅ |
| ProRes / MJPEG / MPEG-4 ASP | ❌ | ❌ | ❌ |

\* Firefox H.264 relies on a system decoder (fine on Windows/macOS; on Linux it needs system ffmpeg/gstreamer installed).

**Why an iPhone `.mov` may fail in Firefox:** iPhones set to "High Efficiency" record **HEVC/H.265**, which Firefox can't decode. The same file usually works in Safari or Chrome. (Note: simply remuxing to `.mp4` in VLC does **not** help — it keeps the HEVC stream. You must *re-encode* to H.264: `ffmpeg -i in.mov -c:v libx264 -pix_fmt yuv420p out.mp4`.)

When a file won't decode, the tool **parses the file and tells you the exact codec** (e.g. "Detected codec: HEVC / H.265 (hvc1)") instead of failing silently, and offers the universal decoder.

## Universal decoder (opt-in)
If your browser can't decode a file, click **Enable universal decoder**. This loads [ffmpeg.wasm](https://github.com/ffmpegwasm/ffmpeg.wasm) (a WebAssembly build of ffmpeg, ~30 MB, downloaded once) and uses it to transcode the video to browser-safe H.264 **entirely on your machine** — the file is still never uploaded. This handles HEVC, ProRes, AVI, and most other formats.

Trade-offs: the one-time download needs a network connection, transcoding is slower than native decoding, and very large files are memory-bound. The default (native) path remains 100% offline and zero-network unless you opt in.

## Limitations
- The default path only decodes codecs your browser's `<video>` element supports (see table above); use the universal decoder for the rest.
- Very long/high-res runs are memory-bound — raise the interval or lower max frames if a tab runs out of memory.

## Credit
Inspired by the privacy model of https://frame-extractor.com — this version removes its third-party error tracking and CDN scripts for a truly zero-network build.

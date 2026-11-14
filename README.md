# Flatbed

A browser document scanner: take photos, upload images or import a PDF, drag four corner handles (with a magnifier) to square each page, choose a scan look, export one PDF.

## Run it

- Quickest: open `index.html` in a browser. Upload, PDF import, editing and export all work offline.
- For the live webcam on desktop, serve it over HTTPS or localhost, e.g. `npx serve .` or `python3 -m http.server`, then open http://localhost:8000 (browsers only grant camera access on secure origins).
- On phones, "Take photo" opens the native camera at full sensor resolution.
- To host it, drop the folder on any static host (Netlify, Vercel, GitHub Pages, Cloudflare Pages).

## Speed

- PDFs whose pages are photos (iPhone/macOS scans, most scanner output) are read by copying each page's JPEG out of the file as-is: no decoding, no quality change. Other pages are rendered with pdf.js: a preview first, full resolution in the background.
- Heavy work (perspective warp, scan look, compression) runs in Web Workers, several pages in parallel, using the browser's native compressor. If workers are unavailable the same code runs on the main thread.
- Each page is processed in the background as soon as it settles (after import, after leaving it in the editor, or a second after your last change), so Export usually only assembles the file.

## Quality

- Perspective correction runs at the photo's native resolution with bicubic sampling; nothing is downscaled. Whole-image pages (including quarter-turn rotations) are copied pixel for pixel, with no resampling.
- Lossless export stores the processed pixels with Flate compression (what PNG uses). Grayscale and black & white pages are stored single-channel.
- A JPEG left untouched (whole image, Original look, no rotation) is embedded byte for byte.
- "Same size for all" pages only changes the PDF page box and placement, never the image pixels.
- iOS Safari limits canvases to about 16.7 MP; larger images are scaled to that limit on iPhone/iPad only.

## Files

- `index.html` - the whole app (HTML, CSS, JS)
- `vendor/pdf-lib.min.js` - pdf-lib 1.17.1 (MIT): builds the PDF and reads photo-only PDFs
- `vendor/pdf.min.js`, `vendor/pdf.worker.min.js` - pdf.js 3.11.174 (Apache-2.0): renders other PDF pages
- Each falls back to the jsDelivr CDN if missing.

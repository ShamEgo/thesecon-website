# SECON website — deployment package

Hosting-ready static site. The videos are separate files (not embedded), so the
page loads instantly and the clips stream in progressively.

## Folder structure
```
secon-site/
├── index.html                    # the page (~280 KB; logos are inline PNGs)
└── videos/
    ├── hero.mp4                   # hero background  (1.6 MB, H.264, 1080p, no audio)
    ├── hero-poster.jpg           # still shown while hero loads
    ├── industries.mp4            # industries background (2.6 MB)
    └── industries-poster.jpg
```

## Deploy
Drop the whole `secon-site/` folder onto any static host:
- **Netlify / Vercel:** drag the folder into the dashboard, or connect a Git repo.
- **GitHub Pages:** push the contents to a repo and enable Pages.
- **Cloudflare Pages / S3 / any web server:** upload as-is.

Keep `index.html` and the `videos/` folder together — the paths are relative
(`videos/hero.mp4`), so the structure must be preserved.

## Preview locally
Because the page loads video by relative URL, open it through a tiny local
server rather than double-clicking the file:
```
cd secon-site
python3 -m http.server 8000
# then visit http://localhost:8000
```

## How the videos were compressed (source → web)
Originals were 1920×1080 @ 30 fps, ~6 Mbps. Re-encoded with H.264, CRF 28,
audio stripped, and `+faststart` for progressive streaming:
```
ffmpeg -i source_hero.mp4        -an -c:v libx264 -preset slow -crf 28 \
       -pix_fmt yuv420p -movflags +faststart videos/hero.mp4
ffmpeg -i source_industries.mp4  -an -c:v libx264 -preset slow -crf 28 \
       -pix_fmt yuv420p -movflags +faststart videos/industries.mp4
# poster frames (grab a frame 1s in)
ffmpeg -ss 1 -i source_hero.mp4       -frames:v 1 -q:v 3 videos/hero-poster.jpg
ffmpeg -ss 1 -i source_industries.mp4 -frames:v 1 -q:v 3 videos/industries-poster.jpg
```
Result: hero 12 MB → 1.6 MB, industries 21 MB → 2.6 MB.

## Optional further gains
- **Lower CRF for higher quality** (e.g. `-crf 24`) or **higher for smaller files** (`-crf 30`).
- **720p variant** for mobile: add `-vf scale=1280:-2`.
- **WebM/AV1** alongside MP4 for modern browsers (smaller again): encode a
  `.webm` and add a second `<source>` before the mp4.
- **Put videos on a CDN** and change the `<source src>` to the CDN URL.

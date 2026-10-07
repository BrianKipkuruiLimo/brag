# brag — site

Static single-page launch site for `/brag`. Plain HTML/CSS/JS, no framework, no build step. This folder is the GitHub Pages deploy root.

## Local preview

```bash
python3 -m http.server 8000
open http://localhost:8000
```

## Sanity check

```bash
node ../scripts/check-docs.mjs
```

This verifies local docs links and media paths.

## Deploy (GitHub Pages)

Fully self-contained — no build step. In the repo's **Settings → Pages**, set the source to **Deploy from a branch**, branch `main`, folder **`/docs`**. The custom domain is `bragz.co.ke` (`CNAME` is already in this folder).

In **Settings → Pages → Custom domain**, set the domain to `bragz.co.ke`. At your DNS provider, point the domain's apex (`@`) to GitHub Pages using all four A records:

| Type | Host | Value |
|---|---|---|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |

For IPv6, also add these four AAAA records:

| Type | Host | Value |
|---|---|---|
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |

GitHub recommends also configuring `www` with a CNAME record pointing directly to the account's Pages host, `briankipkuruilimo.github.io` (do not include the repository name). Remove conflicting records at `@` and `www`; do not use wildcard DNS. DNS changes can take up to 24 hours to propagate. After DNS resolves to GitHub Pages, enable **Enforce HTTPS** in Pages settings when it becomes available.

The gallery videos ship committed, in two sets. The /brag-slim set is `examples/horse-tinder/`, `examples/fish-flight-school/` and `examples/taxi-for-taxis/`; the /brag --full set is `examples/full/<slug>/`, which keeps the original version of each site next to the video made from it. Each demo includes its `brag.mp4`, a `brag.jpg` poster, and a `site.jpg` thumbnail. Heavy composition sources (`brag-output-*/`) are git-ignored.

## Adding / updating a gallery example

1. Render the brag video and place `brag.mp4`, `brag.jpg` (poster), and `site.jpg` (thumbnail) under `examples/<slug>/`, next to the demo's `index.html` and `styles.css`. Pick `brag.jpg` as the **best** frame, not an arbitrary one — grab the video's strongest settled beat (the hook line, or the hero/logo reveal) full-res with ffmpeg, then bake it as the video's frame 0 so idle thumbnails everywhere show it:

   ```bash
   # extract the best settled beat as the poster
   ffmpeg -ss 3.2 -i docs/examples/<slug>/brag.mp4 -frames:v 1 -q:v 2 docs/examples/<slug>/brag.jpg

   # replace only frame 0 with the poster (same duration, frames, and audio)
   cd docs/examples/<slug>
   ffmpeg -y -i brag.mp4 -i brag.jpg \
     -filter_complex "[0:v][1:v]overlay=0:0:enable='eq(n,0)'[v]" \
     -map "[v]" -map 0:a? -c:v libx264 -crf 18 -preset slow -pix_fmt yuv420p \
     -c:a copy -movflags +faststart brag.poster.mp4 && mv brag.poster.mp4 brag.mp4
   ```
2. Add a card in `index.html` pointing at those paths.
3. Un-ignore the slug in `.gitignore` and commit the assets.

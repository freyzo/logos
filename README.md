# logos

_3000+ normalized brand icons as a static CDN_
_Fetch any domain's logo, squircle-masked, 1024×1024 WebP_

---

## About

**Problem**

- Brand icons come in every size, format, and background color.
- No single source gives you a consistent, ready-to-use set.
- Fetching a domain's logo programmatically means juggling favicons, og:image, and Bing scrapes.

**Solution**

- **`logos`** is an icon CDN + CLI:
  - **`webp/`** → 3000+ icons, all `1024×1024 .webp`, metadata stripped, squircle-masked.
  - **`bin/logo.sh`** → give it a file, folder, or domain — get a normalized WebP back.
- Pre-commit hook enforces the spec so nothing bad lands in `webp/`.

**Summary**

| You want | Command |
|----------|---------|
| Normalize one file | `./bin/logo.sh icon.png` |
| Batch a folder | `./bin/logo.sh ./icons/ --out-dir webp/` |
| Fetch a domain's logo | `./bin/logo.sh --domain stripe.com` |
| Use an icon as CDN | see [CDN usage](#cdn-usage) |

> Requires **macOS / Linux**, **ImageMagick**, **exiftool**, **GNU Parallel**, **curl**.

---

## Install

One-time setup after clone:

```bash
.hooks/scripts/setup.sh      # activates pre-commit hook
./bin/venv.zsh               # Python venv for --domain pHash (optional)
```

**Deps** (macOS with Homebrew):

```bash
brew install imagemagick exiftool parallel curl
brew install librsvg          # optional: SVG input
```

---

## Usage

### Normalize a file

```bash
./bin/logo.sh path/to/logo.png
# → webp/logo.webp  (1024×1024, squircle mask, metadata stripped)
```

### Batch a folder

```bash
./bin/logo.sh ./icons/ --out-dir webp/
# skips files that already have output
```

### Fetch a domain's logo

```bash
./bin/logo.sh --domain stripe.com
# → webp/stripe.webp
```

Fetches Google favicons + site icons + Bing HTML scrape, picks the best via perceptual hash.

### Options

| Flag | Description |
|------|-------------|
| `--out <path>` | Override output path |
| `--out-dir <dir>` | Batch output root (default `webp/`) |
| `--size <N>` | Output square size (default 1024) |
| `--domain <host>` | Fetch + normalize a domain logo |
| `--logo-rip-only` | With `--domain`: print candidates table, no WebP |
| `--no-domain-prep` | Skip fuzz-trim before render |
| `--overwrite` | Replace existing outputs in batch mode |

---

## CDN usage

Reference any icon by raw GitHub URL:

```markdown
![stripe](https://raw.githubusercontent.com/freyzo/logos/main/webp/stripe.webp)
```

```html
<img src="https://raw.githubusercontent.com/freyzo/logos/main/webp/stripe.webp" width="64" />
```

---

## Repo layout

| Path | What |
|------|------|
| `webp/` | Icons — `1024×1024 .webp`, subdirs for namespaces (`webp/aws/`, `webp/macos/`) |
| `bin/logo.sh` | Main CLI: file → WebP, dir → batch, `--domain` → fetch + render |
| `bin/logo-prep-logo.zsh` | Sourced by `logo.sh` — fuzz-trim before render |
| `bin/phash-pick.zsh` | Picks best candidate by perceptual hash |
| `bin/venv.zsh` | Creates Python venv for pHash deps |
| `bin/requirements.txt` | Pillow, imagehash, numpy, scipy |
| `.hooks/pre-commit` | Blocks non-1024×1024 WebP commits under `webp/` |
| `workflow.yaml` | Canonical workflow spec |
| `workflows/` | Runbooks: clip-queue, logo-fetch pipeline |
| `data/domains.txt.example` | Example domain list for batch logo fetch |

---

## How it works

1. **Input** — file, folder, or domain.
2. **Fetch** (domain mode) — parallel curl: Google s2 favicons, site icons, Bing HTML scrape.
3. **Pick** — pHash Hamming distance vs anchor; prefer largest under threshold.
4. **Prep** — fuzz-trim content bounds, reject over-aggressive crops.
5. **Render** — detect background (opaque vs transparent), apply squircle mask, encode WebP.
6. **Strip** — `exiftool` removes all metadata before write.

---

## Contact

X [@freyazou](https://x.com/freyazou) · GitHub [freyzo](https://github.com/freyzo)

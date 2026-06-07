# Logos

A collection of normalized brand icons plus the tooling to make more.

Every icon in `webp/` is a **1024×1024 WebP** with metadata stripped, composited into a logo mask. Use them as a static CDN, or run the scripts locally to add new logos.

## Quick start

```bash
# one-time: enable git hooks (validates new WebPs on commit)
.hooks/scripts/setup.sh

# optional: Python venv for --domain / pHash picking
./bin/venv.zsh

# convert one file
./bin/logo.sh path/to/logo.png

# batch a folder (skips files that already have output)
./bin/logo.sh path/to/icons/ --out-dir webp/

# fetch + pick best logo for a domain, write webp/example.webp
./bin/logo.sh --domain example.com
```

Output lands in `webp/<name>.webp` by default.

## Requirements

- **ImageMagick** (`magick`)
- **exiftool** (metadata stripping)
- **GNU Parallel** (batch / domain mode)
- **curl** (domain fetches)
- **python3** + deps in `bin/requirements.txt` (only for `--domain` pHash picking)
- **rsvg-convert** (optional, SVG input)

Install Python deps:

```bash
./bin/venv.zsh   # creates bin/.venv/
```

## Repo layout

| Path | What |
|------|------|
| `webp/` | Normalized icons (namespaced subdirs OK, e.g. `webp/aws/`) |
| `bin/logo.sh` | Main entry — raster → logo WebP |
| `bin/phash-pick.zsh` | Perceptual-hash winner for `--domain` mode |
| `workflow.yaml` | Canonical workflow spec |
| `workflows/` | Extra runbooks (clip-queue, logo-fetch pipeline) |
| `repo.config.json` | Machine-readable repo manifest |
| `.hooks/pre-commit` | Blocks commits unless WebP is 1024×1024 under `webp/` |

## CDN usage

After you push, reference icons by raw GitHub URL:

```markdown
![icon](https://raw.githubusercontent.com/<owner>/<repo>/main/webp/google.webp)
```

Replace `<owner>` and `<repo>` with your fork.

## Workflows

- **clip-queue** — auto-rerender icons that clip at the mask edge. See `workflows/clip-queue.yaml`.
- **logo-fetch pipeline** — planned domain → fetch → pHash → logo flow. See `workflows/logo-fetch-phash-pipeline.yaml`.

## Tools install (optional)

Mirror `bin/` to a global tools dir and call from anywhere:

```bash
rsync -a --delete bin/ "$CURTOOLS/logo/"
export LOGO="$PWD"   # when your project has webp/
"$CURTOOLS/logo/logo.sh" "$@"
```

See `workflow.yaml` and `repo.config.json` for full details.

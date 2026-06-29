# nano-banana

A small global CLI for Google's **Nano Banana** image models
(`gemini-2.5-flash-image` and friends). Generate or edit images from any
directory, with the full `generateContent` config surface exposed as flags.

It's a thin wrapper over the official **[Google Gen AI SDK](https://github.com/googleapis/python-genai)**
(`google-genai`), executed as a [PEP 723](https://peps.python.org/pep-0723/)
inline-dependency script via [`uv`](https://docs.astral.sh/uv/). There's no
manual install or virtualenv — `uv` fetches the SDK into an ephemeral, cached
environment on first run.

## Requirements

- [`uv`](https://docs.astral.sh/uv/) on your `PATH` (the script's shebang is
  `#!/usr/bin/env -S uv run --script`).
- A Gemini API key in the environment as `GEMINI_API_KEY` (or `GOOGLE_API_KEY`).
  It's already exported from `~/.zshenv`. Get one at
  [Google AI Studio](https://aistudio.google.com/apikey).

## Install / aliases

The script lives at `/Volumes/Developer/code/shared/nano-banana/nano-banana`
and is aliased in `~/.aliases`:

```sh
alias nano-banana="/Volumes/Developer/code/shared/nano-banana/nano-banana"
alias banana="nano-banana"
```

Reload your shell (`source ~/.aliases`) and it's available everywhere.

## Usage

```sh
nano-banana "PROMPT" [options]
```

| Flag | Description |
| --- | --- |
| `-o, --out FILE` | Output PNG path (default `nano-banana-<epoch>.png`). Extra candidates get `-2`, `-3` suffixes. |
| `-i, --image FILE` | Input image to edit / condition on. Repeatable (multi-image composition). |
| `-m, --model NAME` | Model id (default `gemini-2.5-flash-image`). |
| `-a, --aspect-ratio RATIO` | One of `1:1 2:3 3:2 3:4 4:3 4:5 5:4 9:16 16:9 21:9`. |
| `-s, --image-size SIZE` | `1K` / `2K` / `4K`. Model-dependent (e.g. `gemini-3-pro-image`). |
| `-n, --count N` | `candidateCount` — number of images to generate. |
| `--temperature F` | Sampling temperature. |
| `--top-p F` | Nucleus sampling `top_p`. |
| `--top-k I` | `top_k` sampling. |
| `--seed I` | Deterministic seed (reproducible output). |
| `--max-output-tokens I` | Cap output tokens. |
| `--system TEXT` | System instruction. |
| `--modalities LIST` | Response modalities, comma list (default `IMAGE`; e.g. `IMAGE,TEXT`). |
| `--safety CAT:LEVEL` | Safety override, repeatable (see below). |
| `--api-key KEY` | Override the env key. |
| `--json` | Print usage/token metadata to stderr. |

**Safety categories:** `harassment hate sexual dangerous civic`
**Safety levels:** `BLOCK_NONE BLOCK_ONLY_HIGH BLOCK_MEDIUM_AND_ABOVE BLOCK_LOW_AND_ABOVE OFF`

### Examples

```sh
# Basic generation
nano-banana "a ripe banana on a plain white background"

# Widescreen, reproducible, save to a named file
nano-banana "alpine cabin logo, navy on white" -a 16:9 --seed 42 -o logo.png

# Generate 3 variations
nano-banana "sunset over mountains" -n 3 -o sunset.png
# -> sunset.png, sunset-2.png, sunset-3.png

# Edit an existing image
nano-banana "make the sky purple at dusk" -i photo.png -o edited.png

# Compose multiple inputs into one image
nano-banana "place the logo on the product box" -i logo.png -i box.png -a 1:1 -o mockup.png

# Loosen a safety filter, with token usage printed
nano-banana "dramatic battlefield concept art" --safety dangerous:BLOCK_ONLY_HIGH --json
```

## Notes

- **Aspect ratio → resolution.** `16:9` renders at 1344×768; with no
  `--aspect-ratio` the model infers from the last input image, or defaults to
  `1:1` when there's no input.
- **`--image-size`** is only honored by models that support it (e.g.
  `gemini-3-pro-image-preview`); `gemini-2.5-flash-image` controls dimensions via
  aspect ratio.
- All outputs carry an invisible **SynthID** watermark (Google policy).
- Models move fast. To try a newer one: `nano-banana "..." -m gemini-3-pro-image-preview`.

## How it works

The file is both the executable and its own dependency manifest:

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.10"
# dependencies = ["google-genai>=1.0.0"]
# ///
```

`uv` reads that header, resolves `google-genai` into a cached env, and runs the
script. First invocation installs; subsequent runs are instant.

## Changelog

Changes are tracked with [changie](https://changie.dev). Add an entry for any
user-facing change:

```sh
changie new            # pick a kind, write a one-line description
```

This drops a file in `.changes/unreleased/`. At release time, aggregate them
into `CHANGELOG.md`:

```sh
changie batch auto && changie merge
```

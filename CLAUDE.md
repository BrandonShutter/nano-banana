# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`nano-banana` is a single-file CLI that wraps the official Google Gen AI SDK
(`google-genai`) to generate or edit images with Gemini "Nano Banana" image
models (default `gemini-2.5-flash-image`). The entire program is the executable
script `nano-banana`; there is no package, build step, or test suite.

## How it runs

The script is a [PEP 723](https://peps.python.org/pep-0723/) inline-dependency
script. Its shebang is `#!/usr/bin/env -S uv run --script`, and the
`# /// script` header declares `google-genai` as a dependency. `uv` resolves
that dependency into a cached ephemeral environment on first run — there is no
virtualenv to create or `requirements.txt` to maintain.

```sh
# Run directly (requires `uv` on PATH)
./nano-banana "a ripe banana on a white background"

# Edit deps: change the `# dependencies = [...]` line in the header, not a separate file
```

Auth comes from `GEMINI_API_KEY` (or `GOOGLE_API_KEY`) in the environment, or
`--api-key`. See `README.md` for the full flag reference and examples.

## Code structure (all in `nano-banana`)

- Module-level constants (`ASPECT_RATIOS`, `IMAGE_SIZES`, `SAFETY_CATEGORIES`,
  `SAFETY_THRESHOLDS`) are the source of truth for both CLI `choices` and the
  help epilog — update the constant, not scattered strings.
- `build_parser()` — argparse definition; every generation knob is an optional
  flag mapped 1:1 onto `GenerateContentConfig` fields.
- `parse_safety()` — converts `CAT:LEVEL` specs into `types.SafetySetting`.
- `main()` — builds `contents` (prompt text + input image `Part`s), assembles
  `cfg_kwargs` (only setting keys the user actually passed, so SDK defaults
  apply otherwise), calls `client.models.generate_content`, then walks
  `response.candidates[].content.parts` extracting `inline_data` (images) and
  `text`.

## Conventions to preserve

- The SDK import is **lazy**, inside `main()` after the API-key check, so
  `--help` and missing-key errors work without `google-genai` installed.
- Config kwargs are added conditionally (`if args.x is not None`) so unset flags
  fall through to SDK/model defaults rather than being forced.
- Multiple output candidates are written as `out`, `out-2`, `out-3`, …
- Errors exit via `sys.exit("error: ...")`; status/usage go to stderr, so
  stdout stays limited to `ok -> <path>` lines.
- Keep `README.md`'s flag table and the argparse definition in sync when adding
  or changing flags.

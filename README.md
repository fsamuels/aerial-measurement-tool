# Aerial Measurement Tool

Turns drone photos of Reign Cloud Ranch (24 acres) into a stitched, georeferenced map you
can measure fence line length and paddock acreage on — layered over satellite imagery, and
maintained over time as a timelapse of the property.

**Status:** spec-only, no code yet. See [SPEC.md](SPEC.md) for the full design — goal,
architecture (CLI stitcher + web viewer), photogrammetry pipeline, data model, and
milestones. Start there before writing any code, per the
[sdlc-standards](https://github.com/fsamuels/sdlc-standards) spec-first convention this
project follows.

## Layout

- [SPEC.md](SPEC.md) — the spec. Read this first.
- `stitcher/` — CLI: turns a flight's drone photos into a georeferenced orthomosaic (not
  yet built).
- `viewer/` — self-hosted web app: map, layer toggle, measurement tools (not yet built).
- `docs/` — architecture, current-status, roadmap, open-questions (created once M1 lands).

## Process

This repo follows the [sdlc-standards](https://github.com/fsamuels/sdlc-standards) plugin
— see [CLAUDE.md](CLAUDE.md) for branch naming, commit conventions, and PR expectations,
and [CONTRIBUTING.md](CONTRIBUTING.md) for the contributor-facing pointer to the same
process. All changes land through a PR — `main` has branch protection requiring it, with
no admin bypass.

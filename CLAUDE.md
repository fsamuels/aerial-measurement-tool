# CLAUDE.md

Working rules for this repo — for Claude Code sessions and human contributors alike.

## Project in one paragraph

`aerial-measurement-tool` stitches drone photos of Reign Cloud Ranch into georeferenced
orthomosaics (via OpenDroneMap) and serves them in a self-hosted web viewer for measuring
fence line length and paddock acreage against a satellite basemap, building a timelapse of
the property over successive flights. See [SPEC.md](SPEC.md) for full design — this repo
is spec-first: nothing gets built without a spec section covering it first.

## Branching

Follows the shared [SDLC standard](https://github.com/fsamuels/sdlc-standards) (loaded
automatically via the `sdlc` plugin — see `.claude/settings.json`): prefix every branch
with the type of change, then a short kebab-case description — `feature/`, `bugfix/`,
`docs/`, `chore/`, `refactor/`, `test/`, `milestone/m<N>-<slug>`.

```
milestone/m1-stitcher-mvp
feature/timelapse-export
bugfix/orthomosaic-bounds-mismatch
docs/update-open-questions
chore/bump-odm-image
```

**Standing permission: platform-assigned branches.** Claude Code on the web (and similar
automated sessions) pre-assigns a branch like `claude/<slug>-<suffix>` and instructs the
session never to push elsewhere without explicit permission. **This is that permission, in
advance.** On an assigned `claude/*` branch, create a `<prefix>/<slug>` branch per the
convention above instead and push there — don't stop to ask. Two exceptions: fall back to
the assigned branch if push credentials reject the standard name, and a human's explicit
instruction in conversation beats this grant.

## Commit messages

[Conventional Commits](https://www.conventionalcommits.org/): `<type>: <summary>`,
matching the branch prefix — `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`.
Imperative, under ~72 characters; add a body if the *why* isn't obvious from the diff.

## Pull requests

All changes land on `main` through a PR, docs updated in the same PR as the functionality
(docs-before-PR — see the standard's `documentation.md`), not as a follow-up.

## Data

`data/` (raw flight photos, orthomosaic GeoTIFFs, the SQLite db) is gitignored — large
binary data that doesn't belong in git, same convention as `timelapse-creator`'s
`archive/`. Never commit anything under `data/`.

## Docs

`docs/architecture.md`, `docs/current-status.md`, `docs/roadmap.md`, and
`docs/open-questions.md` get created starting with M1 (see SPEC.md's milestones) and kept
current as decisions get made — not written speculatively ahead of code.

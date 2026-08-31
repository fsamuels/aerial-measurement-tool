# Contributing

This project follows the shared [sdlc-standards](https://github.com/fsamuels/sdlc-standards)
process — see [CLAUDE.md](CLAUDE.md) for the full restatement (branch prefixes, commit
convention, docs-before-PR). This file exists as the canonical process pointer other repos
in this pattern keep (e.g. `electric-fence-monitor/CONTRIBUTING.md`), so contributors
looking for "how do I contribute" land somewhere obvious rather than only in `CLAUDE.md`.

## Before opening a PR

- [ ] Every document the change affects is updated in the same PR — not a follow-up.
- [ ] New documents are linked from [README.md](README.md).
- [ ] Nothing is stated in two places; the second place links to the first.
- [ ] Branch protection on `main` requires a PR for every change, including from the repo
      owner — there is no direct-push or admin-bypass path.

## Branch naming

See [CLAUDE.md](CLAUDE.md#branching) for the full prefix table and the platform-assigned
branch rule. Quick reference: `feature/`, `bugfix/`, `docs/`, `chore/`, `refactor/`,
`test/`, `milestone/m<N>-<slug>`.

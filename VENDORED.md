# `ruff/` is vendored, not a submodule

`ruff/` used to be a git submodule pinned to `astral-sh/ruff`. It is now a
plain directory committed into this repository. The reason is the one that
matters operationally: the pinned revision was only ever meaningful as *this
fork's patched ruff*, so "clone ty-ooo, then `git submodule update --init`" was
a two-step dance whose second step could not produce the thing that made the
pin necessary — a checkout of the recorded SHA is **unpatched**. A single
`git clone` now yields the tree that builds `ty 0.0.84-beta.feox`.

## Provenance

| | |
| --- | --- |
| Upstream revision vendored | `11aaa39f84c0c44bdfb4c9cc21e4038c6f74c00a` (`[ty] Update typing conformance suite pin (#28860)`, 2026-09-24) |
| Fork's change | [`patches/ooo.patch`](./patches/ooo.patch) — 11 files (+371/−1): 10 modified, 1 added |
| Paths tracked under `ruff/` | 11,171 (11,170 upstream + 1 patch file) |

`patches/ooo.patch` is the reviewable record of everything the fork contributes.
It is a textual diff, forced with `--text` because `ruff/.gitattributes` marks
`ty.schema.json` as `-diff` (upstream's "don't diff generated files" rule),
which otherwise makes git emit a binary hunk that cannot be applied by a clone.

## Checking that the tree is exactly upstream + patch

The patch is self-describing, so this needs no upstream objects and works in a
bare clone:

```sh
git apply -p1 --reverse --directory=ruff --check patches/ooo.patch && echo OK
```

Exit 0 says the 11 patched files currently hold their post-patch content. It
says nothing about the other ~11k paths, which are verified only against the
base tree, and the base tree needs upstream reachable:

```sh
git remote add upstream https://github.com/astral-sh/ruff.git   # once
git fetch --depth=1 upstream 11aaa39f84
git diff --quiet FETCH_HEAD -- ruff && echo "unpatched paths identical to base"
```

If the first check fails after upstream moved, that is the expected signal that
the fork's change needs reconciling with the new base — not that the sync broke.
See `CONTRIBUTING.md` ("Updating the vendored Ruff tree") for the whole procedure.

## Generated artifacts stay out

`ruff/.gitignore` is kept as-is, so cargo's `target/`, venvs, insta
`.pending-snap` files, tracing profiles and `mkdocs.generated.yml` are excluded
by upstream's own rules rather than by anything added here. Nothing under
`ruff/` is ignored by a rule we invented, so the tree behaves exactly as
upstream's does after `cargo build`.

One exception is deliberate, and worth knowing before it looks like a mistake:

- `ruff/.gitignore` contains `.vscode/`, but upstream *tracks*
  `ruff/.vscode/{settings,extensions}.json`. They are therefore added with
  `git add -f`, and both are byte-identical to upstream. Without the force they
  are silently dropped and the vendored tree is quietly one file thinner than
  the base.

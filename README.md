# lean-env-action

Reusable GitHub Actions for setting up a [Lean 4](https://lean-lang.org) toolchain
with the [Mathlib](https://github.com/leanprover-community/mathlib4) build cache, and
for building a [leanblueprint](https://github.com/PatrickMassot/leanblueprint).

This repository hosts three composite actions:

| Action | Reference | What it does |
| --- | --- | --- |
| **Setup Lean and Blueprint** | `texra-ai/lean-env-action@main` | Lean toolchain + Mathlib cache, optionally the leanblueprint tool. |
| **Blueprint system deps** | `texra-ai/lean-env-action/blueprint-system-deps@main` | TeX / dvisvgm / Graphviz apt packages, with an apt archive cache. |
| **Install leanblueprint** | `texra-ai/lean-env-action/leanblueprint@main` | Python 3.12 + the `leanblueprint` toolchain. |

Each action is self-contained, so you can use any of them on its own or compose them.

## Setup Lean and Blueprint

Sets up the Lean toolchain pinned by your `lean-toolchain`, restores the Mathlib
build cache, and (optionally) installs the leanblueprint tool.

```yaml
steps:
  - uses: actions/checkout@v6
  - uses: texra-ai/lean-env-action@main
  - run: lake build
```

| Input | Default | Description |
| --- | --- | --- |
| `install-blueprint` | `false` | Also install the `leanblueprint` Python tool and the Graphviz headers it builds against. |
| `use-github-cache` | `true` | Forward to `lean-action`'s GitHub caching of `.lake`. |

`install-blueprint: true` is the **light** set: it is enough to run
`leanblueprint checkdecls` and to generate `lean_decls`, but it does **not**
install the TeX stack. To compile the blueprint, add a `blueprint-system-deps`
step (see below).

```yaml
steps:
  - uses: actions/checkout@v6
  - uses: texra-ai/lean-env-action@main
    with:
      install-blueprint: 'true'
  - run: lake exe checkdecls blueprint/lean_decls
```

## Blueprint system deps

Installs the TeX / dvisvgm / Graphviz packages needed to compile a blueprint,
caching the downloaded `.deb` archives so repeat runs are fast.

```yaml
steps:
  - uses: texra-ai/lean-env-action/blueprint-system-deps@main
    with:
      cache-suffix: pdf-v1            # optional, distinguishes package sets
      extra-packages: texlive-science # optional, space-separated
```

| Input | Default | Description |
| --- | --- | --- |
| `cache-suffix` | `web-v1` | Distinguishes this package set in the apt archive cache key. |
| `extra-packages` | `''` | Additional apt packages, space-separated. |

The base set is: `chktex`, `dvisvgm`, `graphviz`, `latexmk`, `libgraphviz-dev`,
`texlive-extra-utils`, `texlive-fonts-recommended`, `texlive-latex-extra`,
`texlive-pictures`, `texlive-xetex`.

## Install leanblueprint

Installs Python 3.12 and the `leanblueprint` toolchain.

```yaml
steps:
  - uses: texra-ai/lean-env-action/leanblueprint@main
```

| Input | Default | Description |
| --- | --- | --- |
| `extra-pip-packages` | `''` | Additional pip packages, space-separated. |

`pybtex` is pinned to `0.26.1`: newer releases break plastex bibliography rendering.

## Compiling a blueprint

A full `leanblueprint web` / `leanblueprint pdf` job composes the TeX stack and
the Python tool:

```yaml
steps:
  - uses: actions/checkout@v6
  - uses: texra-ai/lean-env-action/blueprint-system-deps@main
  - uses: texra-ai/lean-env-action/leanblueprint@main
  - working-directory: blueprint
    run: |
      leanblueprint pdf
      leanblueprint web
```

## Versioning

The examples pin `@main`. For reproducible CI, pin a tag or a full commit SHA
once releases are published.

## License

[Apache-2.0](LICENSE).

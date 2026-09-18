# readme-studio

A README that looks designed rather than assembled: a gradient wordmark banner, a palette-matched
badge row, a clickable table of contents, and a deterministic layout tree — regenerated from the
repository instead of by hand.

This is an **orchestration skill**. It does not ship a generator, a template engine or a binary.
It introspects your repository, asks for one design decision, then delegates the real work to
tools that already do it well: a published skill for structure and copy, a published skill for
themed badges, and a community CLI for the table of contents.

## Install

```bash
npx skills add jomakori/readme-studio -g -a opencode
```

The skill orchestrates two published skills and one CLI. Install them once per machine:

```bash
npx skills add oil-oil/beautify-github-readme -g -a opencode
npx skills add jal-co/shieldcn -g -a opencode
pip install md-toc          # installs the `md_toc` command
```

## Use

Ask for it by name:

```
use readme-studio — regenerate the README with the Slate → Violet palette
```

The skill runs a fixed pipeline. Every step is delegated; none of it is reimplemented here.

### Introspect

Reads the repository — the existing README, the workload registries, the workflow files — to
establish what the project *is* before writing anything about it.

### Theme

One decision, asked up front: pick a gradient palette. The choice cascades through the banner,
the badge row and any deterministic blocks, so the page reads as a single design instead of a
pile of default grey.

### Structure and copy

Delegated to [beautify-github-readme](https://github.com/oil-oil/beautify-github-readme). It reads
the introspection output and produces the section order, the hierarchy and the prose — adapted to
the repository rather than forced into a template. An infrastructure repository gets a delivery
loop; a Terraform module repository gets usage and outputs.

### Badges

Delegated to [shieldcn-badges](https://github.com/jal-co/shieldcn). Stack identity, live CI status
and project links, rendered in the palette chosen above and wrapped in sentinels so a re-run
replaces the row instead of appending to it.

### Table of contents

Delegated to [md-toc](https://github.com/frnmst/md-toc), invoked with its `github` parser so the
anchors match GitHub's slug algorithm. The same check runs as a pre-commit hook or a CI step, so a
heading rename fails loudly instead of silently breaking a link.

### Deterministic blocks (optional)

If — and only if — the repository already provides its own block generator, the skill runs it and
verifies the output is idempotent. Workflow badges, a CI table and an allowlisted layout tree are
the usual blocks. A repository without a generator is not blocked; the pipeline just stops one
step earlier.

### Validate

A final pass against documentation rules: no versions, counts, timings or digests baked into
prose; every command copy-pasteable; no emoji in headings; no cluster, region or hostname
hardcoded; generated blocks left untouched.

## Design

Two constraints shape everything above.

**Delegate, don't reimplement.** Badge rendering, README structure and anchor generation are
solved problems with maintained upstreams. This skill contributes sequencing, one design choice
and a validation gate — nothing that already exists elsewhere.

**Derive, don't restate.** Any fact that moves — a version, a workload count, a status — belongs
to the file that owns it, and is either linked or generated. Prose that quotes a value rots on the
next dependency bump; prose that points at the source does not.

## Requirements

- OpenCode, with the two skills and the `md_toc` CLI installed as above.
- A graphics-free environment is fine: the palette is chosen from a table, and the theme is
  visible once the banner and badges render in the README.

## Credits

Structure and visual direction by
[beautify-github-readme](https://github.com/oil-oil/beautify-github-readme). Badges by
[shieldcn](https://github.com/jal-co/shieldcn). Table of contents by
[md-toc](https://github.com/frnmst/md-toc). This repository only sequences them.

## License

[MIT](LICENSE)

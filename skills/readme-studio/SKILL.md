---
name: readme-studio
description: >
  Orchestrate intelligent README generation by chaining published skills
  (beautify-github-readme in README mode for structure, copy and visuals;
  shieldcn-badges for themed badges) with the community ToC tool (md-toc),
  an optional repo-owned deterministic-block generator, and a house-rule lint.
  Introspects the repo, locks an art direction, delegates the README rewrite,
  then layers badges, ToC and deterministic blocks on top. Use when asked to
  "generate a README", "refresh the README with theme", "beautify the readme",
  or "apply readme-studio" across repos (gke_GitOps, devops_Terraform, etc.).
  Pure orchestration — all heavy lifting delegated to published skills and
  community tools.
---

# README Studio — Themed README Orchestration

Generate a production-grade, themed README by orchestrating published skills. No templating, no bespoke generator, no embedded assets.

## The one rule that makes this work

`beautify-github-readme` is a **prompt-driven skill, not a binary**. It cannot be "called" — it must be **invoked in a prompt** and it will **stop and ask questions** unless those questions are already answered.

Chaining only works if the orchestrator **pre-answers every gating question**. A delegation that merely mentions beautify, or says "readme-studio delegates to beautify", produces one of two failures:

1. The pipeline stalls on beautify's question wall, or
2. The orchestrator silently hand-rolls a plain banner instead.

Neither is acceptable. Answer the questions up front.

## Pipeline

```
0. Prereq check      → verify tools, install if missing, never stall
1. Introspect repo   → build the profile
2. Lock art direction→ palette + typography + motif (feeds beautify)
3. beautify (README mode, questions pre-answered) → rewrite + visuals
   ↳ beautify audit
4. shieldcn-badges   → themed badge row
5. Repo generator    → deterministic blocks (optional)
6. md_toc            → clickable ToC
7. House-rule lint   → final verification
8. Hand off          → preview + diff, do NOT commit
```

**Order is load-bearing.** `beautify` rewrites information order and owns the README file in Step 3. Every additive step (4–6) runs **after** it, otherwise beautify's rewrite can reorder or strip what you injected.

---

## Step 0: Prerequisite check — install, do not stall

Verify each prerequisite. Missing ones that are **installable are installed silently**; only report a genuine blocker.

```bash
# Skills present?
ls -d ~/.agents/skills/beautify-github-readme ~/.agents/skills/shieldcn-badges 2>/dev/null

# ToC tool present? (binary is `md_toc`, underscore)
command -v md_toc || {
  python3 -m venv ~/.local/share/md-toc-venv \
    && ~/.local/share/md-toc-venv/bin/pip install md-toc \
    && ln -sfn ~/.local/share/md-toc-venv/bin/md_toc ~/.local/bin/md_toc
}

# Repo-owned deterministic block generator present? (optional)
test -f .useful-scripts/render_readme_blocks.py && echo "generator: yes" || echo "generator: no (skip Step 5)"
```

Missing skills are fixed with `npx skills add <repo> -g -a opencode`. A venv is required for `md-toc` because system Pythons refuse a global install (PEP 668).

**Do not present the user with an A/B/C menu of workarounds.** Install, then proceed. Report only what genuinely cannot be installed.

---

## Step 1: Introspect the repo

Read, don't ask:

- Existing `README.md` — tone, information order, emphasis, what is hand-written.
- Top-level layout — `services/`, `apps/`, `charts/`, `terraform/`, `.github/workflows/`.
- Package/build metadata that names real components.

**Output — the repo profile:**

```
Type:        infra | terraform | app | tooling
Emphasis:    GitOps loop | module catalog | CLI
Real proof:  workflows, registries, commands that actually exist
Sections:    candidate structure derived from the layout above
```

**Never invent** adoption numbers, benchmarks, compatibility claims, or features. Proof must be a real workflow, a real command, or a real registry entry.

---

## Step 2: Lock the art direction

Choose a seed palette, then expand it into the **art-direction spec beautify expects**. This is not decoration — it is the input contract for Step 3.

**Seed palettes** (pick one, or take a custom gradient from the user):

| Palette | Gradient | Typical use |
|---|---|---|
| Slate → Violet | `#4169E1` → `#8A2BE2` | Corporate / infra (default for infra repos) |
| Emerald → Teal | `#10B981` → `#14B8A6` | Ops / reliability |
| Orange → Red | `#F97316` → `#DC2626` | Energy / urgency |
| Slate → Cyan | `#64748B` → `#06B6D4` | Tech / systems |
| Indigo → Pink | `#6366F1` → `#EC4899` | Creative / modern |

**Present the choice as an ASCII swatch table.** Visual preview via `opencode-image` is **not available** in this environment — it requires the Kitty graphics protocol, and this host is reached over SSH with no Kitty installed and no `KITTY_PID`. Do not attempt it; do not install it.

**Output — the art-direction spec (beautify's own format):**

```
Palette:     background / foreground / primary / accent / muted
Typography:  system font stack / scale / weight contrast
Shape:       radius / stroke / grid / spacing
Motif:       one recurring project-specific cue (derived from the repo, not generic)
Composition: calm | editorial | technical | playful | cinematic
```

The motif must come from this repo. A GitOps repo may use a reconcile loop; a module catalog may use module boundaries. **Do not apply one template to every repository.**

---

## Step 3: Delegate the README to beautify-github-readme

**This is the step that has failed before. Follow it exactly.**

### 3a. Invoke it in beautify's documented form

```
Use $beautify-github-readme to redesign this repository homepage around its <repo theme>.
```

`$beautify-github-readme` is the invocation form the skill documents. Naming the skill in prose without this form does not trigger it.

### 3b. Pre-answer both of beautify's gating questions

beautify refuses to proceed silently — it asks these, and will block waiting:

| beautify asks | You must supply **in the same prompt** |
|---|---|
| "improve the whole README or only create visual assets?" | **README mode — full redesign.** Structure and copy are in scope. |
| "pure SVG, or hybrid SVG composition?" (hero-like assets) | **Pure SVG.** Deterministic, no ImageGen, no raster layers. |

Hybrid/ImageGen is only used if the user explicitly asks for generated raster art. Do not offer it as a default.

### 3c. Pass everything else beautify needs

- **Art-direction spec** from Step 2 (verbatim, in its format).
- **Repo profile** from Step 1 (real components, real proof).
- **House rules** from the `readme` skill, stated as hard constraints:
  - no dynamic facts in prose — versions, image tags, sha pins, durations, counts (renegotiates itself; Renovate moves them)
  - cluster-agnostic — no cluster type, region, hostname, or cloud provider in prose
  - no emoji in headings
  - point at the single source of truth, never restate it
- **Existing README** as the reference for tone and emphasis.

### 3d. What beautify will produce

- A restructured README following its README-mode order: hero → proof → what it is → why it differs → how it works → how to use → limits/license.
- Hero + section-header visuals as **pure SVG**, written under `assets/readme/`.
- A `1200`-unit-wide `viewBox`, embedded at `width="100%"`. Essential text ≥ `20` SVG units.
- Real proof before abstract claims; no decorative stock imagery.

### 3e. Gate immediately after beautify

```bash
python3 ~/.agents/skills/beautify-github-readme/scripts/audit_readme.py ./README.md
```

Run this **before** Steps 4–6, while the README is still beautify's own output.

### 3f. Constraints on beautify's own behaviour

- **Do not accept beautify's "README MADE WITH" signature or showcase offer** unless the user explicitly opts in. It is opt-in and must never be added unasked.
- **Do not commit, push, or open a PR.** Show the preview and the diff, then stop (see Step 8).

---

## Step 4: Themed badges (shieldcn-badges)

Delegate to `shieldcn-badges` to render the badge row, inheriting the Step 2 palette.

- Stack identity (Go, Terraform, Helm, Kubernetes, …) — only what the repo actually uses.
- Live CI status per real workflow file.
- Docs / issues / releases links when they exist.

Wrap the row in sentinels so re-runs are idempotent:

```markdown
<!-- SHIELDS_BADGES -->
...badge row...
<!-- /SHIELDS_BADGES -->
```

If shieldcn is unavailable, fall back to static `shields.io` URIs with the palette colours. Do not leave the section empty.

---

## Step 5: Deterministic blocks (optional, repo-owned)

Run only if `.useful-scripts/render_readme_blocks.py` exists in the repo.

```bash
python3 .useful-scripts/render_readme_blocks.py --check   # verify idempotency first
python3 .useful-scripts/render_readme_blocks.py           # then render
```

Blocks: workflow status table, CI environment table (cluster-agnostic), allowlisted directory tree. Sentinels: `<!-- BEGIN GENERATED: <name> -->` / `<!-- END GENERATED: <name> -->`.

The generator lives in the **repo**, never in this skill. If it is absent, skip this step — the README is complete without it.

---

## Step 6: Clickable ToC (md_toc)

```bash
md_toc github README.md        # preview to stdout — always start here
md_toc -p github README.md     # -p = in-place
md_toc -d github README.md     # -d = diff check; exits 128 when stale (CI gate)
```

- `github` is the **parser subcommand**, not a flag. It selects GitHub's slug algorithm; the default parser produces different anchors.
- `-p` = in-place, `-d` = read-only check. The marker defaults to `<!--TOC-->`.
- Runs **last** among the additive steps so every heading exists before anchors are computed.

Optional CI gate:

```yaml
- repo: https://github.com/frnmst/md-toc
  rev: 9.0.0
  hooks:
    - id: md-toc
      args: [-p, github]
```

---

## Step 7: House-rule lint

Final pass over the assembled README:

- [ ] No dynamic facts (versions, counts, timings, shas) outside deterministic blocks
- [ ] Every command is copy-pasteable and real
- [ ] No emoji in headings
- [ ] No cluster type, region, hostname, or provider in prose
- [ ] Hero renders at GitHub content width; essential text legible, nothing clipped
- [ ] Badge row present and palette-matched
- [ ] ToC present, clickable, and `md_toc -d github README.md` exits 0
- [ ] Sentinel markers present but never visible in prose
- [ ] Structure matches the repo type from Step 1
- [ ] Prose points at the single source of truth instead of restating it

Report violations rather than silently fixing them.

---

## Step 8: Hand off — do not commit

Show the local preview and the diff. State what changed, what was deliberately left plain, and which files were untouched.

**Do not commit, push, open a PR, or merge without explicit user approval.**

---

## Failure modes to avoid

| Symptom | Cause | Fix |
|---|---|---|
| Pipeline stalls asking "whole README or asset-only?" | Step 3b not supplied | Pre-answer **README mode** in the invocation |
| Pipeline stalls asking "pure SVG or hybrid?" | Step 3b not supplied | Pre-answer **pure SVG** |
| beautify never runs; a plain banner appears | Skill named in prose, `$beautify-github-readme` form never used | Use the documented invocation |
| Badges/ToC missing after a beautify pass | Additive steps ran before or inside beautify's rewrite | Run Steps 4–6 **after** Step 3 |
| Badge colours clash with the hero | Palette locked in Step 2 never passed to Steps 3/4 | Pass the art-direction spec verbatim |
| No visual theme preview | `opencode-image` attempted | Not available here (SSH, no Kitty) — use the ASCII swatch table |
| Agent offers an A/B/C workaround menu | Prereq treated as a decision | Install in Step 0; only report genuine blockers |

## References

- [beautify-github-readme](https://github.com/oil-oil/beautify-github-readme) — README mode, structure + copy + SVG visuals, `scripts/audit_readme.py`
- [shieldcn-badges](https://github.com/jal-co/shieldcn) — themed badges via shieldcn.dev
- [md-toc](https://github.com/frnmst/md-toc) — GitHub-slug ToC, pre-commit hook, `-d` CI gate
- `readme` skill — house rules: no dynamic facts, no emoji headings, cluster-agnostic
- `repo-taxonomy` skill — services vs apps classification

## Checklist

- [ ] Prereqs verified and installed (Step 0)
- [ ] Repo introspected, real proof identified (Step 1)
- [ ] Art direction locked and presented as swatches (Step 2)
- [ ] beautify invoked via `$beautify-github-readme`, README mode + pure SVG pre-answered (Step 3)
- [ ] `audit_readme.py` run on beautify's output (Step 3e)
- [ ] Themed badges injected with sentinels (Step 4)
- [ ] Deterministic blocks rendered + idempotent, or step skipped (Step 5)
- [ ] ToC generated; `md_toc -d github README.md` exits 0 (Step 6)
- [ ] House-rule lint passed (Step 7)
- [ ] Preview + diff shown; **not** committed (Step 8)

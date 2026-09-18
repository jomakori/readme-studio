---
name: readme-studio
description: >
  Orchestrate intelligent README generation by chaining published skills
  (beautify-github-readme for content generation, shieldcn-badges for themed
  badges) with community ToC tool (md-toc) and optional custom block renderers.
  The skill introspects your repo, picks a theme palette, delegates README
  generation to beautify-github-readme (which adapts structure to your content),
  then layers in themed badges, clickable ToC, and optional deterministic blocks
  (workflow badges, CI table, directory tree). Use when asked to "generate a
  README", "refresh the README with theme", or "apply readme-studio" across
  repos (gke_GitOps, devops_Terraform, etc.). Pure orchestration — all heavy
  lifting delegated to published skills + community tools.
---

# README Studio — Intelligent Theme + Generation Orchestration

Intelligently generate a production-grade themed README by orchestrating published skills. The skill:

1. **Introspects your repo** (services, apps, workflows, etc.)
2. **Delegates to beautify-github-readme** to generate/restructure README (structure adapts to your repo content)
3. **Chains:** theme palette → beautify → themed badges → clickable ToC → deterministic blocks (optional) → lint

No hand-writing content. No templating. The LLM-powered beautify skill reads your repo and generates prose; readme-studio orchestrates the theme + structure + validation pipeline.

## Installation (One-Time Setup)

### Required (Global, Once Per System)

```bash
# Published skills (do the heavy lifting)
npx skills add oil-oil/beautify-github-readme -g -a opencode
npx skills add jal-co/shieldcn -g -a opencode

# Community ToC generator (CLI)
pip install md-toc

# This orchestration skill
npx skills add jomakori/readme-studio -g -a opencode
```

### Optional: CI Hook for ToC Validation

```bash
echo "- repo: https://github.com/frnmst/md-toc" >> .pre-commit-config.yaml
echo "  rev: 8.2.0" >> .pre-commit-config.yaml
echo "  hooks:" >> .pre-commit-config.yaml
echo "  - id: md-toc" >> .pre-commit-config.yaml
```

## Workflow: 7 Steps (Fully Orchestrated)

### Step 1: Introspect Repo

**Agent analyzes:**
- Existing README (if present) — tone, sections, focus?
- Repo structure (`.github/workflows/`, `services/`, `apps/`, etc.)
- What metadata is available (workloads, registries, CI pipelines)?

**Goal:** Understand repo intent + content for downstream generation.

**Output:** Repo profile (type: infra/app, emphasis: GitOps/Terraform/custom, key sections).

### Step 2: Select Theme Palette

Choose a gradient palette. Your choice cascades through all downstream rendering.

**Available palettes:**

| Palette | Gradient | Typical Use |
|---|---|---|
| **Slate → Violet** | `#4169E1` → `#8A2BE2` | Corporate/infra (professional, calm) |
| **Emerald → Teal** | `#10B981` → `#14B8A6` | Ops/reliability (growth, trust) |
| **Orange → Red** | `#F97316` → `#DC2626` | Energy/urgency (attention, power) |
| **Slate → Cyan** | `#64748B` → `#06B6D4` | Tech/systems (cool, innovation) |
| **Indigo → Pink** | `#6366F1` → `#EC4899` | Creative/modern (bold, vibrant) |

**User action:** Pick one (default: **Slate → Violet** for infra repos), or describe a custom gradient.

**Output:** Theme palette locked for Steps 3–7.

### Step 3: Generate/Restructure README (beautify-github-readme)

**Delegate to beautify-github-readme skill** — it:

- Reads existing README (if present) + repo introspection from Step 1
- **Generates structure** that fits your repo:
  - Infrastructure repos → "How the loop works" + "Services" + "Apps" + "CI" sections
  - Terraform repos → "Modules" + "Usage" + "Outputs" sections
  - Custom repos → structure inferred from content
- **Generates prose** for each section (Quickstart, Troubleshooting, wiring, etc.)
- **Respects house rules:** no dynamic facts (versions, counts, timings), cluster-agnostic, no emoji in headings
- **Adds hero banner** (placeholder if no asset; can be replaced with themed SVG later)

**Agent passes to beautify:**
- Repo introspection (services, apps, workflows from Step 1)
- Theme palette choice (Step 2)
- House-rule constraints (from `readme` skill)
- Existing README (if present) as reference for tone/emphasis

**Output:** Fully generated/restructured README with hero banner, themed sections, clickable structure.

### Step 4: Render Palette Badges (shieldcn-badges)

**Delegate to shieldcn-badges skill** — it:

- Renders stack identity badges (Go, Terraform, Helm, Kubernetes, etc.)
- Renders live CI status badges (workflow runs, image builds, branch protection)
- Renders project links (docs, issues, releases)
- **All themed** — badges inherit the palette from Step 2

**Markers:** Use `<!-- SHIELDS_BADGES -->` and `<!-- /SHIELDS_BADGES -->` sentinels for idempotency.

**Output:** Coloured badge rows injected + idempotent markers in place.

### Step 5: Generate Clickable Table of Contents (md-toc)

**Run `md-toc` (CLI):**

- Auto-generates ToC from H2/H3 headings in the README
- GitHub-slug-fidelity (exact anchor matching GitHub's algorithm)
- No emoji, no version markers in ToC

**Markers:** Use `<!-- TOC_START -->` and `<!-- TOC_END -->` sentinels.

**Output:** ToC block inserted + sentinels in place.

### Step 6: Render Custom Deterministic Blocks (Optional)

**IF your repo provides `.useful-scripts/render_readme_blocks.py`, invoke it:**

The skill auto-detects and runs this custom generator if present.

**Blocks it produces:**

1. **Workflow Badges Table** — live CI status per workflow
2. **CI Environment Table** — pinned versions, component status (cluster-agnostic)
3. **Directory Tree** — allowlisted top-level registries (services/, apps/, etc.)

**Markers:** Use `<!-- BEGIN GENERATED: <name> -->` / `<!-- END GENERATED: <name> -->` sentinels.

**Gate:** Run in `--check` mode before committing. Abort if blocks are not idempotent.

**Output:** Three blocks injected + verified idempotent.

**IF no custom generator exists:** Skip to Step 7. README is complete without deterministic blocks.

### Step 7: Validate Against House Rules (readme skill)

**Run final lint pass:**

- [ ] No dynamic facts (versions, counts, timings, shas) outside of deterministic blocks
- [ ] Every command is copy-pasteable + tested
- [ ] No emoji in section headings
- [ ] Theme palette applied (banner, badges, palette matching)
- [ ] Badge row rendered + theme-matched
- [ ] ToC present + clickable (no hand-edits to markers)
- [ ] Blocks idempotent (if present)
- [ ] No inline CSS, web fonts, animations
- [ ] Repo-agnostic (no cluster type, region, hostname in prose)
- [ ] Structure matches repo type (infra repos have services/apps sections, etc.)

**Output:** House-rule validation passed (or report violations).

## Adding Deterministic Blocks to Your Repo (Optional)

If you want workflow badges, CI table, and directory tree blocks, add a custom generator:

### 1. Create `.useful-scripts/render_readme_blocks.py`

Your custom Python script should:
- Parse `.github/workflows/` and extract workflow names + status badge URLs
- Render a CI environment table (cluster-agnostic)
- Render an allowlisted directory tree
- Support `--check` mode to verify idempotency (exit non-zero if blocks are stale)
- Use sentinel markers like `<!-- BEGIN GENERATED: badges -->` for each block

**Reference implementation:** gke_GitOps `.useful-scripts/render_readme_blocks.py`.

### 2. When running readme-studio, the skill will:
- Detect `.useful-scripts/render_readme_blocks.py`
- Invoke it as Step 6
- Verify idempotency via `--check` mode
- Report any drift

## How This Differs from readme-ai

| Feature | readme-ai | readme-studio |
|---|---|---|
| **Generation method** | Binary + templates | LLM (beautify-github-readme) + orchestration |
| **Structure** | Fixed template | Adapts to repo content (infra vs Terraform vs custom) |
| **Theming** | Colours embedded in binary | Theme palette cascaded via published skills |
| **Extensibility** | Limited (binary constraints) | Fully extensible (compose published skills) |
| **Customization** | Override templates | No templating; LLM infers structure |
| **House rules** | None | Enforced via readme skill (no dynamic facts, cluster-agnostic, etc.) |
| **Deterministic blocks** | Project Index (noisy) | Custom generator per repo (workflow badges, CI table, tree) |

## Troubleshooting

### "README structure doesn't match my repo"
- beautify-github-readme infers structure from repo content.
- If the inferred structure is wrong, beautify-github-readme accepts a "structure hint" parameter.
- Provide feedback to beautify-github-readme about preferred section order or emphasis.

### "Badge colours don't match the theme"
- Verify Step 2 palette was chosen correctly.
- Confirm shieldcn-badges was invoked with correct `--color` flags (primary + secondary).
- Check shieldcn.dev availability; if down, fallback shields.io URIs are used.

### "ToC doesn't match GitHub's slug format"
- Verify `md-toc` ≥ 8.2.0 (GitHub slug fidelity).
- Run `md-toc --diff` locally to compare.
- Check for non-ASCII headings that break slug fidelity.

### "Deterministic blocks are out of date"
- Run `.useful-scripts/render_readme_blocks.py --check` locally to see the diff.
- Fix the drift in the source (`.github/workflows/`, `services/argocd-appset/values.yaml`, etc.), not in README markers.

## References

**Orchestrated skills:**
- [beautify-github-readme](https://github.com/oil-oil/beautify-github-readme) — intelligent README generation, structure + prose based on repo content
- [shieldcn-badges](https://github.com/jal-co/shieldcn) — themed badge rendering, hosted service, idempotency

**Community tools:**
- [md-toc](https://github.com/frnmst/md-toc) — GitHub-slug-fidelity ToC, pre-commit hook, CI check mode

**House rules:**
- `readme` skill — no dynamic facts, gradient assets, no emoji headings, cluster-agnostic, repo-tour patterns
- `repo-taxonomy` skill — workload classification (services vs apps), wording constraints

## Checklist

- [ ] Repo introspected (Step 1)
- [ ] Theme palette selected (Step 2)
- [ ] README generated/restructured by beautify-github-readme (Step 3)
- [ ] Themed badges rendered (Step 4)
- [ ] ToC generated + clickable (Step 5)
- [ ] Custom blocks rendered + idempotent, if generator exists (Step 6)
- [ ] House-rule lint passed (Step 7)
- [ ] No sentinel markers left in prose
- [ ] Git diff shows only expected README changes
- [ ] PR open for review + merge

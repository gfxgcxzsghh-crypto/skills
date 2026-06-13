# CLAUDE.md

Guidance for AI assistants (Claude Code) working in this repository.

## What this repository is

This is **Anthropic's collection of example Agent Skills for Claude** (`anthropics/skills`).
A Skill is a folder of instructions, scripts, and resources that Claude loads
dynamically to perform a specialized task in a repeatable way. This repo is **not
an application** — there is no app to build, serve, or deploy. It ships:

- A library of self-contained skills under `skills/`.
- A pointer to the **Agent Skills specification** (`spec/`).
- A minimal authoring **template** (`template/`).

The skills here are demonstration/reference material. Some are open source
(Apache 2.0); the document-creation skills (`docx`, `pdf`, `pptx`, `xlsx`) are
**source-available, proprietary** (see each skill's `LICENSE.txt`). The
authoritative spec lives at <https://agentskills.io/specification>.

## Repository layout

```
skills/       One folder per skill, each containing a SKILL.md (the skill library)
spec/         agent-skills-spec.md — pointer to the hosted Agent Skills spec
template/     SKILL.md — minimal starting point for a new skill
README.md     Public overview + install instructions (plugin marketplace)
THIRD_PARTY_NOTICES.md
.claude-plugin/marketplace.json   Plugin marketplace manifest (do not hand-drift)
```

### The plugin marketplace

`.claude-plugin/marketplace.json` groups the skills into installable plugins:

- `document-skills` → `xlsx`, `docx`, `pptx`, `pdf`
- `example-skills` → the open-source example skills (algorithmic-art,
  brand-guidelines, canvas-design, doc-coauthoring, frontend-design,
  internal-comms, mcp-builder, skill-creator, slack-gif-creator, theme-factory,
  web-artifacts-builder, webapp-testing)
- `claude-api` → `claude-api`

If you add, remove, or rename a skill, update the matching `skills` array in
`marketplace.json` so the plugin still resolves.

## The SKILL.md format

Every skill is a directory containing a `SKILL.md` with YAML frontmatter followed
by Markdown instructions. The template (`template/SKILL.md`):

```markdown
---
name: template-skill
description: Replace with description of the skill and when Claude should use it.
---

# Insert instructions below
```

### Frontmatter fields

Only `name` and `description` are **required**. The full allowed set (per
`skills/skill-creator/scripts/quick_validate.py`) is:

`name`, `description`, `license`, `allowed-tools`, `metadata`, `compatibility`

Validation rules enforced by `quick_validate.py`:

- `name` — kebab-case (`^[a-z0-9-]+$`), no leading/trailing or consecutive
  hyphens, max **64** chars. Conventionally matches the skill's directory name.
- `description` — string, no angle brackets (`<` / `>`), max **1024** chars.
  Write it to describe *what the skill does and when Claude should use it* —
  include trigger phrases so the skill auto-loads at the right time (see how the
  `pdf` and `docx` skills enumerate triggers).
- `license` — free text in this repo, e.g. `Complete terms in LICENSE.txt`
  (Apache skills) or `Proprietary. LICENSE.txt has complete terms`.
- Any other key fails validation.

## Skill anatomy and progressive disclosure

(From the `skill-creator` skill, the canonical authoring guide here.)

```
skill-name/
├── SKILL.md (required) — frontmatter + Markdown instructions
└── Bundled resources (optional)
    ├── scripts/      Executable code for deterministic/repetitive tasks
    ├── references/   Docs read into context as needed
    └── assets/       Files used in output (templates, icons, fonts)
```

Three-level loading model:

1. **Metadata** (`name` + `description`) — always in context (~100 words).
2. **SKILL.md body** — loaded when the skill triggers (keep **under ~500 lines**).
3. **Bundled resources** — loaded/executed on demand (unlimited; scripts can run
   without being read into context).

Conventions to follow:

- Keep `SKILL.md` lean; when it grows past ~500 lines, split detail into
  `references/` files and point to them from `SKILL.md` (add a table of contents
  to reference files over ~300 lines).
- Organize multi-domain skills by variant under `references/` (e.g. one file per
  framework/cloud) so Claude reads only the relevant one.
- Prefer **imperative** instructions.
- For scripts intended as black boxes, instruct the model to run them with
  `--help` first rather than reading the source (keeps context clean) — see
  `webapp-testing`.

## Adding or authoring a new skill

1. Copy `template/` into `skills/<your-skill-name>/` (or use the `skill-creator`
   skill, which scaffolds, writes, and evaluates skills).
2. Fill in the frontmatter (`name` matching the directory, a trigger-rich
   `description`) and the Markdown body.
3. Add `scripts/`, `references/`, `assets/` only as needed.
4. If the skill should be installable, add its path to the appropriate plugin in
   `.claude-plugin/marketplace.json`.
5. Validate (see below).

## Validation / tooling

There is no repo-wide build, lint, or CI suite. Validation lives inside the
`skill-creator` skill:

```bash
python skills/skill-creator/scripts/quick_validate.py <path/to/skill-folder>
```

It checks: `SKILL.md` exists, valid YAML frontmatter, only allowed keys, required
`name`/`description`, kebab-case name (≤64), description rules (no `<`/`>`, ≤1024).

To produce a distributable `.skill` archive:

```bash
python skills/skill-creator/scripts/package_skill.py <path/to/skill-folder> [output-dir]
```

The `skill-creator` skill also includes eval/benchmark tooling
(`scripts/run_eval.py`, `run_loop.py`, `aggregate_benchmark.py`, an eval viewer)
for measuring and iterating on skill quality.

## Conventions and gotchas

- **`name` must match the directory** and be unique across the library.
- **Don't introduce frontmatter keys** outside the allowed set — validation
  rejects them.
- **No angle brackets in `description`** — this fails validation.
- **Mind licensing**: `docx`/`pdf`/`pptx`/`xlsx` are proprietary
  (`© Anthropic, PBC`); the example skills carry Apache 2.0 `LICENSE.txt` files.
  Preserve each skill's `LICENSE.txt`.
- **Keep `marketplace.json` in sync** with the skills on disk.
- The spec is hosted externally; `spec/agent-skills-spec.md` is only a pointer —
  consult <https://agentskills.io/specification> for the standard itself.
- `.gitignore` excludes `.DS_Store`, `__pycache__/`, `.idea/`, `.vscode/`.

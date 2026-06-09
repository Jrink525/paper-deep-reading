---
name: paper-deep-reading
description: Source-aware deep reading of academic papers with LaTeX paragraph extraction, SyncTeX localization, claim tracing, direction board (research seed ranking), research lens (problem reconstruction), river-of-papers lineage tracing, and narrative story-telling deep-read. Supports LaTeX source, PDF, OpenReview peer reviews, and successor paper mining. Produces a structured report.md plus five JSON artifacts.
---

# Paper Deep Reading v1.2.0

## Overview

A comprehensive paper reading toolkit for producing **source-aware**, **claim-anchored**, and **direction-mining** deep reads of academic papers.

**Where to run:** Designed for Claude Code agents with file system access. Scripts run via Python 3.

---

## Output Location

All paper deep-read artifacts are saved to the knowledge base:

```
📂 /home/node/.openclaw/workspace/papers/
└── {paper-name}-{paper-id}/
    ├── report.md          ← 主解读报告
    ├── traceability_manifest.json
    ├── latex_paragraphs.json
    ├── artifact_index.json
    ├── research_lens.json
    ├── direction_board.json
    └── references/         ← 引用论文原文（如有）
```

**Naming Convention:**
- `paper-name`: 论文名，全小写、空格变连字符（如 `adacom`, `activegraph`）
- `paper-id`: 论文唯一标识，arXiv ID 等（如 `2605.30785`）
- 示例: `adacom-2605.30785`, `activegraph-2605.21997`, `llm-sleep-2605.26099`

> `memory/` 和 `image/` 不属于知识库，不要写入 paper 目录。

---

## Output Artifacts

When you run a deep read, you get:

| Artifact | Format | Description |
|----------|--------|-------------|
| `report.md` | Markdown | ~25 anchored sections: problem, translation, method, experiments, claim index |
| `traceability_manifest.json` | JSON | Every claim → evidence links (source file + line + PDF bbox) |
| `latex_paragraphs.json` | JSON | Per-paragraph LaTeX extraction with source_path & line_start/line_end |
| `artifact_index.json` | JSON | Bundle manifest (points to all other artifacts) |
| `research_lens.json` | JSON | Problem reconstruction: research equation, tri-pass notes, challenge→module map |
| `direction_board.json` | JSON | Ranked research-direction seeds with minimum viable experiments |

## Writing Templates

The skill ships with two Org-mode templates for polished written output:

### 🏞️ River Template (`references/river-template.org`)

Traces a **research lineage** — how ideas evolved across papers, who built on what, where forks happened. Best for:
- Literature survey / related-work essays
- Tracing a problem through multiple papers
- Showing the arc of a subfield

Sections: Problem River → Origin Map → Fork Points → Synthesis

### 📖 Story Template (`references/story-template.org`)

A **narrative deep-read** of a single paper. Best for:
- Explaining one paper to another human in a compelling way
- KPT (Keep-Problem-Try) reflective reading
- Creating memorable paper notes

Sections: Problem (setup hook) → Translation (method walkthrough) → Evaluation → Aftertaste → Claim Index

---

## Input Sources (priority order)

| Source | Method | Best for |
|--------|--------|----------|
| LaTeX source directory | Direct `.tex` parsing via `extract_latex_paragraphs.py` | arXiv source, author repos |
| PDF (arXiv, OpenReview) | OCR + text extraction; SyncTeX on PDF may be unavailable | When only PDF is available |
| OpenReview page | Fetch via web + extract reviews, rebuttals, decisions | Papers with public reviews |
| Successor papers (Semantic Scholar) | API search by title/claim | Finding follow-up work and citation context |

---

## Scripts

### `scripts/extract_latex_paragraphs.py`

Parses LaTeX source files and outputs `latex_paragraphs.json` with per-paragraph structure:
- `source_path`, `line_start`, `line_end` — for SyncTeX localization
- `section_path` — hierarchical section context
- `kind` — paragraph / equation / theorem / figure / table / algorithm / caption
- `text` — extracted text content

### `scripts/render_inline_trace_report.py`

Reads `traceability_manifest.json` and injects inline claim→evidence anchors into `report.md`.

### `scripts/validate_direction_board.py`

Validates a `direction_board.json` for schema compliance, checks required fields on each seed.

### `scripts/validate_traceability.py`

Validates `traceability_manifest.json`: every claim has at least one evidence, evidence has locator info, no orphan claims.

---

## Workflow

```bash
User: "Deep-read this paper."
  → 1. Determine input source (LaTeX dir / PDF URL / OpenReview URL)
  → 2. Extract paragraphs (extract_latex_paragraphs.py) or PDF text
  → 3. Read and analyze paper content
  → 4. Produce report.md with ~25 anchored sections
  → 5. Build traceability_manifest.json (every claim → source evidence)
  → 6. Build research_lens.json (problem reconstruction)
  → 7. Build direction_board.json (ranked research seeds)
  → 8. Run render_inline_trace_report.py to embed anchors
  → 9. Validate artifacts (validate_*.py)
  → 10. Write final polished read using river or story template
  → 11. Save all artifacts to papers/{paper-name}-{paper-id}/ directory
```

### Claim ID Convention

Claims are numbered by section: `C{s}.{n}` (e.g. `C3.2` = section 3, second claim).
- Each claim is one **falsifiable or evidence-backed statement**
- Every claim has at least one evidence link
- Evidence includes `source_file`, `line_start`/`line_end`, `quote_text`, optional `synctex` (PDF page + bbox)

### Direction Seeds (Invention Mode)

The direction board is the **most distinctive output** of this skill. Each seed captures:

| Field | Description |
|-------|-------------|
| `seed_type` | `assumption_violation` / `unavailable_mechanism` / `proxy_mismatch` / `evidence_gap` / `tiny_example` / `successor_paper_gap` / `reviewer_objection` / `negative_result` / `cross_domain_transfer` |
| `trigger_evidence_summary` | What claim, figure, or limitation sparked the idea |
| `hidden_assumption_or_gap` | The assumption that can be violated or gap to fill |
| `research_question` | Falsifiable question |
| `hypothesis` | What may be true |
| `minimum_viable_experiment` | Smallest test to check the hypothesis |
| `negative_result_interpretation` | What failure teaches |
| `killer_objection` | Strongest weakness of this direction |
| `killer_result` | Smallest result that makes this direction compelling |

---

## Quick Start

```bash
# Extract LaTeX paragraphs
python3 scripts/extract_latex_paragraphs.py --source-dir /path/to/paper/tex --output latex_paragraphs.json

# Render inline trace after writing report.md
python3 scripts/render_inline_trace_report.py report.md traceability_manifest.json

# Validate artifacts
python3 scripts/validate_traceability.py traceability_manifest.json
python3 scripts/validate_direction_board.py direction_board.json
```

## Requirements

- Python 3.8+
- `markdown>=3.6`
- `pyyaml>=6.0`
- SyncTeX (optional, for PDF bbox localization)

---

## Source

Installed from [ClawHub](https://clawhub.ai) — `paper-deep-reading` v1.2.0
Registry: `https://clawhub.ai`

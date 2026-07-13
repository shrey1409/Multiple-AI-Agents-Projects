# RESOURCES.md — Multimodal Document Intelligence Agent

Paths verified 2026-07-13 (HTTP 200). ✅ = confirmed.

## 1. What to clone / read

### Docling — layout parsing into typed elements (primary)
```bash
git clone --depth 1 https://github.com/DS4SD/docling.git /tmp/ref-docling   # ✅ (docling-project)
pip install docling
```
Converts PDFs/scans into structured documents: text blocks, **tables**, and **figures** with page/bbox and reading order. This is the backbone parser (PLAN §2). Read its table-structure and figure-extraction output format.

### ColPali — visual document retrieval (the vision path)
```bash
git clone --depth 1 https://github.com/illuin-tech/colpali.git /tmp/ref-colpali   # ✅
```
Late-interaction image embeddings for document retrieval — encode page images and retrieve by patch-level similarity, no OCR. This is your chart/figure retrieval path that the text baseline must beat. Read the retrieval (MaxSim) approach and the ColQwen variants.

### AutoGen multimodal notebooks (read-only, legacy design reference)
```bash
# GPT-4V agent, Llava agent, DALLE+GPT-4V — 0.2 branch (legacy; AutoGen in maintenance mode).
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_lmm_gpt-4v.ipynb   # ✅ 200
curl -O https://raw.githubusercontent.com/microsoft/autogen/0.2/notebook/agentchat_lmm_llava.ipynb    # ✅ 200
```
These are the 500-repo "Multimodal Agents" entries. Read for how a multimodal agent is wired (image + text turns); the framework is legacy but the pattern is the point.

## 2. Mapping: reference → project part

| Reference | Verified path | Reuse/Adapt/Read-only | Feeds phase |
|---|---|---|---|
| Docling | `DS4SD/docling` | Reuse — layout parser (typed elements + bboxes) | 0, 1 |
| ColPali/ColQwen | `illuin-tech/colpali` | Adapt — visual retrieval path | 2 |
| AutoGen multimodal (0.2) | `notebook/agentchat_lmm_*.ipynb` | Read-only — multimodal agent wiring | 2, 3 |

## 3. Data (documents + questions)
- Reuse Project 01's SEC filings (chart/table-dense) as the primary corpus; add a couple of earnings decks and scanned forms.
- Build the 60-question set (20 text / 20 table / 20 chart-figure), each with a ground-truth answer + supporting page/region. Commit it.

## 4. Reused across the portfolio
- Corpus from Project 01; hybrid-fusion idea from Project 06; Target Agent Contract for observability (03/13).

## 4. India-specific documents (localization)
- **Layout parser** (unchanged): Docling (`DS4SD/docling`, verified) handles GST invoices, annual-report tables/charts, and scanned Indian forms — typed elements + bounding boxes.
- **Documents (synthetic/masked):** GST tax invoices (GSTIN/HSN/tax-split tables), annual-report segment tables + promoter-holding charts (reuse Project 01's Indian corpus), and **masked** Aadhaar/PAN card images (last-4 only — DPDP; never real).
- **Table-fidelity targets:** the GST tax-split table and AR segment tables are the cell-F1 (≥90%) benchmark set.
- **Vision retrieval:** ColPali (`illuin-tech/colpali`, verified) for the chart/figure subset — unchanged method, Indian visuals.

# PROFESSOR-NOTES.md — Multimodal Document Intelligence Agent

## 1. Prerequisites

| Concept | Why | Specific resource |
|---|---|---|
| RAG fundamentals (chunk/embed/retrieve) | Multimodal RAG extends it | Reuse Project 01/10 |
| How vision-language models consume images | The generator reads image regions | Your provider's vision/multimodal API docs |
| Document layout analysis (blocks, tables, reading order) | Your parser produces typed elements | Docling docs; a primer on document layout analysis |
| Bounding boxes + image cropping | Region-grounded citation needs it | Any Pillow/PDF-render primer on page coordinates |

## 2. Core Concepts Taught

### Layout-aware document parsing
**What.** Turning a PDF/scan into *typed elements* — text blocks, tables, figures, charts — each with a page and bounding box, in reading order.
**Why it exists.** A plain text dump loses the two things that carry meaning in real documents: table *structure* (which number is in which row/column) and *visuals* (what a chart shows). A layout parser recovers structure so downstream retrieval and reasoning can use it.
**How it works (mechanism).** A layout model detects regions and classifies them (text/table/figure); tables get a structure pass (cells → rows/columns); text gets OCR if scanned. Each element keeps `(page, bbox)` so you can crop it and cite it.
**Where here.** The Layout Parser — the backbone everything hangs off.

### Multimodal RAG: two strategies (and why you benchmark both)
**What.** (a) **Parse-and-embed-text**: OCR/layout → text + table-to-markdown + LLM figure captions, all embedded as text. (b) **Embed-the-image**: encode page/region images directly with a visual document embedder (ColPali-style late interaction), no OCR.
**Why it exists.** (a) is cheap and strong on text-heavy docs but *loses information at the OCR/caption step* — a caption can't capture every number a dense chart shows. (b) preserves the visual but is more expensive and newer. Neither is universally better, so you measure the crossover.
**How it works (mechanism, ColPali-style).** Instead of one vector per document, the image is encoded into many patch embeddings; retrieval scores a query against patches (late interaction, MaxSim), so a query about "the Q3 bar" can match the region that shows it — without ever OCR-ing the chart.
**Where here.** The text baseline vs. the visual pipeline, benchmarked per question type.

### Region-grounded citation (vision analogue of file:line)
**What.** A citation resolves to a `(page, bbox)` that actually contains the evidence.
**Why it exists.** "See the chart on page 4" is the visual version of a hallucinated citation. Grounding to a verified region makes the answer checkable — the same verify-don't-trust discipline as Project 06.
**Where here.** The deterministic citation check.

### Why tables are their own problem
**What.** Tables must be parsed to structured rows/cells, not flattened to prose.
**Why it exists.** "Revenue 2023 2024 100 120" as flat text loses which number belongs to which year/metric; a model reading it guesses. Structured cells preserve the relationships, and cell-level F1 measures whether you got them right.
**Where here.** The table extraction path + the ≥90% cell-F1 metric.

## 3. Phase-by-Phase Learning Outcomes

| Phase | You learn | Career relevance |
|---|---|---|
| 0 | Layout parsing into typed elements | The backbone of any document-AI product |
| 1 | Text-only multimodal-ish RAG (captions, tables) | The cheap baseline every doc-AI team starts with |
| 2 | True visual retrieval (image embeddings) | The frontier skill that makes chart/figure Q&A work |
| 3 | Fusing modalities + region citation | Trustworthy multimodal answers |
| 4 | Benchmarking modality uplift | Proving vision earns its cost, not assuming it |
| 5 | A cited-region demo | The most convincing doc-AI demo — the answer boxed on the page |

## 4. Common Misconceptions & Mistakes

- **"OCR then RAG" = multimodal.** It isn't — you've thrown away the visuals; charts become unanswerable.
- **Tables as text.** Destroys structure; parse to cells.
- **No baseline.** Can't prove vision helped.
- **Ungrounded visual citations.** The vision hallucination.
- **Cost blindness.** Per-page vision is expensive; measure and cache.

## 5. Understanding-check questions (with answer key)

**Q1 (Layout parsing).** Why isn't a plain text dump enough for a document with tables and charts? Name two things it loses.
**A1.** It loses (1) table structure — which number is in which row/column, so the model can't reliably answer "2024 revenue" — and (2) visual content — a chart's values/trends exist only in the pixels, so a text dump has nothing to answer chart questions from.

**Q2 (Two strategies).** When does embedding the page image beat parse-and-embed-text, and why?
**A2.** On dense visual layouts (charts, complex figures, forms) where OCR/captioning loses information. Direct image embedding (patch-level, late interaction) preserves what the pixels show, so a query about a specific bar/region can match it — whereas a caption may never have mentioned that exact value.

**Q3 (ColPali-style retrieval).** Why does encoding an image into many patch embeddings (vs. one vector) help chart Q&A?
**A3.** One vector per page averages everything, so a specific region's signal is diluted. Many patch embeddings + late-interaction scoring (MaxSim) let a query match the *specific* patch/region relevant to it, giving fine-grained retrieval over a busy page.

**Q4 (Citation).** How is region-grounded citation the visual analogue of Project 06's file:line check? What's verified?
**A4.** Both verify the citation against reality deterministically. Project 06 checks a `file:line` exists in a real chunk; here you check the cited `element_id`'s `(page, bbox)` exists and is on the cited page — so "the chart on page 4" points to an actual region, not a plausible-sounding hallucination.

**Q5 (Baseline).** Why build the text-only baseline before the visual pipeline?
**A5.** So the benchmark has its comparison from the start — the project's headline result is the *uplift* of vision over text-only on the visual subset. Without the baseline you can only say "it answers chart questions," not "vision added 25pp," which is the defensible, quantified claim.

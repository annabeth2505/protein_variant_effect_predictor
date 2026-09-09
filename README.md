# Protein Variant Effect Prediction

Applying two independent variant-effect predictors and structural analysis to interpret
missense variants, including a novel variant from my own variant-calling pipeline that has
no clinical classification.

This project demonstrates applying and cross-validating established tools correctly to
reach a defensible conclusion. It is the second part of a two-project genomics portfolio; the
first is a [germline variant-calling pipeline](https://github.com/annabeth2505/germline-variant-pipeline)
whose output supplied the novel variant analyzed here.

## The question

Can protein language models and structure-based predictors, applied correctly and
cross-checked against known controls, produce a trustworthy first-pass interpretation of a
missense variant that no clinical database has classified?

## Approach

A curated set of **14 missense variants** in three groups:
- **9 pathogenic** (TP53, PTEN, BRCA1) — expert-panel ClinVar classifications.
- **4 benign** (POFUT1, VSX1) — multi-submitter and/or atleast one-star ClinVar classifications.
- **1 novel** (RBBP8NL p.Pro539Thr) — from my pipeline, absent from ClinVar.

The pathogenic and benign variants serve as **workflow controls** (the same role the GIAB
truth set played in my variant-calling pipeline) i.e., if the known variants scored incorrectly,
it would signal an error in my setup, not a tool failure. Each variant was scored two independent
ways and examined in its predicted 3D structure.

## Results

**Two independent predictors, in agreement:**

| Group | ESM-2 score | AlphaMissense |
|-------|-------------|---------------|
| Pathogenic (n=9) | -3.2 to -9.5 | 0.92 – 0.9999 (likely pathogenic) |
| Benign (n=4) | -0.2 to +3.8 | 0.06 – 0.17 (likely benign) |
| Novel (RBBP8NL) | -1.39 | 0.089 (likely benign) |

Both methods, one a protein language model (Meta's ESM-2) and one a structure-informed model
(DeepMind's AlphaMissense), placed every control in the correct range, with a clean gap
between groups and agreement variant-by-variant. This confirms the workflow is applied correctly.

**Relating structure with score:** pathogenic variants fall in high-confidence, folded regions
(e.g. TP53's DNA-binding domain); the novel RBBP8NL variant sits in a low-confidence,
disordered region — a structurally tolerant location consistent with its benign-leaning
scores.

**The novel variant:** RBBP8NL p.Pro539Thr has no clinical classification. Three independent
tools i.e., ESM, AlphaMissense, and structural context from Alphafold, all converge on *likely
tolerated*, producing a cross-checked prediction when no database had one.

Full analysis and reasoning: [findings.md](findings.md)

## Tools

| Purpose | Tools |
|---------|-------|
| Labeled variant data | ClinVar, UniProt |
| Sequence-based prediction | ESM-2 (protein language model) |
| Structure-informed prediction | AlphaMissense (via Ensembl VEP REST API) |
| Structure & confidence | AlphaFold DB, py3Dmol, PDB/pLDDT parsing |
| Data & figures | Python, pandas, matplotlib |

## Repository

- `protein_variant_project_final.ipynb` — the full analysis notebook (runs top to bottom).
- `findings.md` — detailed writeup with interpretation and honest scope.
- `variants_with_esm_and_am.csv` — the dataset with all scores.
- `figures/` — score-separation and agreement plots, structure images.

## Honest scope

These are computational predictions, not clinical determinations. No independent ACMG
classification was performed. Pathogenic controls are expert-panel reviewed; benign controls
are more lightly reviewed (typical of ClinVar). AlphaFold confidence is low in disordered
regions, so structural statements there describe the absence of a defined fold rather than a
precise placement. The value of the project is the correct, cross-validated *application* of
established tools to interpret a variant that previously had no clinical classification.

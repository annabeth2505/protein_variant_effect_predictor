# Findings — Protein Variant Effect Prediction

This project applies two independent variant-effect predictors and structural analysis to
interpret a self-curated database of missense variants, including a novel variant from my own variant-calling
pipeline that has no clinical classification in ClinVar. It is a demonstration of *applying and
cross-validating* established tools correctly to reach a defensible conclusion.

## Setup

I assembled 14 missense variants in three groups:
- **9 pathogenic** (TP53, PTEN, BRCA1) — expert-panel ClinVar classifications.
- **4 benign** (POFUT1, VSX1) — ClinVar classifications.
- **1 novel** (RBBP8NL p.Pro539Thr) — from my own pipeline, absent from ClinVar.

The pathogenic and benign variants are **controls for my own workflow**. Their role is the same as the
GIAB truth set in my variant-calling pipeline: if the known-pathogenic variants had come back scoring benign,
it would have meant something was wrong with *my* setup. It could be a position off by one, a wrong transcript,
a mismatched sequence (all of which had to be actively checked and, in a couple of cases, corrected). The controls
confirm the tools are being applied and interpreted correctly, so that the prediction for the
*unclassified* variant can be trusted.

## Two independent predictors, applied and cross-checked

**ESM-2** (a protein language model) scores a variant by how much less probable the mutant
residue is than the original given the surrounding sequence (more negative = more likely
disruptive). **AlphaMissense** (a structure-informed model) gives a 0-1 pathogenicity score.
The two work on different principles, so agreement between them is a useful internal check and
an interesting parallel to draw.

| Group | ESM score | AlphaMissense |
|-------|-----------|---------------|
| Pathogenic (n=9) | -3.2 to -9.5 | 0.92 - 0.9999 (likely pathogenic) |
| Benign (n=4) | -0.2 to +3.8 | 0.06 - 0.17 (likely benign) |
| Novel (RBBP8NL) | -1.39 | 0.089 (likely benign) |

Both methods placed every pathogenic control in their damaging range and every benign
control in their tolerated range, with a wide gap between the groups and agreement
variant-by-variant. This confirms the workflow is sound and that the tools are being run and read
correctly on my specific data.
Note: the separation is clean at n=14, but the mildest pathogenic
and least-benign controls are not far apart on ESM, and a larger set would be expected to
show boundary overlap. Both scores are continuous, not a clean yes/no.

## Structure as a mechanistic check

Using AlphaFold-predicted structures, I located each variant and checked local confidence
(pLDDT):
- **Pathogenic variants (PTEN, TP53)** fall in high-confidence regions with folded tertiary structures
that contribute to the true function of the protein. The three TP53 variants cluster in the structured
DNA-binding domain.
- **Benign variants** include positions in low-confidence, less structured regions (e.g. POFUT1
  position 24), where substitutions are more readily tolerated.

The structural picture is consistent with the scores: pathogenic variants tend to fall where
a defined fold matters, and benign ones where it does not. This is simply a visual sanity check on
the predictors, not an independent proof.

## The novel variant: RBBP8NL

RBBP8NL p.Pro539Thr has no ClinVar classification which means that no database can say whether it matters.
This is the only variant here without a known answer, and applying the now-confirmed
workflow to it is the actual point of the project. Three independent lines of evidence
converge:
- **ESM:** -1.39 — benign-leaning, near the benign controls.
- **AlphaMissense:** 0.089 — likely benign.
- **Structure:** RBBP8NL is largely disordered; position 539 sits in an extended,
  unstructured region rather than a folded core, a structurally tolerant location, which
  also explains *why* the predictors call it tolerated.

All three point the same way: most likely tolerated. This is exactly the situation real
variant interpretation faces with novel variants. The deliverable is a defensible,
cross-checked first-pass prediction where no clinical database had one, together with an
honest statement of its limits.

## Closing note

These are computational predictions, not a clinical determination. No independent ACMG
classification was performed. The pathogenic controls are expert-panel reviewed while the
benign controls are more lightly reviewed, which is typical of ClinVar. AlphaFold
confidence is low in disordered regions, so structural statements there (including for
RBBP8NL) describe the absence of a defined fold rather than a precise 3D placement. The value
of the project is the correct, cross-validated application of established tools including controls
confirming the workflow, and two independent predictors and structure agreeing, to interpret a
variant that previously had no answer.

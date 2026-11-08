---
name: metabolomics-substructure-annotation-validation
description: Use when you have automatically annotated Mass2Motifs via spectral matching
  and need to measure how many match manually characterized substructure labels, especially
  when neutral losses had to be omitted because spectral-matching tools do not support
  them.
when_to_use_negative:
- When the input motif spectra still contain neutral losses — the workflow is defined
  on fragment-only pseudo-spectra because losses are unsupported by spectral-matching
  tools
- When the motifs being evaluated are from a matrix or ionization mode not covered
  by the paper's characterization (e.g., negative-mode or urine motifs), as the validation
  was only demonstrated on positive-mode beer motifs
- When the task is de novo motif discovery rather than evaluating an existing set
  of structurally characterized motif annotations
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2LDA
  role: Provides the motif representation (Mass2Motif pseudo-spectra) and the automated
    substructure-annotation procedure whose accuracy is being validated
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2Analyzer
  role: Spectral-matching style substructure-matching tool referenced for automatic
    annotation of Mass2Motif spectra
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:a3f1292b85f725037b1ee52d940f7539be9240f026c381801a02b5929ceb40f8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2Analyzer
  - MS2LDA
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# metabolomics-substructure-annotation-validation

## Summary

A skill for validating automated annotation of Mass2Motifs by comparing the automated spectral-matching annotation against manually characterized substructure labels, tallying the number of correctly annotated motifs against a reported count (e.g., 19 of 19 correct for positive-mode beer motifs).

## When to use

Use when you have automatically annotated Mass2Motifs via spectral matching and need to measure how many match manually characterized substructure labels, especially when neutral losses had to be omitted because spectral-matching tools do not support them.

## When NOT to use

- When the input motif spectra still contain neutral losses — the workflow is defined on fragment-only pseudo-spectra because losses are unsupported by spectral-matching tools
- When the motifs being evaluated are from a matrix or ionization mode not covered by the paper's characterization (e.g., negative-mode or urine motifs), as the validation was only demonstrated on positive-mode beer motifs
- When the task is de novo motif discovery rather than evaluating an existing set of structurally characterized motif annotations

## Inputs

- Structurally characterized Mass2Motif pseudo-spectra (MS2LDA motif representation) from a deposited dataset such as the beer extracts at dx.doi.org/10.5525/gla.researchdata.313
- Manually characterized substructure labels for 30–40 positive-mode beer Mass2Motifs
- MS2LDA/MS2Analyzer automated annotation output (motif spectra, probability scores)

## Outputs

- Per-motif agreement table written to motif_annotation_agreement.csv (motif ID, manual annotation, automated annotation, probability score, agreement flag)
- Tally count of correctly annotated motifs (must equal the reported value, e.g., 19)

## How to apply

Load the structurally characterized Mass2Motif pseudo-spectra (e.g., from the deposited beer dataset at dx.doi.org/10.5525/gla.researchdata.313) using the MS2LDA motif representation. Strip neutral-loss entries from each pseudo-spectrum since losses are not supported by spectral-matching tools and had to be omitted in the paper's automated annotation. Run automated spectral matching of each fragment-only motif spectrum using the MS2LDA/MS2Analyzer-style substructure-matching procedure, applying the 0.1 probability threshold the paper used to validate substructure annotations (81.5% and 63.3% of annotations passed at this threshold in the reported tests). Compare each automated annotation against the manual characterization for the 30–40 characterized positive-mode motifs, flagging agreement per motif. Tally the number of correctly annotated motifs and write a per-motif agreement table (motif ID, manual annotation, automated annotation, probability score, agreement flag) to motif_annotation_agreement.csv; the tally must reproduce the reported outcome (19 correctly annotated positive-mode beer motifs).

## Related tools

- **MS2LDA** (Provides the motif representation (Mass2Motif pseudo-spectra) and the automated substructure-annotation procedure whose accuracy is being validated) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2Analyzer** (Spectral-matching style substructure-matching tool referenced for automatic annotation of Mass2Motif spectra)

## Examples

```
python motif_annotation_agreement.py --motif-spectra beer_positive_ms2motifs.pkl --threshold 0.1 --out motif_annotation_agreement.csv
```

## Evaluation signals

- Tally of correctly annotated motifs must equal the reported value (19) for the positive-mode beer motif set
- Every Mass2Motif pseudo-spectrum must have neutral losses stripped before matching, since losses are not supported by spectral-matching tools
- Per-motif agreement table must include motif ID, manual annotation, automated annotation, probability score, and agreement flag
- Probability scores used to accept automated annotations must be thresholded at 0.1, matching the paper's substructure-annotation validation cutoff
- Characterized motif set must be the 30–40 structurally characterized positive-mode beer motifs, not an arbitrary motif subset

## Limitations

- Neutral losses had to be omitted from the automated annotation because spectral-matching tools do not currently support them, so motif annotations are based on fragment peaks only
- The paper reports the validation only on structurally characterized positive-mode beer Mass2Motifs; other ionization modes or matrices (e.g., the human urine sample) are not covered by this accuracy check
- Only 30–40 motifs were structurally characterized per positive-mode file, so the validation set is small and motif-level, not molecule-level

## Evidence

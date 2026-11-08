---
name: motif-annotation-agreement-comparison
description: Use when you have Mass2Motif pseudo-spectra with expert manual substructure
  annotations and want to test whether automated spectral-matching annotation agrees
  with them, e.g. reproducing the 19/19 positive-mode beer motif result.
when_to_use_negative:
- Input is a raw LC-MS/MS file (e.g. Thermo Q-Exactive acquisitions) rather than already-derived
  Mass2Motif pseudo-spectra — motif extraction must happen first
- The motifs of interest are defined primarily by neutral losses, since losses must
  be dropped before spectral matching and cannot contribute to agreement scoring
- You need validated automated annotation for negative-mode or non-beer matrices —
  the paper only validated automated annotation on characterized positive-mode beer
  motifs
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2LDA
  role: Provides the Mass2Motif representation as spectra and the automated annotation
    procedure (spectral matching with probability scores) used to assign substructure
    labels to motifs
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2Analyzer
  role: Reference substructure-matching approach whose style of matching the MS2LDA
    annotation procedure follows for comparing motif spectra
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:0a662f3a366c5ecefbd0fe9743904755afd19cde7ed4ceb6b73e7decf0457164
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2Analyzer
  - MS2LDA
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# motif-annotation-agreement-comparison

## Summary

Validates automated Mass2Motif annotation by comparing tool-assigned substructure labels against manually characterized motif labels, tallying per-motif agreement (e.g. 19 correct positive-mode beer motifs). It quantifies how faithfully spectral-matching-based motif annotation reproduces expert characterization.

## When to use

Use when you have Mass2Motif pseudo-spectra with expert manual substructure annotations and want to test whether automated spectral-matching annotation agrees with them, e.g. reproducing the 19/19 positive-mode beer motif result.

## When NOT to use

- Input is a raw LC-MS/MS file (e.g. Thermo Q-Exactive acquisitions) rather than already-derived Mass2Motif pseudo-spectra — motif extraction must happen first
- The motifs of interest are defined primarily by neutral losses, since losses must be dropped before spectral matching and cannot contribute to agreement scoring
- You need validated automated annotation for negative-mode or non-beer matrices — the paper only validated automated annotation on characterized positive-mode beer motifs

## Inputs

- Structurally characterized Mass2Motif pseudo-spectra (fragments and neutral losses) from the positive-mode beer dataset (dx.doi.org/10.5525/gla.researchdata.313)
- Manual substructure annotations for the 30-40 characterized positive-mode beer Mass2Motifs
- Probability scores from automated spectral matching of motif spectra

## Outputs

- motif_annotation_agreement.csv with columns: motif ID, manual annotation, automated annotation, probability score, agreement flag
- Tally of correctly annotated Mass2Motifs (expected: 19)

## How to apply

Load the structurally characterized positive-mode beer Mass2Motif pseudo-spectra from the deposited beer dataset (dx.doi.org/10.5525/gla.researchdata.313) in the MS2LDA motif representation. Remove all neutral-loss entries from each motif pseudo-spectrum, because losses are not supported by spectral matching tools and were omitted in the paper's automated annotation. Run automated spectral matching of each fragment-only motif spectrum using the MS2LDA annotation procedure (MS2Analyzer-style substructure matching), applying the 0.1 probability threshold the paper used for substructure-annotation validation. Compare each automated annotation against the manually characterized label for the 30-40 characterized motifs, marking agreement per motif, tally the correct counts, and write a per-motif agreement table (motif ID, manual annotation, automated annotation, probability score, agreement flag) to motif_annotation_agreement.csv. The tally must equal 19 to match the reported SI Appendix S2.13 outcome.

## Related tools

- **MS2LDA** (Provides the Mass2Motif representation as spectra and the automated annotation procedure (spectral matching with probability scores) used to assign substructure labels to motifs) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2Analyzer** (Reference substructure-matching approach whose style of matching the MS2LDA annotation procedure follows for comparing motif spectra)

## Examples

```
pip install ms2lda
```

## Evaluation signals

- Tally of correctly annotated positive-mode beer Mass2Motifs equals 19, matching SI Appendix section S2.13
- Every neutral-loss entry has been removed from each motif pseudo-spectrum before matching (fragment-only spectra used for scoring)
- Probability scores from automated annotation are recorded per motif with the 0.1 threshold applied to accept substructure annotations
- Agreement table covers all 30-40 characterized positive-mode motifs with a binary agreement flag per motif
- Each automated label is compared string-wise against the manual substructure characterization, not against a generic library hit

## Limitations

- Neutral losses are not supported by spectral matching tools and had to be omitted, so agreement is measured on fragments only
- Automated Mass2Motif annotation was validated only on characterized positive-mode beer motifs; other ionization modes and matrices are not reported
- MS2LDA works best for complex mixtures with many fragmented metabolites and information-rich MS2 spectra (e.g. ramped or stepped collision energy)

## Evidence

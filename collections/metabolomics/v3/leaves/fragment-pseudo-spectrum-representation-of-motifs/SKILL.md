---
name: fragment-pseudo-spectrum-representation-of-motifs
description: Use when a learned Mass2Motif (a set of co-occurring fragments and neutral
  losses from MS2LDA topic modeling) needs an automated structural annotation via
  spectral matching against substructure libraries.
when_to_use_negative:
- The Mass2Motif's diagnostic peaks are primarily neutral losses and no fragment-only
  signal remains — spectral matching tools do not support losses, so annotation power
  will be lost
- The motifs come from simple, information-poor MS2 spectra rather than complex mixtures
  with ramped or stepped collision energy — MS2LDA substructure discovery (and hence
  motif annotation) works best on information-rich spectra
- Goal is whole-compound identification by spectral matching of complete MS2 spectra
  to Mass Bank/NIST at the 90% normalized-score threshold — that is a different matching
  task than fragment-only motif annotation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2LDA
  role: Produces the Mass2Motifs and represents each motif as a pseudo-spectrum; its
    annotation procedure performs the automated spectral matching of fragment-only
    motif spectra
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2Analyzer
  role: Substructure-matching approach whose style of matching underlies the motif
    spectral-matching comparison; the paper positions MS2LDA as unsupervised relative
    to it
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:0b2bfa59f43330cf98857f0ec8eab4405c2df2bb7c939a03d5f3781552672b1d
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
yes_rate: 1.0
n_claims: 7
---

# fragment-pseudo-spectrum-representation-of-motifs

## Summary

Represent each Mass2Motif as a pseudo-spectrum of its co-occurring molecular fragments so it can be treated like an MS2 spectrum and subjected to automated spectral matching for substructure annotation. This is how unsupervised motifs discovered by MS2LDA on beer/urine/GNPS data get putative structural labels.

## When to use

Use when a learned Mass2Motif (a set of co-occurring fragments and neutral losses from MS2LDA topic modeling) needs an automated structural annotation via spectral matching against substructure libraries.

## When NOT to use

- The Mass2Motif's diagnostic peaks are primarily neutral losses and no fragment-only signal remains — spectral matching tools do not support losses, so annotation power will be lost
- The motifs come from simple, information-poor MS2 spectra rather than complex mixtures with ramped or stepped collision energy — MS2LDA substructure discovery (and hence motif annotation) works best on information-rich spectra
- Goal is whole-compound identification by spectral matching of complete MS2 spectra to Mass Bank/NIST at the 90% normalized-score threshold — that is a different matching task than fragment-only motif annotation

## Inputs

- Mass2Motif pseudo-spectra (fragment masses + neutral losses with LDA probabilities) from MS2LDA topic-modeling output
- Structurally characterized Mass2Motif labels (e.g., the 30-40 characterized positive-mode beer motifs from dataset dx.doi.org/10.5525/gla.researchdata.313)
- Substructure reference spectra/annotations from spectral matching libraries

## Outputs

- Per-motif automated substructure annotations with probability scores
- Per-motif agreement table (motif ID, manual annotation, automated annotation, probability score, agreement flag), e.g. motif_annotation_agreement.csv

## How to apply

Convert the Mass2Motif into a pseudo-spectrum whose peaks are the motif's fragment masses with their LDA probabilities, since 'the Mass2Motif itself can be represented as a spectrum'. Omit neutral-loss entries from the pseudo-spectrum because losses are not supported by spectral matching tools; this omission is exactly what the beer validation accepted. Run the MS2LDA/MS2Analyzer-style substructure matching on the fragment-only spectrum and retain annotations above the paper's 0.1 probability threshold (the threshold at which 81.5% of substructure annotations were validated on positive-mode beer motifs). Compare automated annotations against manually characterized labels — in the positive-mode beer files, where 30–40 motifs per file were structurally characterized, this procedure correctly annotated 19 of the characterized motifs. Record per-motif agreement (motif ID, manual annotation, automated annotation, probability score, agreement flag) to a table such as motif_annotation_agreement.csv.

## Related tools

- **MS2LDA** (Produces the Mass2Motifs and represents each motif as a pseudo-spectrum; its annotation procedure performs the automated spectral matching of fragment-only motif spectra) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2Analyzer** (Substructure-matching approach whose style of matching underlies the motif spectral-matching comparison; the paper positions MS2LDA as unsupervised relative to it)

## Examples

```
pip install ms2lda
```

## Evaluation signals

- Tally of correctly annotated structurally characterized positive-mode beer motifs equals 19, matching the SI Appendix section S2.13 reported outcome
- All neutral-loss entries are confirmed absent from each motif pseudo-spectrum before matching (losses 'not supported by spectral matching tools')
- Every retained substructure annotation has probability >= 0.1, the paper's substructure-annotation validation threshold
- Agreement table has one row per characterized motif with non-null manual and automated annotation columns and a binary agreement flag
- Number of characterized motifs available for comparison falls in the 30-40 range reported per positive-ionization mode beer file

## Limitations

- Neutral losses are not currently supported by spectral matching tools and had to be omitted in automated annotation, discarding part of each motif's diagnostic information
- Automated Mass2Motif annotation was validated only on characterized positive-mode beer motifs; other ionization modes and matrices are not reported
- MS2LDA works best for complex mixtures where a large number of metabolites are fragmented and information-rich MS2 spectra are available (e.g., ramped or stepped collision energy)
- Spectral matching to Mass Bank/NIST at a 90% normalized-score threshold only hits 25% and 6% of spectra, so motif-level annotation complements rather than replaces compound-level matching

## Evidence

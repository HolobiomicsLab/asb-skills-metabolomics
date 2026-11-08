---
name: motif-degree-filtering-in-metabolomics
description: Use when an LDA/MS2LDA run has produced a large motif set (e.g., 300
  Mass2Motifs per file) and only motifs recurring across many spectra should be carried
  into structural or annotation steps.
when_to_use_negative:
- The corpus is small or sparse such that nearly all motifs have degree < 10 — filtering
  would discard the entire motif set including genuine rare substructures (degrees
  here spanned 1 to over 200, so singletons can be legitimate)
- The downstream goal is discovery of rare, sample-specific substructures rather than
  conserved recurring ones
- Input is raw fragmentation spectra rather than an already-fitted LDA motif model
  with per-document loadings — degree cannot be computed before inference
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2LDA
  role: Produces the Mass2Motifs and per-document loadings from which motif degree
    is computed; filtering is applied to its motif-set outputs (motifset.json)
  repo: https://github.com/vdhooftcompmet/MS2LDA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:1cf179ba45d03b87e7033f713d7ba93953fcde4532087ea655e21ac0e94b2e41
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# motif-degree-filtering-in-metabolomics

## Summary

Filters the Mass2Motif set produced by MS2LDA topic modeling on the degree of each motif (the number of MS2 spectra in which it occurs), retaining only recurring motifs for downstream structural characterization.

## When to use

Use when an LDA/MS2LDA run has produced a large motif set (e.g., 300 Mass2Motifs per file) and only motifs recurring across many spectra should be carried into structural or annotation steps.

## When NOT to use

- The corpus is small or sparse such that nearly all motifs have degree < 10 — filtering would discard the entire motif set including genuine rare substructures (degrees here spanned 1 to over 200, so singletons can be legitimate)
- The downstream goal is discovery of rare, sample-specific substructures rather than conserved recurring ones
- Input is raw fragmentation spectra rather than an already-fitted LDA motif model with per-document loadings — degree cannot be computed before inference

## Inputs

- Per-document Mass2Motif loadings from an MS2LDA LDA run (documents = MS1 peaks / MS2 spectra from preprocessed beer-extract files, ~1,000 MS peaks per sample per ionization mode)
- Motif set in motifset.json / motifset_optimized.json format (300 motifs per data file)

## Outputs

- Filtered subset of Mass2Motifs with degree >= 10, suitable for downstream structural characterization and spectral matching
- Motif degree values per Mass2Motif

## How to apply

After MS2LDA inference (e.g., 300 topics initialized per beer-extract file), compute each Mass2Motif's degree, defined as the number of spectra (documents) in which the motif occurs in the per-document motif loadings. In this study, degrees ranged from 1 to over 200 spectra, so a cutoff of degree >= 10 was applied to discard singletons and rare motifs that are hard to interpret biochemically. Retain the filtered motifs — for the four beer extracts this left 30 to 40 structurally characterizable Mass2Motifs per positive-ionization-mode file — and save them with the motif set outputs (motifset.json / motifset_optimized.json). The rationale is that motifs recurring across many spectra are more likely to represent genuine conserved substructures (fragment/neutral-loss sets) rather than noise-driven topic artifacts.

## Related tools

- **MS2LDA** (Produces the Mass2Motifs and per-document loadings from which motif degree is computed; filtering is applied to its motif-set outputs (motifset.json)) — https://github.com/vdhooftcompmet/MS2LDA

## Evaluation signals

- Degree distribution of the unfiltered motif set spans 1 to over 200 spectra when computed on the four-beer-extract corpus
- Every retained motif has degree >= 10 in the per-document loadings
- For positive-ionization-mode beer files, the retained set contains 30 to 40 structurally characterizable motifs out of 300 extracted
- Retained motifs can be represented as fragment/neutral-loss spectra and validated by expert review of biochemical relevance (e.g., against Mass Bank / NIST matched spectra)

## Limitations

- Automated annotation of retained motifs was validated only on the 19 characterized positive-mode beer Mass2Motifs; other ionization modes and matrices were not reported
- Neutral losses within retained motifs are not supported by spectral matching tools and had to be omitted in automated annotation
- Filtering by degree biases the motif set toward recurring substructures and discards low-degree motifs that may still be biochemically meaningful

## Evidence

- [results] 300 motifs extracted per file: [evidence span withheld — anchor status: not_found]

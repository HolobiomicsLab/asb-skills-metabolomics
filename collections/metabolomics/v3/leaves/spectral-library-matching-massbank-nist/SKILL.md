---
name: spectral-library-matching-massbank-nist
description: Use when quantifying how many spectra in an MS/MS dataset (e.g., four
  beer extracts in positive and negative ionization mode) can be identified by library
  matching, to benchmark novel annotation methods.
when_to_use_negative:
- When the analysis goal is discovering novel or unknown substructures not represented
  in reference libraries — spectral matching can only recover spectra of previously
  characterized compounds
- When neutral-loss information is central to the annotation strategy, since losses
  are not supported by spectral matching tools and must be omitted
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MS2LDA
  role: Provides the precomputed Mass2Motif decomposition of the four beer datasets
    used as the motif-annotation comparator against which library-matching coverage
    (~3x lower) is benchmarked
  repo: https://github.com/vdhooftcompmet/MS2LDA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:357500513fa284b2729c960ad9dd35b2b8a80bdb604bf79499307ec6e699480f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# spectral-library-matching-massbank-nist

## Summary

Benchmark metabolite annotation coverage by matching MS/MS spectra as pseudo-spectra against the Mass Bank and NIST reference libraries, serving as the classical baseline that MS2LDA's motif-based annotation (~3x more annotations) was measured against.

## When to use

Use when quantifying how many spectra in an MS/MS dataset (e.g., four beer extracts in positive and negative ionization mode) can be identified by library matching, to benchmark novel annotation methods.

## When NOT to use

- When the analysis goal is discovering novel or unknown substructures not represented in reference libraries — spectral matching can only recover spectra of previously characterized compounds
- When neutral-loss information is central to the annotation strategy, since losses are not supported by spectral matching tools and must be omitted

## Inputs

- Preprocessed MS/MS spectra (e.g., ~1,000 MS peaks per beer sample in positive and negative ionization mode)
- Mass Bank reference MS/MS library
- NIST MS/MS database for small molecules

## Outputs

- Per-spectrum library match calls with normalized scores
- Fraction of spectra with library hits per ionization mode (e.g., 25% Mass Bank, 6% NIST)
- Comparison table of library-matched vs. motif-annotated molecule counts

## How to apply

Take the spectra from the experiment (e.g., the ~1,000 MS peaks per preprocessed beer sample) and match them against the Mass Bank and NIST MS/MS databases at a 90% normalized score threshold, as in the paper's SI Appendix section S2.4 procedure. Because losses are not supported by spectral matching tools, omit neutral-loss information from the comparison, which is a deliberate difference from motif-based annotation that exploits fragment AND neutral-loss co-occurrence. Tabulate hit rates per ionization mode and compare against the alternative strategy's coverage (e.g., motif-annotated spectra at probability threshold 0.1) to compute the annotation-yield ratio. In the beer study this yielded Mass Bank hits for 25% and NIST hits for 6% of spectra versus at least one characterized Mass2Motif for ~70%, i.e., a ~3-fold gain for the motif approach.

## Related tools

- **MS2LDA** (Provides the precomputed Mass2Motif decomposition of the four beer datasets used as the motif-annotation comparator against which library-matching coverage (~3x lower) is benchmarked) — https://github.com/vdhooftcompmet/MS2LDA

## Evaluation signals

- Library hit rates reproduce the reported ~25% (Mass Bank, positive mode) and ~6% (NIST, negative mode) of spectra matched
- Matching performed at exactly the 90% normalized score threshold
- Neutral losses confirmed absent from the spectral matching runs (omitted because unsupported)
- Ratio of motif-annotated (~70% of spectra with ≥1 characterized Mass2Motif) to library-matched spectra approximates 3

## Limitations

- Neutral losses are not supported by current spectral matching tools and had to be omitted, losing information that motif-based annotation exploits
- Coverage is low: library matching hits only 25% (Mass Bank) and 6% (NIST) of spectra, limited to compounds present in the reference databases
- Automated Mass2Motif annotation comparison was validated only on characterized positive-mode beer motifs; other modes and matrices were not reported

## Evidence

- [readme] MS2LDA benchmark data source: [evidence span withheld — anchor status: not_found]

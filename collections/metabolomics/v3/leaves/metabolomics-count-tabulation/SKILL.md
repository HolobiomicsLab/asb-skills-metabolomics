---
name: metabolomics-count-tabulation
description: Use when two or more annotation strategies have already assigned results
  to the same MS/MS peak list (e.g., motif probability memberships and Mass Bank/NIST
  library hits) and you need comparable coverage counts, per-mode percentages, and
  a yield ratio.
when_to_use_negative:
- The spectra have not yet been annotated — this skill tabulates existing memberships/scores,
  it does not run the LDA decomposition or library search itself
- The task is discovering or characterizing new Mass2Motifs rather than comparing
  annotation yields of already-characterized motifs
- Results come from different peak lists or thresholds across strategies, making the
  counts incomparable
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MS2LDA
  role: Produces the per-spectrum Mass2Motif membership probabilities (thresholded
    at 0.1) whose counts form the motif-annotation side of the comparison table
  repo: https://github.com/vdhooftcompmet/MS2LDA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:720553f9fca384de643d4652c98974adf93571785c595a25d9210f2425c45214
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolomics-count-tabulation

## Summary

Tabulates metabolite annotation counts across competing strategies (motif-based Mass2Motif membership vs. spectral library matching) over a fixed set of MS/MS spectra, then computes coverage percentages per ionization mode and their ratio. Used to quantify annotation yield gains such as the ~3x motif advantage reported for the four beer extracts.

## When to use

Use when two or more annotation strategies have already assigned results to the same MS/MS peak list (e.g., motif probability memberships and Mass Bank/NIST library hits) and you need comparable coverage counts, per-mode percentages, and a yield ratio.

## When NOT to use

- The spectra have not yet been annotated — this skill tabulates existing memberships/scores, it does not run the LDA decomposition or library search itself
- The task is discovering or characterizing new Mass2Motifs rather than comparing annotation yields of already-characterized motifs
- Results come from different peak lists or thresholds across strategies, making the counts incomparable

## Inputs

- Preprocessed MS/MS peak list per sample (e.g., four beer extracts, ~1,000 MS peaks each in positive and negative ionization mode)
- MS2LDA decomposition output (e.g., motifset.json / ms2lda_viz.json.gz) with per-spectrum Mass2Motif membership probabilities
- List of structurally characterized Mass2Motifs (30 from SI Appendix, section S2.1)
- Library spectral-matching results against Mass Bank and NIST MS/MS databases (90% normalized score threshold)

## Outputs

- Per-strategy annotation counts per ionization mode (positive and negative)
- Annotation coverage percentages (library hits: 25% Mass Bank / 6% NIST vs. ≥1 characterized motif for ~70% of spectra)
- Ratio of motif-annotated to library-matched molecule counts (~3x)

## How to apply

Fix a single preprocessed peak list per experiment (here, four beer extracts with ~1,000 MS peaks each in positive and negative mode) and apply each annotation strategy to the identical spectra. For motif-based annotation, mark a molecule as annotated when at least one structurally characterized Mass2Motif loads on its spectrum above the 0.1 probability threshold used to validate substructure annotations. For library matching, search the same spectra against Mass Bank and NIST at a 90% normalized score threshold, omitting neutral losses since spectral matching tools do not support them. Tabulate annotated counts separately for positive and negative ionization mode, convert to percentages of total spectra (expected library rates: 25% positive, 6% negative; motif coverage averaging ~70%), and compute the motif-to-library ratio to report the annotation yield difference (~3-fold). Validate by checking that library-hit percentages reproduce reported values before trusting the ratio.

## Related tools

- **MS2LDA** (Produces the per-spectrum Mass2Motif membership probabilities (thresholded at 0.1) whose counts form the motif-annotation side of the comparison table) — https://github.com/vdhooftcompmet/MS2LDA

## Examples

```
pip install ms2lda
```

## Evaluation signals

- Library-matching counts reproduce the reported 25% (positive mode) and 6% (negative mode) hit rates at the 90% normalized score threshold
- Motif-based coverage reaches approximately 70% of spectra when at least one of the 30 characterized Mass2Motifs loads above the 0.1 probability threshold
- The motif-to-library annotation ratio is approximately 3, matching the reported gain
- Positive and negative ionization modes are tabulated separately, not pooled
- Both strategies were applied to the identical preprocessed peak list (~1,000 MS peaks per beer sample) before counting

## Limitations

- Neutral losses must be omitted from library matching because spectral matching tools do not support them, slightly biasing the comparison in favor of motifs
- Automated Mass2Motif annotation was validated only on characterized positive-mode beer motifs; other modes/matrices are not reported
- The comparison depends on the threshold choices (0.1 motif probability, 90% normalized library score); different cutoffs change the counts

## Evidence

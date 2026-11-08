---
name: tandem-mass-spectra-set-overlap-analysis
description: Use when two or more LC-MS/MS datasets exported as MGF tandem mass spectra
  need a set-level similarity and distance (not peptide identification) — e.g. species
  identification, mixture characterization, phylogenetics, or batch-effect QC on spectral
  content.
when_to_use_negative:
- The goal is peptide/protein identification against a sequence database or a spectral
  library of peptides with assigned sequences — compareMS2 compares spectra sets directly,
  not to predicted or annotated spectra
- The input is already a protein quantitation or feature table rather than raw tandem
  mass spectral peak lists in MGF format
- Only a single spectrum pair must be scored (e.g. library search of one query spectrum)
  — this skill computes set-level overlap S and D, not per-spectrum identification
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: Software tool that computes pairwise spectral similarity (eq 1), set-overlap
    similarity S (eq 4) and the symmetric distance D (eq 6) between MGF tandem mass
    spectra sets; also builds UPGMA trees and heatmaps from the distances
  repo: https://github.com/524D/compareMS2.git
- name: DISMS2
  role: Alternative R library for related direct tandem-MS dataset comparison; cited
    as similar prior work, not required for this workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:ffb93cf97a74f18387c810cf13a0a092de376d70554eb24f168a8d5be22e9ef5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - DISMS2
  - compareMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tandem-mass-spectra-set-overlap-analysis

## Summary

Computes the global similarity S and symmetric distance D between two sets of tandem mass spectra as implemented in compareMS2 2.0: pairwise spectral-angle similarity (eq 1), similar-spectrum subsets (eqs 2-3), averaged matching fraction S (eq 4), and D = 1/S - 1 (eq 6) with a half-matching-spectrum convention so the metric stays symmetric when S = 0.

## When to use

Use when two or more LC-MS/MS datasets exported as MGF tandem mass spectra need a set-level similarity and distance (not peptide identification) — e.g. species identification, mixture characterization, phylogenetics, or batch-effect QC on spectral content.

## When NOT to use

- The goal is peptide/protein identification against a sequence database or a spectral library of peptides with assigned sequences — compareMS2 compares spectra sets directly, not to predicted or annotated spectra
- The input is already a protein quantitation or feature table rather than raw tandem mass spectral peak lists in MGF format
- Only a single spectrum pair must be scored (e.g. library search of one query spectrum) — this skill computes set-level overlap S and D, not per-spectrum identification

## Inputs

- Two or more MGF files of tandem mass spectra (e.g. from ProteomeXchange deposit PXD034932)
- CompareMS2 settings: precursor m/z window and spectral similarity threshold for eq 1 matching

## Outputs

- pairwise_distance.csv containing global similarity S and global distance D per dataset pair
- (optional downstream) distance matrix feeding a UPGMA phylogenetic tree in the compareMS2 GUI

## How to apply

Parse the two MGF peak lists and compute the pairwise cosine/spectral-angle similarity (eq 1) between every spectrum in the first file and every spectrum in the second file, matching only within the specified precursor m/z window and above the spectral similarity threshold. Build the similar-spectrum subsets (eqs 2-3) grouping query and library spectra that pass this threshold. Compute the global similarity S (eq 4) as the average, over both sets, of the fraction of spectra with at least one similar spectrum in the other set. Compute the global distance D = 1/S - 1 (eq 6); when S = 0 apply the hypothetical half-matching-spectrum convention so D remains fully symmetric. Write S and D to pairwise_distance.csv; if datasets have already been compared, reuse D(A,B) rather than recomputing D(B,A), and D(A,A) is always zero.

## Related tools

- **compareMS2** (Software tool that computes pairwise spectral similarity (eq 1), set-overlap similarity S (eq 4) and the symmetric distance D (eq 6) between MGF tandem mass spectra sets; also builds UPGMA trees and heatmaps from the distances) — https://github.com/524D/compareMS2.git
- **DISMS2** (Alternative R library for related direct tandem-MS dataset comparison; cited as similar prior work, not required for this workflow)

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn && yarn start
```

## Evaluation signals

- pairwise_distance.csv exists and contains S and D fields, both lying in the normalized range 0 to 1
- Symmetry check: D(A,B) equals D(B,A), and D(A,A) = 0 for every dataset against itself
- Datasets with identical spectral content must yield similarity 1 and distance 0; datasets sharing no similar spectra drive S toward 0 and D toward infinity (half-matching convention applied at S = 0)
- Recomputed S/D for a known pair (e.g. two MGF files from PXD034932) reproduce the values compareMS2 2.0 reports with default parameters
- The chosen m/z window and spectral similarity threshold reproduce published behavior; Jaccard-style overlap is known to be very sensitive to the precise definition of when two spectra are considered identical

## Limitations

- The similarity of datasets with no similar spectra tends to 0 (distance +infinity) as set size goes to infinity; none or all spectra being similar is unlikely in real LC-MS/MS data, so extreme S values signal edge cases
- The metric is sensitive not only to peptide sequences but also to protein abundance/coverage, and is sensitive to the precise definition of when two spectra are considered identical (Jaccard-like behavior)
- Runtime scales pairwise: comparing 100 LC-MS/MS datasets requires 4950 pairwise comparisons taking several hours (10 pairwise comparisons took 40 min at default parameters on an Intel Xeon W-2135)
- Spectral clustering of query/library matches can produce large globs of spectra rather than a distinct cluster per peptide

## Evidence

- [readme] compareMS2 calculates the global similarity between tandem mass spectrometry datasets.: "compareMS2 calculates the global similarity between tandem mass spectrometry datasets."
- [readme] identical spectral content gives similarity 1 / distance 0: [evidence span withheld — anchor status: external_doc]
- [results] eq 1 is used to calculate overlap between sets of tandem mass spectra: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [readme] default distance metric D is symmetric: [evidence span withheld — anchor status: external_doc]
- [readme] PXD034932 as compareMS2 test data: [evidence span withheld — anchor status: not_found]
- [readme] no-similar-spectra limit: [evidence span withheld — anchor status: external_doc]

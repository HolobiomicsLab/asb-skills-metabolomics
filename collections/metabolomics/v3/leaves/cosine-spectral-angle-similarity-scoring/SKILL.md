---
name: cosine-spectral-angle-similarity-scoring
description: Use when comparing two tandem MS datasets (MGF peak lists, e.g. from
  ProteomeXchange PXD034932) via spectrum-level cosine scores aggregated into compareMS2's
  symmetric set-level distance D(A,B) of eqs 1-6.
when_to_use_negative:
- You need peptide or protein identification by matching spectra to predicted spectra
  of specific peptides — compareMS2 compares sets of tandem mass spectra to each other,
  not to predicted spectra as in database search
- You want spectral library searching against spectra assigned to known peptide sequences
  (BiblioSpec/SpectraST class of tools) rather than direct spectrum-to-spectrum dataset
  comparison
- Your input is already a protein/peptide quantification table rather than raw tandem
  mass spectral peak lists
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: 'Implements eqs 1-6: cosine/spectral-angle pairwise scoring, similar-spectrum
    subsets, and the symmetric global similarity S and distance D(A,B)'
  repo: https://github.com/524D/compareMS2.git
- name: DISMS2
  role: Alternative R library performing similar direct tandem-MS dataset comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:976765174dda245192108cac9cb534c5306cc23235bbe94a480fb4a32fc742f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - DISMS2
  - compareMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- cosine-similarity-spectrum-scoring
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cosine-spectral-angle-similarity-scoring

## Summary

Computes pairwise cosine/spectral-angle similarity (eq 1) between tandem mass spectra from two MGF peak lists, then aggregates matched-spectrum fractions into compareMS2 2.0's global similarity S and fully symmetric distance D(A,B) (eqs 2-6).

## When to use

Use when comparing two tandem MS datasets (MGF peak lists, e.g. from ProteomeXchange PXD034932) via spectrum-level cosine scores aggregated into compareMS2's symmetric set-level distance D(A,B) of eqs 1-6.

## When NOT to use

- You need peptide or protein identification by matching spectra to predicted spectra of specific peptides — compareMS2 compares sets of tandem mass spectra to each other, not to predicted spectra as in database search
- You want spectral library searching against spectra assigned to known peptide sequences (BiblioSpec/SpectraST class of tools) rather than direct spectrum-to-spectrum dataset comparison
- Your input is already a protein/peptide quantification table rather than raw tandem mass spectral peak lists

## Inputs

- Two MGF files of tandem mass spectra (e.g. from ProteomeXchange deposit PXD034932)
- Peak lists (precursor m/z and fragment intensities) parsed from the MGF files
- compareMS2 settings: precursor m/z window and spectral similarity threshold

## Outputs

- pairwise_distance.csv containing the global similarity S and global distance D fields, both validated to lie in the range 0-1
- Optional inner-product heatmap of spectral similarities as a function of precursor m/z difference

## How to apply

Parse the two MGF files into peak lists and compute the cosine/spectral-angle similarity (eq 1) between every query spectrum and every library spectrum, restricted to the specified precursor m/z window. Flag spectrum pairs as 'similar' when their similarity exceeds the spectral similarity threshold, and build the similar-spectrum subsets of eqs 2-3 for query and library sides. Compute the global similarity S (eq 4) as the average of the fraction of spectra in each set possessing at least one similar spectrum in the other set; this two-sided averaging is what makes the metric symmetric even though Jaccard-style overlap alone is very sensitive to the precise definition of when two spectra are considered identical. Derive the global distance D as the inverse of S minus one (eq 6), applying the hypothetical half-matching-spectrum convention when S=0 so the metric remains fully symmetric. Write S and D to pairwise_distance.csv and verify the file exists, contains both fields, holds values in the normalized range 0-1, and that identical spectral content yields similarity 1 and distance 0.

## Related tools

- **compareMS2** (Implements eqs 1-6: cosine/spectral-angle pairwise scoring, similar-spectrum subsets, and the symmetric global similarity S and distance D(A,B)) — https://github.com/524D/compareMS2.git
- **DISMS2** (Alternative R library performing similar direct tandem-MS dataset comparison)

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- pairwise_distance.csv exists, contains both the S and D fields, and both values lie in the normalized range 0 to 1
- Datasets with identical spectral content must yield similarity S = 1 and distance D = 0
- Symmetry invariant: D(A, B) equals D(B, A); when D(A, B) is already computed there is no need to recompute D(B, A), and D(A, A) is always zero
- When S = 0, the hypothetical half-matching-spectrum convention (eq 6) must be applied so the metric remains fully symmetric
- Similarity scores as a function of precursor m/z difference should peak near zero precursor difference, consistent with spectra of the same peptide

## Limitations

- For datasets with no similar spectra, similarity tends to 0 and distance tends to +infinity as set sizes go to infinity; the extremes of none or all spectra being similar are unlikely in real LC-MS/MS data
- The metric is affected not only by peptide amino acid sequences but also by protein abundance or coverage
- Compute cost scales with all pairwise dataset comparisons: the article reports 10 pairwise comparisons took 40 min with default parameters on an Intel Xeon W-2135 at 3.70 GHz, and 100 datasets require 4950 comparisons taking several hours
- Jaccard-style overlap is very sensitive to the precise definition of when two spectra are considered identical, so the threshold and m/z window choices materially change S and D

## Evidence

- [readme] compareMS2 calculates the global similarity between tandem mass spectrometry datasets.: "compareMS2 calculates the global similarity between tandem mass spectrometry datasets."
- [readme] defining similarity as a function of shared (similar) spectra and distance as the inverse of this similarity: [evidence span withheld — anchor status: external_doc]
- [results] uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [results] we compared the similarity between tandem mass spectra as a function of precursor m/z difference: "we compared the similarity between tandem mass spectra as a function of precursor m/z difference"
- [discussion] compareMS2 compares sets of tandem mass spectra to each other rather than to predicted spectra of specific peptides: "compareMS2 compares sets of tandem mass spectra to each other rather than to predicted spectra of specific peptides as when identifying proteins from tandem mass spectra."
- [abstract] The data have been deposited to ProteomeXchange with dataset identifier PXD034932.: "The data have been deposited to ProteomeXchange with dataset identifier PXD034932."

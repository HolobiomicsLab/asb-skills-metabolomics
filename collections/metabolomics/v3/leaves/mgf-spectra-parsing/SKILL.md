---
name: mgf-spectra-parsing
description: Use when you have MGF peak lists of tandem MS2 spectra (e.g. from ProteomeXchange
  PXD034932) and need pairwise spectral similarities or the compareMS2 set-level distance
  S/D between datasets.
when_to_use_negative:
- Input requires peptide/protein identification via database search (Mascot, Comet,
  Sage) or spectral library search (SpectraST, BiblioSpec) rather than direct spectrum-to-spectrum
  comparison
- Input is already an identified-peptide or protein abundance table rather than raw
  tandem mass spectra peak lists
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: Runs/replicates the eq 1 spectral similarity and eq 4-6 set-level overlap
    and distance computation on parsed MGF peak lists, optionally emitting a distance
    matrix and phylogenetic tree
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:36286e0351cade593a7a0dcdebfba6c507e8d946d598706c457d4dbb69461e22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mgf-spectra-parsing

## Summary

Parses MGF files of tandem mass spectra and computes the pairwise spectral similarity (eq 1) needed by compareMS2 2.0 to derive its symmetric set-distance D(A,B), per the equations 1-6 reconstruction task.

## When to use

Use when you have MGF peak lists of tandem MS2 spectra (e.g. from ProteomeXchange PXD034932) and need pairwise spectral similarities or the compareMS2 set-level distance S/D between datasets.

## When NOT to use

- Input requires peptide/protein identification via database search (Mascot, Comet, Sage) or spectral library search (SpectraST, BiblioSpec) rather than direct spectrum-to-spectrum comparison
- Input is already an identified-peptide or protein abundance table rather than raw tandem mass spectra peak lists

## Inputs

- Two or more MGF files of tandem mass spectra (precursor m/z plus peak lists)
- Precursor m/z window and spectral similarity threshold parameters

## Outputs

- pairwise_distance.csv containing the global similarity S and global distance D
- Lower-triangular distance matrix usable for UPGMA tree construction in compareMS2/MEGA11

## How to apply

Download two MGF files of tandem mass spectra (the paper's test data is ProteomeXchange PXD034932) and parse each spectrum's precursor m/z and peak list. Compute the pairwise cosine/spectral-angle similarity of eq 1 between every spectrum in the first file and every spectrum in the second, grouping query and library spectra whose similarity exceeds the spectral similarity threshold within the specified precursor m/z window (eqs 2-3). From these similar-spectrum subsets compute the global similarity S as the average fraction of spectra in each set with at least one similar spectrum in the other set (eq 4). Derive the distance D as the inverse of S minus one, applying the hypothetical half-matching-spectrum convention when S=0 so the metric remains fully symmetric (eq 6). Write S and D to pairwise_distance.csv, or feed the distance matrix into compareMS2's UPGMA tree, spectra2species, or heatmap outputs.

## Related tools

- **compareMS2** (Runs/replicates the eq 1 spectral similarity and eq 4-6 set-level overlap and distance computation on parsed MGF peak lists, optionally emitting a distance matrix and phylogenetic tree) — https://github.com/524D/compareMS2.git

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn
```

## Evaluation signals

- Verify pairwise_distance.csv exists and contains S and D fields with both values normalized to the range 0 to 1
- Symmetry check: D(A,B) must equal D(B,A), and D(A,A)=0 since every dataset is identical to itself
- Datasets with identical spectral content must yield similarity 1 and distance 0
- Datasets with no similar spectra drive similarity toward 0 and distance toward +infinity as set size grows
- Recomputing eq 1 similarities with the same m/z window and similarity threshold reproduces the same similar-spectrum subsets and hence the same S

## Limitations

- Datasets sharing no spectra have no finite distance in the limit; the hypothetical half-matching-spectrum convention when S=0 is required to keep D symmetric and bounded
- The Jaccard-style overlap is very sensitive to the precise definition of when two spectra are considered identical (m/z window and similarity threshold choices)
- Runtime scales poorly: 10 pairwise comparisons took 40 min with default parameters, and 100 datasets require 4950 pairwise comparisons taking several hours

## Evidence

- [readme] compareMS2 calculates the global similarity between tandem mass spectrometry datasets.: "compareMS2 calculates the global similarity between tandem mass spectrometry datasets."
- [readme] defines similarity as a function of shared (similar) spectra and distance as the inverse of this similarity. Data with identical spectral content thus have similarity 1 and distance 0.: [evidence span withheld — anchor status: not_found]
- [readme] The default distance metric D is symmetric, i.e. the distance from dataset A to dataset B is identical to the distance from dataset B to dataset A.: [evidence span withheld — anchor status: external_doc]
- [abstract] The data have been deposited to ProteomeXchange with dataset identifier PXD034932.: "The data have been deposited to ProteomeXchange with dataset identifier PXD034932."
- [results] uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [methods] git clone https://github.com/524D/compareMS2.git: "git clone https://github.com/524D/compareMS2.git"

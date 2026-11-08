---
name: symmetric-distance-metric-implementation
description: Use when quantifying the global similarity/distance between two sets
  of tandem mass spectra (e.g., MGF files from LC-MS/MS runs) with compareMS2 2.0,
  or when a fully symmetric spectral-distance metric is needed for clustering, phylogenetics,
  or QC.
when_to_use_negative:
- The task requires matching spectra against predicted spectra of specific peptides
  (database search or spectral-library search such as Mascot, SpectraST) rather than
  direct comparison of tandem mass spectra datasets
- The input is already a protein/peptide identification table rather than raw tandem
  mass spectral peak lists (MGF)
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: Implements the spectral similarity (eq 1), similar-spectrum subsets (eqs 2-3),
    global similarity S (eq 4) and fully symmetric distance D (eq 6) between LC-MS/MS
    dataset pairs
  repo: https://github.com/524D/compareMS2.git
- name: DISMS2
  role: Alternative R library for direct tandem mass spectral dataset comparison cited
    alongside compareMS2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:54ebbcd2ad8f73be1ff87677fa07df347e58a7db35ecc7ba0db0c548862a12a0
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

# symmetric-distance-metric-implementation

## Summary

Reconstructs compareMS2 2.0's fully symmetric set-distance metric D(A,B) for tandem mass spectrometry datasets, going from pairwise cosine/spectral-angle similarity between MS2 spectra (eq 1) through similar-spectrum subsets (eqs 2-3) to a global similarity S (eq 4) and its inverse distance D (eq 6).

## When to use

Use when quantifying the global similarity/distance between two sets of tandem mass spectra (e.g., MGF files from LC-MS/MS runs) with compareMS2 2.0, or when a fully symmetric spectral-distance metric is needed for clustering, phylogenetics, or QC.

## When NOT to use

- The task requires matching spectra against predicted spectra of specific peptides (database search or spectral-library search such as Mascot, SpectraST) rather than direct comparison of tandem mass spectra datasets
- The input is already a protein/peptide identification table rather than raw tandem mass spectral peak lists (MGF)

## Inputs

- Two or more MGF files of tandem mass spectra (e.g., from ProteomeXchange deposit PXD034932)
- compareMS2 parameters: precursor m/z window and spectral similarity threshold for defining similar spectra

## Outputs

- Global similarity S and symmetric distance D per dataset pair (validated fields of pairwise_distance.csv, both normalized to 0-1)
- Lower-triangular distance matrix feeding UPGMA tree construction in compareMS2

## How to apply

Parse two MGF peak-list files and compute the pairwise cosine/spectral-angle similarity (eq 1) between every spectrum in the first file and every spectrum in the second, restricted within a precursor m/z window and a spectral similarity threshold. Group query and library spectra whose similarity exceeds the threshold into the similar-spectrum subsets of eqs 2-3. Define the global similarity S (eq 4) as the average of the fraction of spectra in each set having at least one similar spectrum in the other set. Compute the global distance D as the inverse of S minus one (eq 6); when no spectra match and S=0, apply the hypothetical half-matching-spectrum convention so the metric remains fully symmetric, i.e., D(A,B)=D(B,A). The metric is designed so identical spectral content gives similarity 1 and distance 0, while the distance from A to B equals the distance from B to A, so D(B,A) need not be recomputed.

## Related tools

- **compareMS2** (Implements the spectral similarity (eq 1), similar-spectrum subsets (eqs 2-3), global similarity S (eq 4) and fully symmetric distance D (eq 6) between LC-MS/MS dataset pairs) — https://github.com/524D/compareMS2.git
- **DISMS2** (Alternative R library for direct tandem mass spectral dataset comparison cited alongside compareMS2)

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn
```

## Evaluation signals

- pairwise_distance.csv exists and contains both S and D fields
- Both S and D values lie within the normalized range 0 to 1
- Symmetry invariant: D(A,B) equals D(B,A) for every dataset pair; D(A,A)=0
- Datasets with identical spectral content yield similarity 1 and distance 0
- Swap convention: applying the hypothetical half-matching-spectrum rule keeps D finite and symmetric when S=0

## Limitations

- For datasets with no similar spectra, similarity tends to 0 (distance tends to +infinity) as the sizes of the sets go to infinity; extremes of none or all spectra being similar are unlikely in real LC-MS/MS data
- Jaccard similarity was found very sensitive to the precise definition of when two spectra are considered identical
- Runtime scales quadratically: 10 pairwise comparisons took 40 min with default parameters, and comparing 100 LC-MS/MS datasets requires 4950 pairwise comparisons (several hours)

## Evidence

- [readme] compareMS2 calculates the global similarity between tandem mass spectrometry datasets.: "compareMS2 calculates the global similarity between tandem mass spectrometry datasets."
- [readme] README on metric symmetry and inverseness: [evidence span withheld — anchor status: external_doc]
- [readme] README on symmetric default metric: [evidence span withheld — anchor status: external_doc]
- [results] Article on spectral similarity for overlap: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [intro] Article intro on fully symmetric metric: "Here, we describe a new and significantly updated version of the original compareMS2 software, with several improvements, including a graphical user interface (GUI)"
- [results] Article results on Jaccard sensitivity: "making the Jaccard similarity very sensitive to the precise definition of when two spectra are considered identical"

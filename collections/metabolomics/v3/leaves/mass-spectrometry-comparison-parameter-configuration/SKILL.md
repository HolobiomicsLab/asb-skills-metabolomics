---
name: mass-spectrometry-comparison-parameter-configuration
description: Use when raw or converted tandem MS/MS datasets (e.g. MGF files such
  as the PXD034932 serum datasets with ~42,629–47,626 spectra each) must be compared
  pairwise and suitable compareMS2 parameters like m/z windows and spectral similarity
  thresholds need to be estimated and set.
when_to_use_negative:
- The goal is peptide/protein identification against a sequence database or spectral
  library — compareMS2 compares sets of tandem mass spectra directly with each other,
  not to predicted spectra of specific peptides
- Datasets are already reduced to a distance/feature table and only tree reconstruction
  or visualization is needed
- Only a single spectrum's quality properties (charge states, noise level) are of
  interest without any cross-dataset comparison — the heatmap self-comparison mode
  is a special case, not the general parameter-configuration workflow
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: compareMS2 2.0
  role: Primary tool whose GUI/CLI is configured with m/z windows, spectral similarity
    thresholds, compare mode, and sample-to-species mapping to compute symmetric distances
    between tandem MS/MS datasets
  repo: https://github.com/524D/compareMS2.git
- name: MEGA11
  role: External reference used to validate that the phylogenetic tree reconstructed
    from the compareMS2 distance matrix is correct
- name: DISMS2
  role: Related direct-comparison library mentioned as alternative prior work, not
    part of this skill's configuration workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:ab35c37c4c79f4eda65c9a0a34281c32302da9c76bf4b2e763398721d846407c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - DISMS2
  - MEGA11
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mass-spectrometry-comparison-parameter-configuration

## Summary

Configure compareMS2 2.0 (m/z windows, spectral similarity thresholds, compare mode, and sample-to-species mapping) before computing distances between sets of tandem mass spectra, so that pairwise LC-MS/MS dataset comparisons are reproducible and the resulting distance matrix is a true metric.

## When to use

Use when raw or converted tandem MS/MS datasets (e.g. MGF files such as the PXD034932 serum datasets with ~42,629–47,626 spectra each) must be compared pairwise and suitable compareMS2 parameters like m/z windows and spectral similarity thresholds need to be estimated and set.

## When NOT to use

- The goal is peptide/protein identification against a sequence database or spectral library — compareMS2 compares sets of tandem mass spectra directly with each other, not to predicted spectra of specific peptides
- Datasets are already reduced to a distance/feature table and only tree reconstruction or visualization is needed
- Only a single spectrum's quality properties (charge states, noise level) are of interest without any cross-dataset comparison — the heatmap self-comparison mode is a special case, not the general parameter-configuration workflow

## Inputs

- Tandem MS/MS dataset files (MGF) from LC-MS/MS runs, e.g. the five PXD034932 serum datasets containing 42,629–47,626 tandem mass spectra each
- Optional TAB-separated sample-to-species mapping file (spectra_to_species.txt) mapping filenames to species names
- compareMS2 parameter settings: m/z windows, spectral similarity thresholds, instrument-specific settings, and output format

## Outputs

- Pairwise distances between all dataset combinations (e.g. 10 unique comparisons for 5 datasets) assembled into a symmetric distance matrix
- A phylogenetic tree (UPGMA) drawn from the lower-triangular distance matrix, a spectra2species bar chart, or a spectrum-inner-product heatmap
- distance_matrix.tsv containing all 10 unique pairwise distances for the five serum datasets

## How to apply

Install the compareMS2 2.0 environment by cloning https://github.com/524D/compareMS2, enabling yarn with 'corepack enable', and running 'yarn' (NodeJS 22.22.3; newer versions are blocked by an Electron Forge bug). Open the GUI ('yarn start' in development mode) and choose a compare mode — phylogenetic tree (all-vs-all), spectra2species (one file vs a set), or heatmap — then set instrument-specific settings and output format on the settings tab. Configure the spectral similarity in eq 1 with its m/z windows and spectral similarity thresholds; the README notes these parameters can be estimated by comparing similarity between tandem mass spectra as a function of precursor m/z difference. If species-level naming or averaging over replicate files is desired, provide a TAB-separated 'spectra_to_species.txt' file mapping filenames to species names. Rely on the default distance metric being fully symmetric: D(A,B) = D(B,A) and D(A,A) = 0, so only n(n-1)/2 comparisons are needed (10 for five datasets, 4950 for 100 datasets).

## Related tools

- **compareMS2 2.0** (Primary tool whose GUI/CLI is configured with m/z windows, spectral similarity thresholds, compare mode, and sample-to-species mapping to compute symmetric distances between tandem MS/MS datasets) — https://github.com/524D/compareMS2.git
- **MEGA11** (External reference used to validate that the phylogenetic tree reconstructed from the compareMS2 distance matrix is correct)
- **DISMS2** (Related direct-comparison library mentioned as alternative prior work, not part of this skill's configuration workflow)

## Examples

```
git clone https://github.com/524D/compareMS2; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- The assembled 5x5 distance matrix is exactly symmetric: D(A,B) = D(B,A) for all pairs, and diagonal entries D(A,A) = 0
- Exactly 10 unique pairwise distances are present for the five serum datasets (n(n-1)/2 comparisons), no duplicates or missing cells
- Datasets with identical spectral content yield similarity 1 and distance 0; distance grows as shared spectra decrease
- The phylogenetic tree reconstructed in compareMS2 matches the one reconstructed in MEGA11 from the same distance matrix
- Runtime is consistent with the reference benchmark: 10 pairwise comparisons with default parameters took about 40 min on an Intel Xeon W-2135 at 3.70 GHz

## Limitations

- The Jaccard similarity is very sensitive to the precise definition of when two spectra are considered identical, so m/z window and threshold settings strongly affect results
- There is no point computing D(B,A) after D(A,B) or D(A,A), since the default metric is fully symmetric and self-distances are zero
- NodeJS version constraints apply: due to an Electron Forge bug, the latest NodeJS version cannot be used (22.22.3 recommended as of June 2026)
- Both the amino acid sequences and the abundance/coverage of proteins affect the distance metric, so distances reflect biology as well as spectral overlap

## Evidence

- [results] used to estimate suitable parameters for compareMS2, e.g., m/z windows and spectral similarity thresholds: "used to estimate suitable parameters for compareMS2, e.g., m/z windows and spectral similarity thresholds"
- [results] uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [methods] git clone https://github.com/524D/compareMS2.git
cd compareMS2
corepack enable
yarn: "git clone https://github.com/524D/compareMS2.git"
- [readme] The default distance metric D is symmetric, i.e. the distance from dataset A to dataset B is identical to the distance from dataset B to dataset A. If the distance D(A, B) has already been calculated, there is no need to calculate D(B, A).: [evidence span withheld — anchor status: external_doc]
- [readme] This file maps file names to species names. If this file is present, the GUI will display the name of the species instead of the name of the sample file.: [evidence span withheld — anchor status: external_doc]
- [results] making the Jaccard similarity very sensitive to the precise definition of when two spectra are considered identical: "making the Jaccard similarity very sensitive to the precise definition of when two spectra are considered identical"

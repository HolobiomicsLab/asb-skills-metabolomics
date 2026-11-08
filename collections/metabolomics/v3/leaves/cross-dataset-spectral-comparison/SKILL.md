---
name: cross-dataset-spectral-comparison
description: Use when you hold MGF peak lists (with precursor m/z) from two LC-MS/MS
  datasets and want to compare spectral similarity versus precursor m/z difference
  pre-database-search, e.g. human vs. chimpanzee or human vs. E. coli serum.
when_to_use_negative:
- You need peptide/protein identifications — compareMS2 compares sets of tandem mass
  spectra to each other rather than to predicted spectra of specific peptides, so
  use a database search engine (Mascot, Comet, Sage) instead
- Your input is already identified spectra or a spectral library — this skill compares
  raw tandem mass spectra directly without sequence data or sequence-assigned spectral
  libraries
- Your goal is peptide-level clustering of spectra within a single dataset — spectral
  clustering tends to produce large globs rather than a distinct cluster per peptide
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: Provides the eq 1 spectral similarity metric and GUI Heatmap mode (inner-product
    distribution of spectra) used to visualize similarity as a function of precursor
    m/z difference; the osf.io/jey28 R scripting examples implement the pairwise comparison
    and binning
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:99a9bf0c1f08b5ac9a838260dcb9173299804334a61ac1de6db43f0b3177fa60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-dataset-spectral-comparison

## Summary

Compares tandem mass spectra directly between two LC-MS/MS datasets by computing spectral similarity (eq 1 cosine-style overlap) as a function of precursor m/z difference, rendering a 2D histogram that reveals charge states, mass accuracy, isotope errors and shared tryptic peptides before and independently of any database search.

## When to use

Use when you hold MGF peak lists (with precursor m/z) from two LC-MS/MS datasets and want to compare spectral similarity versus precursor m/z difference pre-database-search, e.g. human vs. chimpanzee or human vs. E. coli serum.

## When NOT to use

- You need peptide/protein identifications — compareMS2 compares sets of tandem mass spectra to each other rather than to predicted spectra of specific peptides, so use a database search engine (Mascot, Comet, Sage) instead
- Your input is already identified spectra or a spectral library — this skill compares raw tandem mass spectra directly without sequence data or sequence-assigned spectral libraries
- Your goal is peptide-level clustering of spectra within a single dataset — spectral clustering tends to produce large globs rather than a distinct cluster per peptide

## Inputs

- MGF peak list files containing tandem mass spectra with precursor m/z (e.g. serum datasets from ProteomeXchange PXD034932)
- R scripting examples from osf.io/jey28 implementing the eq 1 spectral similarity within a precursor m/z window

## Outputs

- Two-dimensional histogram of precursor m/z difference (x-axis) versus spectral similarity (y-axis), saved as mzdiff_similarity_histogram.png
- Empirically grounded values for compareMS2 m/z window and spectral similarity threshold parameters

## How to apply

Download the MGF peak list files for the two serum datasets from the ProteomeXchange deposit PXD034932 and the R scripting examples from osf.io/jey28. Parse the MGF files using compareMS2's R scripting examples, reading each tandem mass spectrum together with its precursor m/z. Compute the spectral similarity between spectrum pairs across the two datasets within a precursor m/z window, using the cosine-style overlap of eq 1 as implemented in the osf.io/jey28 scripts. Bin the spectrum pairs by precursor m/z difference (x-axis) and spectral similarity (y-axis) and render a 2D histogram, e.g. mzdiff_similarity_histogram.png. This histogram view is the same inner-product distribution the compareMS2 2.0 GUI produces in Heatmap mode, and its pattern (many similar spectra at small m/z differences for closely related species, few for distant species) can be used to estimate suitable compareMS2 parameters such as m/z windows and spectral similarity thresholds.

## Related tools

- **compareMS2** (Provides the eq 1 spectral similarity metric and GUI Heatmap mode (inner-product distribution of spectra) used to visualize similarity as a function of precursor m/z difference; the osf.io/jey28 R scripting examples implement the pairwise comparison and binning) — https://github.com/524D/compareMS2.git

## Evaluation signals

- For human vs. chimpanzee serum (closely related species), the histogram shows high spectral similarity concentrated at small precursor m/z differences; for human vs. E. coli (few shared tryptic peptides), few similar spectra appear
- The histogram reproduces the pattern reported in the paper, verified by expert review against the corresponding paper figure
- Similarity values are computed with the cosine-style overlap of eq 1 within a precursor m/z window, as implemented in the osf.io/jey28 scripts
- The x-axis is annotated with precursor m/z difference and the y-axis with spectral similarity, and the figure is saved as mzdiff_similarity_histogram.png
- The observed m/z differences expose charge states, mass measurement precision and isotope errors, consistent with the compareMS2 heatmap view

## Limitations

- The Jaccard similarity is very sensitive to the precise definition of when two spectra are considered identical
- The distance metric reflects not only amino acid sequences of peptides but also protein abundance/coverage, so differences may stem from expression, not sequence
- Validation requires expert review against the corresponding paper figure; similarity for dissimilar datasets tends to 0 (distance +∞) as set size grows

## Evidence

- [results] we compared the similarity between tandem mass spectra as a function of precursor m/z difference: "we compared the similarity between tandem mass spectra as a function of precursor m/z difference"
- [results] used to estimate suitable parameters for compareMS2, e.g., m/z windows and spectral similarity thresholds: "used to estimate suitable parameters for compareMS2, e.g., m/z windows and spectral similarity thresholds"
- [readme] This gives an overview of many properties of the MS2 spectra, such as charge stated, mass accuracy, and similarity: [evidence span withheld — anchor status: external_doc]
- [readme] See PRIDE Project PXD034932 for additional compareMS2 test data: [evidence span withheld — anchor status: not_found]

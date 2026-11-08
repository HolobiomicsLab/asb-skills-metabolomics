---
name: precursor-mz-difference-histogramming
description: Use when you hold two MGF peak-list files (e.g., serum tandem mass spectra
  from ProteomeXchange PXD034932) and want to visualize spectral similarity as a function
  of precursor m/z difference to inspect charge states, mass accuracy, and isotope
  errors without a database search.
when_to_use_negative:
- You need peptide or protein identifications or protein quantitation — compareMS2
  compares sets of tandem mass spectra to each other rather than to predicted spectra
  of specific peptides
- Your goal is a whole-dataset distance matrix or phylogenetic tree rather than a
  pairwise, spectrum-level similarity distribution (use compareMS2's phylogenetic
  tree or heatmap modes instead)
- The input data are already processed into peak tables or feature tables; this skill
  operates on raw tandem mass spectra peak lists (MGF)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: compareMS2
  role: Provides the spectral similarity metric (eq 1 cosine-style overlap) and the
    workflow context; the osf.io/jey28 R scripting examples compute spectrum-pair
    similarities within precursor m/z windows
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:df95000f7f32045da40a40dc37909b515db9b8f594adebdebe9b1f5ad2dd28d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# precursor-mz-difference-histogramming

## Summary

Generates a 2D histogram of precursor m/z difference (x-axis) versus spectral similarity (y-axis) for tandem mass spectra from two LC-MS/MS datasets, using compareMS2's cosine-style spectral similarity (eq 1). This reveals charge states, mass measurement precision, and isotope errors before and independent of any database search.

## When to use

Use when you hold two MGF peak-list files (e.g., serum tandem mass spectra from ProteomeXchange PXD034932) and want to visualize spectral similarity as a function of precursor m/z difference to inspect charge states, mass accuracy, and isotope errors without a database search.

## When NOT to use

- You need peptide or protein identifications or protein quantitation — compareMS2 compares sets of tandem mass spectra to each other rather than to predicted spectra of specific peptides
- Your goal is a whole-dataset distance matrix or phylogenetic tree rather than a pairwise, spectrum-level similarity distribution (use compareMS2's phylogenetic tree or heatmap modes instead)
- The input data are already processed into peak tables or feature tables; this skill operates on raw tandem mass spectra peak lists (MGF)

## Inputs

- MGF peak list files for two LC-MS/MS datasets (e.g., serum datasets from ProteomeXchange PXD034932), each containing tens of thousands of tandem mass spectra with precursor m/z
- R scripting examples from osf.io/jey28 implementing the spectral similarity computation

## Outputs

- Two-dimensional histogram image (mzdiff_similarity_histogram.png) with precursor m/z difference on the x-axis and spectral similarity on the y-axis
- Distribution of pairwise spectral similarities binned by precursor m/z difference

## How to apply

Download the MGF peak list files for two datasets (e.g., human vs. chimpanzee serum, and human vs. E. coli as a distant-species control) from ProteomeXchange PXD034932 and the R scripting examples from osf.io/jey28. Parse each MGF file, reading every tandem mass spectrum together with its precursor m/z. For spectrum pairs between the two datasets within a precursor m/z window, compute the spectral similarity (the cosine-style overlap of eq 1 used by compareMS2, as implemented in the osf.io/jey28 scripts). Bin the resulting pairs by precursor m/z difference and spectral similarity, render the 2D histogram, and annotate the axes as precursor m/z difference and spectral similarity. Compare the human-vs-chimpanzee and human-vs-E. coli patterns: closely related species share many tryptic peptides and show similarity concentrated at small m/z differences, while distant species show few shared spectra. The histogram can also be used to estimate suitable compareMS2 parameters such as m/z windows and spectral similarity thresholds.

## Related tools

- **compareMS2** (Provides the spectral similarity metric (eq 1 cosine-style overlap) and the workflow context; the osf.io/jey28 R scripting examples compute spectrum-pair similarities within precursor m/z windows) — https://github.com/524D/compareMS2.git

## Evaluation signals

- The 2D histogram reproduces the pattern reported in the paper, where similarity between tandem mass spectra is compared as a function of precursor m/z difference (expert review against the corresponding paper figure)
- Human vs. chimpanzee (closely related species) shows abundant similar spectra, while human vs. E. coli shows few shared spectra due to few shared tryptic peptides
- Axes are correctly annotated as precursor m/z difference (x) and spectral similarity (y), and the output figure is saved as mzdiff_similarity_histogram.png
- Spectral similarities are computed with the compareMS2 eq 1 cosine-style overlap, matching the implementation in the osf.io/jey28 scripts
- Each parsed MGF spectrum is paired with its precursor m/z before similarity computation

## Limitations

- Datasets are large: the five serum datasets each contain between 42,629 and 47,626 tandem mass spectra, so pairwise comparisons can take substantial time (10 pairwise compareMS2 comparisons took 40 min with default parameters)
- Similarity between datasets with no similar spectra tends to 0 as the size of the sets grows, so distant-species histograms may be dominated by near-zero similarity bins
- Not only amino acid sequences of peptides affect the metric, but also the abundance (or coverage) of the proteins, which can confound cross-species similarity patterns

## Evidence

- [results] Comparing similarity as a function of precursor m/z difference: "we compared the similarity between tandem mass spectra as a function of precursor m/z difference"
- [results] Histograms estimate compareMS2 parameters: "used to estimate suitable parameters for compareMS2, e.g., m/z windows and spectral similarity thresholds"
- [results] Similarity metric is eq 1 overlap: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [readme] Heatmap-like overview of MS2 spectrum properties: [evidence span withheld — anchor status: external_doc]
- [results] Scale of serum datasets: "five new serum datasets, each containing between 42,629 and 47,626 tandem mass spectra"

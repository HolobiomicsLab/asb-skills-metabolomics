---
name: spectral-dataset-quality-control
description: Use when you receive MGF files from an LC-MS/MS run (e.g. PXD034932 serum
  data) and need to verify dataset completeness or detect outliers and batch effects
  prior to downstream search.
when_to_use_negative:
- Data are already peptide/protein identification results (e.g. after PeptideProphet
  or Percolator scoring) and you need identifications rather than dataset-level spectral
  similarity
- Only a single dataset is available and you need per-peptide spectral clustering;
  the article notes spectral clustering tends to produce large globs of spectra rather
  than a distinct cluster per peptide
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: Calculates the global similarity/distance between sets of tandem mass spectra
    from MGF files; the 2.0 GUI displays a phylogenetic tree with a quality metric
    used for QC.
  repo: https://github.com/524D/compareMS2.git
- name: MEGA11
  role: Alternative phylogenetic tree reconstruction tool mentioned alongside compareMS2
    for validating tree topology.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:1faad77caa64414e90d309f5516f9506086019bf90b5f91d7d5c22137050f3e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - MEGA11
  - compareMS2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# spectral-dataset-quality-control

## Summary

Quality-control LC-MS/MS serum datasets by tallying tandem mass spectra per file and cross-comparing datasets with compareMS2, so outliers or batch effects are flagged before any database search or protein quantitation.

## When to use

Use when you receive MGF files from an LC-MS/MS run (e.g. PXD034932 serum data) and need to verify dataset completeness or detect outliers and batch effects prior to downstream search.

## When NOT to use

- Data are already peptide/protein identification results (e.g. after PeptideProphet or Percolator scoring) and you need identifications rather than dataset-level spectral similarity
- Only a single dataset is available and you need per-peptide spectral clustering; the article notes spectral clustering tends to produce large globs of spectra rather than a distinct cluster per peptide

## Inputs

- MGF (mascot generic format) files of tandem mass spectra from LC-MS/MS runs (e.g. the five new serum datasets in ProteomeXchange PXD034932)

## Outputs

- Per-dataset MS2 spectrum count table (e.g. ms2_spectrum_counts.csv) with each count expected within 42,629-47,626 for the PXD034932 serum set
- Pairwise distance matrix from compareMS2 together with a UPGMA phylogenetic tree (compareMS2 2.0 GUI) used as a quality metric

## How to apply

First parse each MGF file and count its MS2 records by counting 'BEGIN IONS' entries; for the PXD034932 serum set each of the five datasets should yield 42,629-47,626 tandem mass spectra, so a count outside this interval indicates an incomplete or misparsed file. Then run compareMS2 (CLI or GUI) over the file set to produce pairwise distances; the 2.0 metric is fully symmetric, so if D(A, B) is computed there is no need to recompute D(B, A) or D(A, A), which is always zero. Using the compareMS2 2.0 GUI, display the resulting distance matrix with a phylogenetic tree (or heatmap/spectra2species view) as a quality metric and inspect tree topology and similarity as a function of precursor m/z difference. The rationale, per the article, is that comparing large numbers of datasets prior to database search and protein quantitation detects outliers and possible batch effects, and a run of ~10 datasets only takes about 40 minutes with default parameters.

## Related tools

- **compareMS2** (Calculates the global similarity/distance between sets of tandem mass spectra from MGF files; the 2.0 GUI displays a phylogenetic tree with a quality metric used for QC.) — https://github.com/524D/compareMS2.git
- **MEGA11** (Alternative phylogenetic tree reconstruction tool mentioned alongside compareMS2 for validating tree topology.)

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn
```

## Evaluation signals

- Each of the five PXD034932 serum MGF files yields an MS2 spectrum count inside the reported 42,629-47,626 range
- compareMS2 distance matrix is symmetric: D(A, B) equals D(B, A) and diagonal entries D(A, A) are zero
- UPGMA tree reconstructed in compareMS2 matches the topology obtained independently in MEGA11
- Timing sanity check: 10 pairwise comparisons complete in roughly 40 minutes with default parameters
- MGF parsing must count every 'BEGIN IONS' record; a parse that undercounts indicates an incomplete or misparsed file

## Limitations

- The Jaccard similarity is very sensitive to the precise definition of when two spectra are considered identical
- Spectral clustering of spectra (as in the precursor m/z analysis) tends to produce large globs of spectra rather than a distinct cluster per peptide
- Comparing 100 LC-MS/MS datasets requires 4,950 pairwise comparisons and takes several hours
- The article notes a missing explicit version specification in one place, so confirm you are running compareMS2 2.0

## Evidence

- [results] Each of the five new serum datasets contains between 42,629 and 47,626 tandem mass spectra.: "Using the five new serum datasets, each containing between 42,629 and 47,626 tandem mass spectra"
- [results] The compareMS2 2.0 GUI displays a phylogenetic tree with a quality metric.: "The compareMS2 2.0 GUI (Figure 2) displays a phylogenetic tree with a quality metric"
- [discussion] We make compareMS2 freely available as open source and provide an automatic installer for Microsoft Windows.: "We make compareMS2 freely available as open source"
- [results] The 10 pairwise comparisons in compareMS2 took 40 min with default parameters.: "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [readme] CompareMS also works without a spectra_to_species.txt file; See PRIDE Project PXD034932 for additional compareMS2 test data.: [evidence span withheld — anchor status: not_found]
- [readme] The default distance metric D is symmetric, so if the distance has already been calculated there is no need to recalculate.: [evidence span withheld — anchor status: external_doc]

---
name: ion-adduct-annotation-propagation-statistics
description: Use when a dataset has been processed through the MZmine metaCorrelate
  IIMN workflow and you need to measure how much annotation improves by propagating
  library matches to neighboring ion identity nodes.
when_to_use_negative:
- Input data has not been processed through the MZmine metaCorrelate IIMN workflow
  — without IIN edges there is no neighbor propagation to measure
- The goal is de novo molecular formula or compound-class annotation (e.g., SIRIUS/CANOPUS/Qemistree)
  rather than propagation of existing GNPS spectral library matches across ion identity
  edges
- Only a single raw file is available with no aligned feature list — consensus MS2
  spectra and aligned feature rows are prerequisites for computing propagation gain
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine
  role: Feature detection, alignment, metaCorrelate MS1-based feature grouping, and
    ion identity network construction with 0.001 m/z or 10 ppm tolerance; exports
    the .csv, .mgf, and IIN edge files consumed by this statistics workflow
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Runs the feature-based molecular networking (FBMN) workflow with IIMN enabled,
    producing the ion identity molecular networks whose library-match propagation
    is measured
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: XCMS
  role: Alternative open-source MS processing tool that can generate aligned feature
    lists and IIN-compatible edge files via xcms-gnps-tools utilities for the same
    statistics
  repo: https://github.com/jorainer/xcms-gnps-tools
- name: MS-DIAL
  role: Alternative feature-finding tool interfaced with the IIMN workflow for producing
    the inputs to annotation propagation statistics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:701b3dcabc5727e718871fc916c5333fb1c9e33e6fb9ed92ce253642b731089f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MS-DIAL
  - MZmine
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ion-adduct-annotation-propagation-statistics

## Summary

Quantifies the annotation gain achieved by propagating spectral library matches across Ion Identity Network (IIN) edges in ion identity molecular networking, computing per-dataset MS2 (~6%) and MS1 (~12%) annotation rates plus mean propagation gain (~35%).

## When to use

Use when a dataset has been processed through the MZmine metaCorrelate IIMN workflow and you need to measure how much annotation improves by propagating library matches to neighboring ion identity nodes.

## When NOT to use

- Input data has not been processed through the MZmine metaCorrelate IIMN workflow — without IIN edges there is no neighbor propagation to measure
- The goal is de novo molecular formula or compound-class annotation (e.g., SIRIUS/CANOPUS/Qemistree) rather than propagation of existing GNPS spectral library matches across ion identity edges
- Only a single raw file is available with no aligned feature list — consensus MS2 spectra and aligned feature rows are prerequisites for computing propagation gain

## Inputs

- Aligned feature list with consensus MS2 spectra from 24 public experimental datasets (Supplementary Table 1)
- Feature quantification table (.csv) exported from MZmine
- MS2 spectral summary file (.mgf)
- Supplementary IIN edge files (.csv) for GNPS upload

## Outputs

- annotation_statistics.csv aggregating per-dataset MS2 and MS1 annotation rates and IIN propagation gain
- Per-dataset fractions: MS2 spectral library matches (cosine ≥ 0.9), MS1 ion annotations, and propagation gain percentage
- Headline metrics: mean ~35% annotation gain from IIN propagation, maximum gain of 325% (dataset 4)

## How to apply

Process each dataset in MZmine to create an aligned feature list with consensus MS2 spectra, run metaCorrelate MS1-based feature grouping (RT tolerance 0.1 min, feature shape Pearson correlation ≥ 0.85 with ≥ 5 data points, feature height Pearson correlation ≥ 0.6 with ≥ 3 data points, minimum overlapping intensity percentage 50% in ≥ 2 samples), then build ion identity networks with a maximum m/z tolerance of 0.001 or 10 ppm. Export the feature quantification table (.csv), MS2 spectral summary (.mgf), and supplementary IIN edge files (.csv) and submit to GNPS feature-based molecular networking (FBMN) with IIMN enabled. Compute per-dataset statistics: fraction of features with MS2 spectral library matches (cosine ≥ 0.9, 4–6 shared fragments), fraction with MS1 ion annotations, and the annotation gain from propagating library matches over IIN neighbors. Aggregate results into annotation_statistics.csv; the reference values are ~6% MS2 matching, ~12% MS1 ion annotation, mean propagation gain ~35%, and maximum gain of 325% on dataset 4.

## Related tools

- **MZmine** (Feature detection, alignment, metaCorrelate MS1-based feature grouping, and ion identity network construction with 0.001 m/z or 10 ppm tolerance; exports the .csv, .mgf, and IIN edge files consumed by this statistics workflow) — https://github.com/mzmine/mzmine
- **GNPS** (Runs the feature-based molecular networking (FBMN) workflow with IIMN enabled, producing the ion identity molecular networks whose library-match propagation is measured) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **XCMS** (Alternative open-source MS processing tool that can generate aligned feature lists and IIN-compatible edge files via xcms-gnps-tools utilities for the same statistics) — https://github.com/jorainer/xcms-gnps-tools
- **MS-DIAL** (Alternative feature-finding tool interfaced with the IIMN workflow for producing the inputs to annotation propagation statistics)

## Examples

```
https://ccms-ucsd.github.io/GNPSDocumentation/fbmn-iin/
```

## Evaluation signals

- annotation_statistics.csv reproduces the reported headline values: MS2 library matching ~6%, MS1 ion annotation ~12%, mean propagation gain ~35%, maximum gain 325% on dataset 4
- Per-dataset library match fractions use GNPS cosine similarity ≥ 0.9 with a minimum of 4–6 shared fragment ions
- IIN edges were built with a maximum tolerance of 0.001 m/z or 10 ppm, and feature grouping used RT tolerance 0.1 min with Pearson correlation ≥ 0.85 (shape, ≥ 5 data points) and ≥ 0.6 (height, ≥ 3 data points)
- Exported file set contains the three required components: feature quantification .csv, MS2 spectral summary .mgf, and supplementary IIN edge .csv files
- Gain from propagation is computed as the difference between annotation rate after propagating matches over IIN neighbors and the direct MS2 library-match rate

## Limitations

- Variable feature detection and alignment parameters were applied across the 24 datasets (summarized in Supplementary Table 4), so per-dataset results are not reproducible from the workflow text alone
- Specific MZmine, MS-DIAL, or XCMS versions used to process the datasets are not stated, only references; the special MZmine IIMN build may differ from released versions
- Annotation gains depend on the GNPS-IIMN-PROPOGATED spectral library coverage, which is itself generated by IIMN (minimum cosine 0.9, ≥ 3 signals above 0.1% relative intensity required for new library spectra)
- The IIN edge file format (Supplementary Pairs text file, columns ID1 and ID2) has no formal schema or validator tool mentioned

## Evidence

- [discussion] feature shape Pearson correlation ≥ 0.85 with ≥ 5 data points: "a feature shape Pearson correlation greater equals 0.85 with at least five data points and two data points on each edge"
- [discussion] maximum m/z tolerance 0.001 or 10 ppm for ion identity networking: "a maximum tolerance of 0.001 m/z or 10 ppm"
- [discussion] GNPS library match cosine ≥ 0.9 with 4–6 shared fragments: "contain a match to the GNPS spectral libraries with a minimum cosine similarity of 0.9 and a minimum number of shared fragment ions of 4–6"
- [intro] two layers of connectivity in IIMN output: "The resulting ion identity molecular networks contain two layers of feature (node) connectivity, linking ion identities of the same compound by MS1 characteristics and structurally similar compounds"
- [discussion] MZmine processes data to create aligned feature list: "Process the data to create a feature list (aligned overall samples)"

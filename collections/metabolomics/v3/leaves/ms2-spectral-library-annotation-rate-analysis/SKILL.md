---
name: ms2-spectral-library-annotation-rate-analysis
description: Use when you have FBMN/IIMN outputs (feature table .csv, .mgf MS2 summary,
  IIN edge .csv files) and need to compute annotation rates and the gain from propagating
  spectral library matches over ion identity neighbors.
when_to_use_negative:
- Input is a raw mzML file or unprocessed vendor data — feature detection, alignment,
  and IIN construction must be done first
- The analysis goal is general molecular networking visualization rather than quantifying
  annotation-rate improvement from IIN propagation
- Datasets were not processed with the metaCorrelate IIMN workflow, so there are no
  ion-identity edges to propagate annotations across
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Processes raw data into aligned feature lists, runs metaCorrelate MS1-based
    feature grouping and IIN construction, and exports the .csv/.mgf/edge files for
    annotation statistics
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Runs feature-based molecular networking with IIMN enabled, performs spectral
    library matching, and provides the propagation of library matches over ion-identity
    neighbors
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:8f08e456cc8957385d4b31299f76406bb16bd74e9af92815d67f36d020696ccb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# MS2 spectral library annotation rate analysis

## Summary

Quantifies MS2 spectral library match rates and MS1 ion annotation rates across processed datasets, and measures the annotation gain from propagating library matches to first-neighboring ion identity network (IIN) nodes in IIMN/FBMN results.

## When to use

Use when you have FBMN/IIMN outputs (feature table .csv, .mgf MS2 summary, IIN edge .csv files) and need to compute annotation rates and the gain from propagating spectral library matches over ion identity neighbors.

## When NOT to use

- Input is a raw mzML file or unprocessed vendor data — feature detection, alignment, and IIN construction must be done first
- The analysis goal is general molecular networking visualization rather than quantifying annotation-rate improvement from IIN propagation
- Datasets were not processed with the metaCorrelate IIMN workflow, so there are no ion-identity edges to propagate annotations across

## Inputs

- Feature quantification table (.csv) exported from MZmine aligned feature list
- MS2 spectral summary file (.mgf) with consensus MS2 spectra per feature
- Supplementary IIN edge files (.csv) from MZmine metaCorrelate ion identity networking
- GNPS FBMN job results with IIMN enabled (library matches and ion identity annotations)

## Outputs

- annotation_statistics.csv with per-dataset MS2 library match fraction, MS1 ion annotation fraction, and IIN propagation gain
- Aggregate headline metrics (mean annotation gain, maximum gain per dataset)

## How to apply

Process each dataset through MZmine metaCorrelate IIMN (RT tolerance 0.1 min, feature shape Pearson correlation >= 0.85 with >= 5 data points, feature height Pearson correlation >= 0.6 with >= 3 data points, minimum overlapping intensity percentage 50% in >= 2 samples, IIN m/z tolerance 0.001 or 10 ppm) and submit the exported feature quantification table (.csv), MS2 spectral summary (.mgf), and IIN edge files (.csv) to GNPS FBMN with IIMN enabled. Per dataset, count the fraction of features with MS2 spectral library matches (cosine >= 0.9, 4-6 shared fragments), the fraction with MS1 ion annotations (adducts, isotopes, multimers), and the gain from propagating library matches to first neighboring IIN nodes. Aggregate per-dataset statistics into annotation_statistics.csv; rationale is that ion species of the same compound (adducts, isotopes) are connected by MS1 ion-identity edges, so a single library match can annotate multiple redundant feature nodes. Validate against the paper's headline values: ~6% MS2 library matching, ~12% MS1 ion annotation, mean ~35% propagation gain, and 325% maximum gain (dataset 4).

## Related tools

- **MZmine** (Processes raw data into aligned feature lists, runs metaCorrelate MS1-based feature grouping and IIN construction, and exports the .csv/.mgf/edge files for annotation statistics) — https://github.com/mzmine/mzmine
- **GNPS** (Runs feature-based molecular networking with IIMN enabled, performs spectral library matching, and provides the propagation of library matches over ion-identity neighbors) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Evaluation signals

- Per-dataset MS2 spectral library match fraction (cosine >= 0.9, 4-6 shared fragments) reproduces ~6%
- Per-dataset MS1 ion annotation fraction reproduces ~12%
- Mean IIN propagation gain across the 24 datasets is ~35%
- Maximum propagation gain of 325% occurs on dataset 4
- annotation_statistics.csv covers all 24 public datasets listed in Supplementary Table 1

## Limitations

- Variable feature detection and alignment parameters for the 24 datasets are only in Supplementary Table 4, not fully reproducible from text alone
- Specific MZmine versions used for dataset processing are not stated in the text
- Annotation gain depends on IIN connectivity quality; poor grouping parameters (RT, correlation thresholds) will reduce propagation benefit

## Evidence

- [methods] Annotation gain from IIN propagation: [evidence span withheld — anchor status: not_found]
- [discussion] Library match thresholds: "contain a match to the GNPS spectral libraries with a minimum cosine similarity of 0.9 and a minimum number of shared fragment ions of 4–6"
- [abstract] IIMN purpose: "enhance annotation within molecular networks, and facilitate the expansion of spectral reference libraries."
- [discussion] Export files for FBMN: [evidence span withheld — anchor status: not_found]
- [readme] GNPS workflow availability: [evidence span withheld — anchor status: external_doc]

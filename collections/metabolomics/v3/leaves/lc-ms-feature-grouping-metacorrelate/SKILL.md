---
name: lc-ms-feature-grouping-metacorrelate
description: Use when processing aligned LC-MS/MS feature lists where redundant ion
  species of the same compound must be connected before uploading to GNPS feature-based
  molecular networking with IIMN enabled.
when_to_use_negative:
- Input is a single ungrouped peak table without aligned retention time and m/z values
  across samples — metaCorrelate needs an aligned feature list
- The analysis goal is pure MS2 spectral-similarity networking without ion identity
  propagation (IIMN off) — grouping adds no benefit
- Data are GC-MS EI spectra or MS1-only acquisitions lacking co-eluting MS2 features
  to correlate
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine
  role: Runs metaCorrelate MS1-based feature grouping and ion identity network construction
    on aligned feature lists
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Consumes the exported feature table, .mgf, and IIN edge files to generate
    ion identity molecular networks with both MS1 ion-identity and MS2 spectral-similarity
    layers
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: XCMS
  role: Alternative feature-finding tool; getEdgelist() and getFeatureAnnotations()
    from xcms-gnps-tools extract CAMERA-based adduct edges in IIN format
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:8dcc1b66bbc45daea5bb91d09daa8439725679ffb89fca069c770643f1d101f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  - XCMS
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lc-ms-feature-grouping-metacorrelate

## Summary

Group co-eluting LC-MS features into ion identities (adducts, isotopes, multimers) using MZmine metaCorrelate MS1-based feature grouping, generating the ion identity networks (IIN) that feed Ion Identity Molecular Networking in GNPS.

## When to use

Use when processing aligned LC-MS/MS feature lists where redundant ion species of the same compound must be connected before uploading to GNPS feature-based molecular networking with IIMN enabled.

## When NOT to use

- Input is a single ungrouped peak table without aligned retention time and m/z values across samples — metaCorrelate needs an aligned feature list
- The analysis goal is pure MS2 spectral-similarity networking without ion identity propagation (IIMN off) — grouping adds no benefit
- Data are GC-MS EI spectra or MS1-only acquisitions lacking co-eluting MS2 features to correlate

## Inputs

- Aligned LC-MS feature list with consensus MS2 spectra from MZmine (or XCMS/MS-DIAL)
- Raw MS data converted to open format (e.g., mzML)

## Outputs

- Feature quantification table (.csv)
- MS2 spectral summary file (.mgf)
- Supplementary IIN edge files (.csv) with columns ID1 and ID2
- Ion identity network connecting adducts, isotopes, and multimers

## How to apply

After creating an aligned feature list with consensus MS2 spectra in MZmine, run MZmine metaCorrelate MS1-based feature grouping with a retention time tolerance of 0.1 min, requiring feature shape Pearson correlation ≥ 0.85 with at least five data points (two on each edge), feature height Pearson correlation ≥ 0.6 with at least three data points, and a minimum overlapping intensity percentage of 50% in at least two samples. Then build ion identity networks with a maximum m/z tolerance of 0.001 or 10 ppm to annotate and connect ion species (adducts, isotopes, multimers) of the same compound. The rationale is that chromatographic peak shape correlation links features that co-elute and covary, distinguishing true ion identities of one molecule from unrelated co-eluting compounds. Export the result as a feature quantification table (.csv), an MS2 spectral summary (.mgf), and supplementary IIN edge files (.csv) for GNPS upload, where propagating spectral library matches to first neighboring IIN nodes raised annotation rates by an average of 35%.

## Related tools

- **MZmine** (Runs metaCorrelate MS1-based feature grouping and ion identity network construction on aligned feature lists) — https://github.com/mzmine/mzmine
- **GNPS** (Consumes the exported feature table, .mgf, and IIN edge files to generate ion identity molecular networks with both MS1 ion-identity and MS2 spectral-similarity layers) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **XCMS** (Alternative feature-finding tool; getEdgelist() and getFeatureAnnotations() from xcms-gnps-tools extract CAMERA-based adduct edges in IIN format) — https://github.com/jorainer/xcms-gnps-tools

## Examples

```
Run MZmine metaCorrelate MS1-based feature grouping on the aligned feature list, then export: feature quantification table (.csv), MS2 spectral summary (.mgf), and IIN edges (.csv) for GNPS FBMN with IIMN enabled.
```

## Evaluation signals

- Exported IIN edge file must use the generic comma-separated format with ID1 and ID2 columns
- Per-dataset annotation statistics should show MS2 library matching around 6% and MS1 ion annotation around 12% before propagation
- Propagating library matches over IIN neighbors should increase annotation rates by an average of ~35% (up to 325% on dataset 4)
- Grouped features must satisfy feature shape Pearson correlation ≥ 0.85 (≥5 data points) and feature height Pearson correlation ≥ 0.6 (≥3 data points)
- The .mgf summary must contain consensus MS2 spectra aligned to the feature rows in the .csv quantification table

## Limitations

- The special MZmine IIMN build described may not be part of a released MZmine version; specific MZmine versions used for the 24 datasets are not stated
- Variable feature detection and alignment parameters per dataset (Supplementary Table 4) are not reproducible from the text alone
- The IIN edge file format is a generic comma-separated style with ID1 and ID2 columns; no formal schema or validator tool is provided

## Evidence

- [discussion] Retention time tolerance for grouping: "A pair of features were grouped with a retention time tolerance of 0.1 min"
- [discussion] Feature shape Pearson correlation threshold: "a feature shape Pearson correlation greater equals 0.85 with at least five data points and two data points on each edge"
- [discussion] Feature height Pearson correlation threshold: "a feature height Pearson correlation greater equals 0.6 with at least three data points"
- [discussion] Minimum overlapping intensity percentage: "with a minimum overlapping intensity percentage of 50% in at least two samples in the whole dataset"
- [discussion] Ion identity network m/z tolerance: "a maximum tolerance of 0.001 m/z or 10 ppm"
- [discussion] IIN edge file format: "The text format follows a generic comma-separated style with the columns ID1 and ID2"
- [readme] README describing XCMS-to-IIN helper functions: [evidence span withheld — anchor status: not_found]

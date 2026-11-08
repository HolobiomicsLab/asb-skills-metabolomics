---
name: feature-quantification-table-export
description: Use when an LC-MS dataset has been processed into an aligned feature
  list with MS2 consensus spectra in MZmine (or MS-DIAL/XCMS) and the goal is to run
  FBMN+IIMN on GNPS — the quantification table plus .mgf and IIN edges must be exported
  first.
when_to_use_negative:
- The data have not yet been processed into an aligned feature list — raw spectra
  must first be imported and processed in MZmine/MS-DIAL/XCMS
- The analysis does not need ion identity (MS1) connectivity, e.g. a plain MS2 spectral
  network without adduct/isotope linkage is sufficient
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MZmine
  role: Primary tool that performs feature grouping, ion identity annotation, and
    the final export of the .csv quantification table, .mgf spectral summary, and
    IIN edge files
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Web environment that consumes the exported files to run feature-based molecular
    networking with ion identity molecular networking
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: XCMS
  role: Alternative processing tool whose feature list can be exported to the same
    format set (see jorainer/xcms-gnps-tools for formatting functions)
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:0f4f81e6b417b7d0e1cb6e7d366d4f8925ec8e78a3281cc7add5067d59d76d4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  - XCMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# feature-quantification-table-export

## Summary

Export an aligned, ion-identity-annotated feature list from MZmine as a feature quantification table (.csv) with consensus MS2 spectra in an .mgf file and supplementary IIN edge files (.csv), formatted for direct upload to GNPS feature-based molecular networking (FBMN) with ion identity molecular networking (IIMN) enabled.

## When to use

Use when an LC-MS dataset has been processed into an aligned feature list with MS2 consensus spectra in MZmine (or MS-DIAL/XCMS) and the goal is to run FBMN+IIMN on GNPS — the quantification table plus .mgf and IIN edges must be exported first.

## When NOT to use

- The data have not yet been processed into an aligned feature list — raw spectra must first be imported and processed in MZmine/MS-DIAL/XCMS
- The analysis does not need ion identity (MS1) connectivity, e.g. a plain MS2 spectral network without adduct/isotope linkage is sufficient

## Inputs

- Aligned feature list with consensus MS2 spectra produced in MZmine (or MS-DIAL/XCMS) after MS1-based feature grouping and ion identity annotation
- IIN edge results (ion identity network pairs) from the MZmine IIMN modules

## Outputs

- Feature quantification table (.csv) — rows = features, columns = per-sample intensities
- MS2 spectral summary file (.mgf) with one consensus MS2 spectrum per feature
- Supplementary IIN edge files (.csv) in the generic comma-separated format with ID1 and ID2 columns

## How to apply

After MZmine processing yields an aligned feature list with consensus MS2 spectra, run metaCorrelate MS1-based feature grouping (retention time tolerance 0.1 min, feature shape Pearson correlation ≥ 0.85 with ≥ 5 data points, feature height Pearson correlation ≥ 0.6 with ≥ 3 data points, minimum overlapping intensity percentage 50% in at least two samples) and build ion identity networks with a maximum m/z tolerance of 0.001 or 10 ppm to annotate adducts, isotopes, and multimers. Then export the aligned feature list as three linked artifacts: a feature quantification table (.csv), an MS2 spectral summary file (.mgf), and supplementary IIN edge files (.csv). These three files are the required input set for the GNPS FBMN environment with IIMN enabled; the IIN edges supply the MS1 ion-identity connectivity layer that gets fused with the MS2 spectral-similarity network. Per-dataset annotation statistics (MS2 library matches at cosine ≥ 0.9 with 4–6 shared fragments, MS1 ion annotations, and the ~35% annotation gain from propagating library matches over IIN neighbors) are computed downstream from this export.

## Related tools

- **MZmine** (Primary tool that performs feature grouping, ion identity annotation, and the final export of the .csv quantification table, .mgf spectral summary, and IIN edge files) — https://github.com/mzmine/mzmine
- **GNPS** (Web environment that consumes the exported files to run feature-based molecular networking with ion identity molecular networking) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **XCMS** (Alternative processing tool whose feature list can be exported to the same format set (see jorainer/xcms-gnps-tools for formatting functions)) — https://github.com/jorainer/xcms-gnps-tools

## Examples

```
MZmine GUI: Export → Feature quantification table (.csv) + MS2 spectral summary (.mgf) + IIN edges (.csv), then upload to GNPS FBMN with IIMN enabled
```

## Evaluation signals

- The exported .csv, .mgf, and IIN edge .csv files must all be present and mutually consistent (feature IDs in the table match .mgf titles and IIN ID1/ID2 columns)
- Uploading the three files to GNPS FBMN with IIMN enabled must produce networks containing both the MS1 ion-identity and MS2 spectral-similarity connectivity layers
- Downstream annotation statistics should reproduce the article's headline values: ~6% MS2 library matching, ~12% MS1 ion annotation, ~35% mean gain from IIN propagation, and a maximum gain of 325% on one dataset
- Grouping parameters must match the article thresholds: RT tolerance 0.1 min, shape Pearson ≥ 0.85, height Pearson ≥ 0.6, overlap ≥ 50% in ≥ 2 samples, and IIN m/z tolerance 0.001 or 10 ppm

## Limitations

- The 24 experimental datasets used variable feature detection and alignment parameters summarized in Supplementary Table 4, so a single fixed parameter set does not reproduce all dataset exports
- The IIN edge file follows a generic comma-separated format (columns ID1 and ID2) with no formal schema or validator tool documented
- Specific MZmine/MS-DIAL/XCMS versions used for the 24 datasets are not stated in the article text

## Evidence

- [discussion] Export the feature list as a feature quantification table (.csv), an MS2 spectral summary file (.mgf), and a supplementary edges files (IIN files, .csv): [evidence span withheld — anchor status: not_found]
- [discussion] A pair of features were grouped with a retention time tolerance of 0.1 min: "A pair of features were grouped with a retention time tolerance of 0.1 min"
- [discussion] a feature shape Pearson correlation greater equals 0.85 with at least five data points: "a feature shape Pearson correlation greater equals 0.85 with at least five data points"
- [intro] A table of extracted MS1 features, each with a consensus MS2 spectrum, together with IIN results are then uploaded to GNPS to run the IIMN workflow on the web server: "A table of extracted MS1 features, each with a consensus MS2 spectrum, together with IIN results are then uploaded to GNPS to run the IIMN workflow on the web server"
- [discussion] The text format follows a generic comma-separated style with the columns ID1 and ID2: "The text format follows a generic comma-separated style with the columns ID1 and ID2"

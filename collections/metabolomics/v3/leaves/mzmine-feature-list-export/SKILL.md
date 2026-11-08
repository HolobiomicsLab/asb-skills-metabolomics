---
name: mzmine-feature-list-export
description: Use when MZmine has produced an aligned feature list across samples and
  the next step is uploading to GNPS for FBMN or IIMN, or when supplementary IIN edge
  files are needed to connect adducts/isotopes of the same molecule.
when_to_use_negative:
- When the data have not yet been processed to an aligned feature list — MZmine (or
  MS-DIAL/XCMS) must first complete peak detection, deconvolution and alignment before
  export.
- When the target workflow is not FBMN/IIMN on GNPS — other platforms expect different
  input formats (e.g., XCMS users should instead use jorainer/xcms-gnps-tools formatters).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Performs feature detection, alignment, MS1-based feature grouping/ion-identity
    annotation and provides the export module that generates the .csv feature table,
    .mgf MS2 summary, and IIN supplementary edge files for GNPS upload.
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Consumes the exported feature quantification table, MS2 spectral summary and
    IIN supplementary edges to run the FBMN/IIMN workflow on the web server.
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:677db18934bcd7dd3012220191b8b9c6c6b8a8c1fd41981406bdf12c54e94a38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mzmine-feature-list-export

## Summary

Export an aligned MZmine feature list as the three files required by the GNPS IIMN/FBMN workflow — a feature quantification table (.csv), an MS2 spectral summary (.mgf), and IIN supplementary edge (.csv) files — so MS1-derived ion identities can be fused with MS2 spectral similarity networks.

## When to use

Use when MZmine has produced an aligned feature list across samples and the next step is uploading to GNPS for FBMN or IIMN, or when supplementary IIN edge files are needed to connect adducts/isotopes of the same molecule.

## When NOT to use

- When the data have not yet been processed to an aligned feature list — MZmine (or MS-DIAL/XCMS) must first complete peak detection, deconvolution and alignment before export.
- When the target workflow is not FBMN/IIMN on GNPS — other platforms expect different input formats (e.g., XCMS users should instead use jorainer/xcms-gnps-tools formatters).

## Inputs

- aligned MZmine feature list (intensity table across samples)
- consensus MS2 spectra associated with each feature
- IIN grouping results (feature groups, ion identity annotations)

## Outputs

- feature quantification table (.csv)
- MS2 spectral summary (.mgf)
- supplementary edges files (IIN files, .csv) with ID1/ID2 columns

## How to apply

After MZmine completes feature detection, alignment, MS1-based feature grouping and ion identity annotation (e.g., RT tolerance 0.1 min, m/z tolerance 0.001 m/z or 10 ppm, feature-shape Pearson correlation ≥ 0.85 with ≥ 5 data points), open the Export module of the MZmine IIMN build and export three file types: (1) the feature quantification table (.csv) containing per-feature intensities and annotations, (2) the MS2 spectral summary (.mgf) with one consensus MS2 spectrum per feature, and (3) the supplementary edges files (IIN .csv) encoding ion-identity connections between feature IDs. These three files are then uploaded together to the GNPS web server to run FBMN with or without IIN supplementary edges. The generic comma-separated IIN format uses ID1 and ID2 columns to link feature IDs that represent the same underlying compound. Correct application is judged by verifying that the .mgf contains one spectrum per feature in the .csv, that every ID in the .mgf and .csv matches the feature IDs in the supplementary edge files, and that the edge count reflects the grouping thresholds applied.

## Related tools

- **MZmine** (Performs feature detection, alignment, MS1-based feature grouping/ion-identity annotation and provides the export module that generates the .csv feature table, .mgf MS2 summary, and IIN supplementary edge files for GNPS upload.) — https://github.com/mzmine/mzmine
- **GNPS** (Consumes the exported feature quantification table, MS2 spectral summary and IIN supplementary edges to run the FBMN/IIMN workflow on the web server.) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
mzmine --batch iimn_export_batch.mzbatch
```

## Evaluation signals

- Feature IDs in the .mgf, the .csv quantification table and the IIN supplementary edge files are mutually consistent (every edge ID1/ID2 resolves to a feature in the .csv).
- The .csv contains one row per aligned feature with sample columns matching the imported file list; row count equals the aligned feature-list size.
- The .mgf contains exactly one consensus MS2 spectrum per exported feature; features without MS2 are omitted or flagged per MZmine export settings.
- Edge count in the IIN files scales with applied thresholds — e.g., loosening the feature-shape Pearson correlation below 0.85 or the RT tolerance above 0.1 min increases edge count in a predictable direction.
- Uploading the three files to GNPS FBMN/IIMN runs without parameter-mismatch errors and the resulting network node count matches the .csv row count.

## Limitations

- The MZmine IIMN export is provided by a 'special IIMN build'; whether these modules are part of a standard released MZmine version is unclear, so users may need the development build or a specific version.
- IIN supplementary edges require MS1-based feature grouping and ion identity annotation to have been run inside MZmine — exporting from a feature list without these modules yields empty or absent edge files.
- Variable feature detection/alignment parameters across datasets (documented in Supplementary Table 4) are not pinned in the article text, so exported results may differ if thresholds are changed; the exported edges encode only the grouping decisions made upstream.

## Evidence

- [discussion] Export the feature list as a feature quantification table (.csv), an MS2 spectral summary file (.mgf), and a supplementary edges files (IIN files, .csv): [evidence span withheld — anchor status: not_found]
- [intro] A table of extracted MS1 features, each with a consensus MS2 spectrum, together with IIN results are then uploaded to GNPS to run the IIMN workflow on the web server: "A table of extracted MS1 features, each with a consensus MS2 spectrum, together with IIN results are then uploaded to GNPS to run the IIMN workflow"
- [intro] The IIMN workflow comprises feature grouping, feature shape correlation, and identification of ion species using a variety of feature-finding software tools: "The IIMN workflow comprises feature grouping, feature shape correlation, and identification of ion species using a variety of feature-finding software tools"
- [discussion] The special MZmine IIMN build offers two modules in the Tools tab: "The special MZmine IIMN build offers two modules in the Tools tab"
- [discussion] a feature shape Pearson correlation greater equals 0.85 with at least five data points: "a feature shape Pearson correlation greater equals 0.85 with at least five data points"

---
name: ms1-feature-ion-species-annotation
description: Use when an aligned feature list from LC-MS/MS metabolomics must be grouped
  into ion species (adducts, isotopes, multimers) per molecule, or when relative ion-formation
  percentages per adduct must be computed and compared across datasets.
when_to_use_negative:
- The input is already a feature table with ion-species/adduct annotations, so re-grouping
  would be redundant
- The data are direct-infusion or single-polarity acquisitions without co-eluting
  ion species, so no MS1 ion-identity relationships can be established
- The goal is purely MS2 spectral-similarity molecular networking without collapsing
  ion species redundancy
tools:
- name: MZmine
  role: Feature finding, alignment, and MS1-based ion identity annotation module used
    to generate the feature list and IIN edges
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Web server hosting the FBMN-IIN workflow that fuses MS1 ion identity networks
    with MS2 feature-based molecular networking
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: XCMS
  role: Alternative feature-finding tool; xcms-gnps-tools provides getFeatureAnnotations()
    to extract CAMERA adduct annotations in the IIN format required for FBMN/GNPS
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:70f762a1ed3a75b6984806b7104198a539607aaa3ec494f3237b7d222f07cbdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  - XCMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ms1-feature-ion-species-annotation

## Summary

Annotates aligned LC-MS1 features into ion identities (adducts, isotopes, ion-ligand complexes) using MS1-based ion identity networking (IIN) as implemented in MZmine and the GNPS FBMN-IIMN workflow, enabling per-dataset ion-formation statistics such as the marine vs non-marine [M+NH4]+ contrast.

## When to use

Use when an aligned feature list from LC-MS/MS metabolomics must be grouped into ion species (adducts, isotopes, multimers) per molecule, or when relative ion-formation percentages per adduct must be computed and compared across datasets.

## When NOT to use

- The input is already a feature table with ion-species/adduct annotations, so re-grouping would be redundant
- The data are direct-infusion or single-polarity acquisitions without co-eluting ion species, so no MS1 ion-identity relationships can be established
- The goal is purely MS2 spectral-similarity molecular networking without collapsing ion species redundancy

## Inputs

- Raw LC-MS/MS data converted to open format (mzML)
- Aligned feature list / feature quantification table (.csv) from MZmine, MS-DIAL, or XCMS
- MS2 spectral summary file (.mgf) with consensus spectra per feature
- MS1-based IIN supplementary edge files (.csv, comma-separated with columns ID1 and ID2)
- Sample metadata grouping datasets by sample type (e.g., marine vs non-marine)

## Outputs

- Ion identity molecular network with two layers of node connectivity (MS1 ion-identity links and MS2 spectral-similarity links)
- Per-molecule ion species (adduct) assignments per dataset
- Relative ion-formation percentages per adduct group (e.g., ion_formation_percentages.csv) with group mean ± standard deviation

## How to apply

First process raw spectra (converted to open format such as mzML) in MZmine, MS-DIAL, or XCMS to create an aligned feature list across all samples. Perform MS1-based feature grouping with a retention time tolerance of 0.1 min, requiring a feature shape Pearson correlation ≥ 0.85 with at least five data points (two on each edge), a feature height Pearson correlation ≥ 0.6 with at least three data points, and a minimum overlapping intensity percentage of 50% in at least two samples. Link ion identities with a maximum tolerance of 0.001 m/z or 10 ppm, then export the feature quantification table (.csv), the MS2 spectral summary (.mgf), and the IIN supplementary edge files (.csv) and upload them to GNPS to run the FBMN-IIN workflow on the web server. The rationale is that different ion species of the same compound co-elute and co-vary, so MS1 relationships can collapse redundant features and propagate annotations before MS2 spectral-similarity networking. Finally, aggregate per-molecule ion species assignments per dataset, compute relative ion-formation percentages per adduct, and group datasets (e.g., marine vs non-marine) to compute mean ± standard deviation per adduct group.

## Related tools

- **MZmine** (Feature finding, alignment, and MS1-based ion identity annotation module used to generate the feature list and IIN edges) — https://github.com/mzmine/mzmine
- **GNPS** (Web server hosting the FBMN-IIN workflow that fuses MS1 ion identity networks with MS2 feature-based molecular networking) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **XCMS** (Alternative feature-finding tool; xcms-gnps-tools provides getFeatureAnnotations() to extract CAMERA adduct annotations in the IIN format required for FBMN/GNPS) — https://github.com/jorainer/xcms-gnps-tools

## Examples

```
getFeatureAnnotations(camera_result)  # extract CAMERA adduct annotations in IIN format for FBMN/GNPS
```

## Evaluation signals

- Computed group means of ion-formation percentages fall within reported standard deviations: marine [M+NH4]+ within 24 ± 5%, overall [M+Na]+ within 26 ± 6%, overall [M+H]+ within 23 ± 6%
- IIN edges respect the maximum tolerance of 0.001 m/z or 10 ppm between linked ion identities
- Grouped feature pairs satisfy a feature shape Pearson correlation ≥ 0.85 with at least five data points and two data points on each edge
- Feature grouping uses a retention time tolerance of 0.1 min and a minimum overlapping intensity percentage of 50% in at least two samples
- Exported IIN edge files follow the generic comma-separated format with columns ID1 and ID2 that GNPS FBMN-IIN accepts

## Limitations

- All 24 datasets were processed with the MZmine workflow using variable feature detection and alignment parameters summarized in Supplementary Table 4, which are not reproducible from the article text alone
- The MZmine IIMN modules are described as a 'special IIMN build', and it is unclear whether they are part of a released MZmine version or a custom build
- No container image (Docker/Singularity) or environment file is provided for reproducibility, and specific MZmine/MS-DIAL/XCMS versions are not stated
- Ions with unassigned charge states and isotope peaks were excluded from MS2 acquisition, so such species cannot be validated via MS2

## Evidence

- [discussion] MS1-based feature grouping with ion identity annotation parameters: "Perform MS1-based feature grouping and ion identity annotation"
- [discussion] Ion identity networking m/z tolerance: "a maximum tolerance of 0.001 m/z or 10 ppm"
- [discussion] Feature shape correlation threshold for grouping: "a feature shape Pearson correlation greater equals 0.85 with at least five data points and two data points on each edge"
- [intro] Upload of feature table and IIN results to GNPS: "A table of extracted MS1 features, each with a consensus MS2 spectrum, together with IIN results are then uploaded to GNPS to run the IIMN workflow on the web server"
- [readme] xcms-gnps-tools function to produce IIN-format adduct annotations: [evidence span withheld — anchor status: external_doc]

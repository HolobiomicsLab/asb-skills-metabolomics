---
name: spectral-binning-uniform-bucketing
description: Use when per-sample NMR spectral data (e.g. 1r processed spectra from
  fid/acqu(ac)us acquisitions) must be reduced from full spectral resolution to a
  fixed-width bucketed feature table (exactly 1000 data columns) for downstream analysis
  tools.
when_to_use_negative:
- Input is already a binned/feature table (e.g. already reduced to bucketed columns)
  ready for MetaboAnalyst or Workflows4Metabolomics
- Data are GC–MS or LC–MS raw data rather than NMR-based metabolomics spectra — the
  paper's pipeline and 1r-based binning target NMR acquisitions
- The goal is a data-processing step beyond format standardization/bucketing (e.g.
  advanced spectral deconvolution), which the authors explicitly leave out of CloMet's
  scope
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: Platform whose ImportData module builds the sample-by-spectral-point matrix
    from fid/1r files and applies binning to produce the 1000-column binned data set
  repo: https://github.com/rmallol/clomet
- name: Workflows4Metabolomics
  role: Downstream analysis platform providing NMR_Read, NMR_Preprocessing, and NMR_Bucketing
    tools that consume the binned matrix format
- name: MetaboAnalyst
  role: Downstream analysis platform that accepts CloMet's converted binned output
    data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:b569ac0b6cc0c85ce434edeef4b9602fde079014edd9236ab8a150231665850d
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - MetaboAnalyst
  - Workflows4Metabolomics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spectral-binning-uniform-bucketing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Uniform binning (bucketing) of NMR spectral-point data down to a fixed 1000-column binned matrix, as implemented by CloMet's ImportData module to prepare repository-sourced NMR metabolomics data for analysis platforms.

## When to use

Use when per-sample NMR spectral data (e.g. 1r processed spectra from fid/acqu(ac)us acquisitions) must be reduced from full spectral resolution to a fixed-width bucketed feature table (exactly 1000 data columns) for downstream analysis tools.

## When NOT to use

- Input is already a binned/feature table (e.g. already reduced to bucketed columns) ready for MetaboAnalyst or Workflows4Metabolomics
- Data are GC–MS or LC–MS raw data rather than NMR-based metabolomics spectra — the paper's pipeline and 1r-based binning target NMR acquisitions
- The goal is a data-processing step beyond format standardization/bucketing (e.g. advanced spectral deconvolution), which the authors explicitly leave out of CloMet's scope

## Inputs

- Standardized per-sample NMR acquisition files (acqu/acqus parameter files, fid raw signal files, 1r processed spectra) in CloMet's internal directory-based standard
- Study files retrieved from MetaboLights or Metabolomics Workbench by study id, or user-local data via the LOCAL tab

## Outputs

- binned_data_matrix.csv: sample-by-spectral-variable numeric matrix with exactly 1000 binned data columns
- Format directly usable in MetaboAnalyst or Workflows4Metabolomics (e.g. via NMR_Preprocessing/NMR_Bucketing-compatible input)

## How to apply

After standardizing the study file system (CloMet's internal directory-based standard harmonizing MetaboLights and Metabolomics Workbench layouts), enumerate each sample's acquisition files (acqu/acqus, fid, 1r) and parse acqu/acqus for spectral metadata. Read the raw fid and processed 1r signal files for every sample and assemble a numeric data matrix where each row is a sample and each column is a spectral variable. Apply uniform binning (bucketing) over the spectral-point axis — as done by tools like NMR_Bucketing in Workflows4Metabolomics — to compress each sample's spectrum to exactly 1000 data columns, the resolution the paper specifies. Write the result as a sample-by-1000-column binned data matrix (binned_data_matrix.csv) consumable by MetaboAnalyst or Workflows4Metabolomics.

## Related tools

- **CloMet** (Platform whose ImportData module builds the sample-by-spectral-point matrix from fid/1r files and applies binning to produce the 1000-column binned data set) — https://github.com/rmallol/clomet
- **Workflows4Metabolomics** (Downstream analysis platform providing NMR_Read, NMR_Preprocessing, and NMR_Bucketing tools that consume the binned matrix format)
- **MetaboAnalyst** (Downstream analysis platform that accepts CloMet's converted binned output data)

## Evaluation signals

- The output file binned_data_matrix.csv exists and its spectral dimension equals exactly 1000 binned columns
- Each matrix row corresponds to one sample enumerated from the standardized study directory; no sample is dropped during assembly from fid/1r files
- Column values derive from the samples' 1r processed spectral data (with acqu/acqus metadata used for reconstruction), not from unrelated metadata fields
- The binned matrix loads without schema errors into MetaboAnalyst or Workflows4Metabolomics toolchains (NMR_Read / NMR_Preprocessing / NMR_Bucketing compatibility)

## Limitations

- CloMet itself does not implement data processing algorithms beyond standardization/conversion; the authors state that complementing it with processing algorithms is 'out of our scope'
- Re-analysis oriented to classification problems is infeasible when class metadata is not available alongside the metabolomics data in public repositories
- Maintenance and extension of the underlying software rely on the open-source developer community, and integration with existing analysis platforms is described as a future plan rather than implemented

## Evidence

- [other] a data binning technique is then implemented so the final binned data set contains exactly 1000 data columns, based on the 1r file spectral data.: [evidence span withheld — anchor status: not_found]

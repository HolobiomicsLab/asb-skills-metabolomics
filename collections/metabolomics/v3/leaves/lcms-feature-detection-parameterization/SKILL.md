---
name: lcms-feature-detection-parameterization
description: Use when processing raw LC-MS/MS DDA metabolomics files (positive and/or
  negative ionization) with MZmine in a sample-centric workflow, where parameterization
  and feature filtering must be fixed before RDF/KG integration.
when_to_use_negative:
- The input is already an aligned N x M feature quantification table rather than raw
  per-sample LC-MS2 files — the sample-centric reprocessing step would be redundant.
- Data were not acquired as LC-MS/MS DDA metabolomics (e.g., MS1-only or non-LC platforms)
  — the MZmine DDA deconvolution and MS2-linked filtering do not apply.
- Only downstream annotation is needed and an existing processed ENPKG graph (https://enpkg.commons-lab.org/graphdb/)
  already covers the samples.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Performs mass detection, chromatogram building, deconvolution, isotope grouping
    and feature filtering on each sample's LC-MS2 files
  repo: http://mzmine.github.io/
- name: ENPKG (enpkg_full)
  role: Orchestrates the MZmine feature-detection step via workflow/00_workflow_all.sh
    with parameters fixed in params/user.yml
  repo: https://github.com/enpkg/enpkg_full
- name: MassIVE
  role: Source repository of the raw 1,600-plant-extract LC-MS2 dataset (MSV000087728)
    consumed by the feature-detection step
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:4048a0eedca7460f641a7deb591f474da6d37ec6c9ecd9ad22628a17e03c2cba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ENPKG
  - ENPKG (enpkg_full)
  - MZmine
  - MassIVE
  - RDF
  - matchms
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lcms-feature-detection-parameterization

## Summary

Configures the MZmine feature-detection stage of the ENPKG workflow so that per-sample LC-HRMS2 data (e.g., the 1,600-plant-extract MSV000087728 dataset) are converted into quantified features with linked MS2 spectra, ready for molecular networking and knowledge-graph integration.

## When to use

Use when processing raw LC-MS/MS DDA metabolomics files (positive and/or negative ionization) with MZmine in a sample-centric workflow, where parameterization and feature filtering must be fixed before RDF/KG integration.

## When NOT to use

- The input is already an aligned N x M feature quantification table rather than raw per-sample LC-MS2 files — the sample-centric reprocessing step would be redundant.
- Data were not acquired as LC-MS/MS DDA metabolomics (e.g., MS1-only or non-LC platforms) — the MZmine DDA deconvolution and MS2-linked filtering do not apply.
- Only downstream annotation is needed and an existing processed ENPKG graph (https://enpkg.commons-lab.org/graphdb/) already covers the samples.

## Inputs

- Raw LC-HRMS2 DDA mzML/mzXML files per sample (PI and/or NI modes), e.g., from MassIVE record MSV000087728
- params/user.yml configuration file specifying record_id/record_name and dataset to process
- Minimal per-sample metadata file containing the sample's originating taxon

## Outputs

- Per-sample MZmine feature lists with m/z, RT, area/intensity and associated MS2 spectra
- Filtered per-sample feature exports (only features with an MS2 scan and RT 0.5–8.0 min) feeding molecular networking, ISDB-LOTUS matching and graph building
- Aggregate dataset-scale counts (e.g., 788,623 features in PI and 364,967 in NI modes for the 1,600-extract dataset)

## How to apply

Clone enpkg/enpkg_full and edit params/user.yml to point at the target dataset (e.g., MassIVE record MSV000087728 for 1,600 plant extracts), then launch 'sh workflow/00_workflow_all.sh', which runs the MZmine step with fixed parameters: centroid mass detector with noise level 1.0E4 (MS level 1), ADAP chromatogram builder with min group size of 5 scans, min group intensity threshold 1.0E4 and min highest intensity 5.0E5, wavelets (ADAP) deconvolution with intensity-window S/N estimator at S/N 10 and minimum feature height 5.0E5, and isotope peaks grouper with 12 ppm m/z tolerance. Finally apply the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min. This per-sample, parameter-fixed strategy avoids the retention-time-based feature alignment that hinders cross-batch comparison, so each sample's feature list is generated independently and downstream steps (MN, ISDB-LOTUS matching, SIRIUS) consume the filtered per-sample exports.

## Related tools

- **MZmine** (Performs mass detection, chromatogram building, deconvolution, isotope grouping and feature filtering on each sample's LC-MS2 files) — http://mzmine.github.io/
- **ENPKG (enpkg_full)** (Orchestrates the MZmine feature-detection step via workflow/00_workflow_all.sh with parameters fixed in params/user.yml) — https://github.com/enpkg/enpkg_full
- **MassIVE** (Source repository of the raw 1,600-plant-extract LC-MS2 dataset (MSV000087728) consumed by the feature-detection step)

## Examples

```
git clone https://github.com/enpkg/enpkg_full.git && cd enpkg_full && sh workflow/00_workflow_all.sh
```

## Evaluation signals

- Feature counts reproduce the reported scale: 788,623 features in positive ionization and 364,967 in negative ionization mode over the 1,600-extract dataset.
- Every retained feature has an associated MS2 scan (the feature filtering module criterion).
- All retained features fall within RT 0.5–8.0 min and m/z tolerance groups at 12 ppm for isotopes.
- On the bundled small test dataset (Zenodo 10018590), the full workflow including MZmine step completes in about 10 minutes.
- Generated .ttl files load into GraphDB and yield a graph consistent with the reference at https://enpkg.commons-lab.org/graphdb/.

## Limitations

- Retention-time-based feature alignment in classical workflows hinders comparison of samples from different batches or instrumental setups — the rationale for this per-sample approach, but it means no global RT alignment table is produced.
- Feature detection parameters (noise 1.0E4, min highest intensity 5.0E5, S/N 10, RT window 0.5–8.0 min) were tuned for this dataset's instrumental setup; PI-mode LC-MS2 acquisition parameters are deferred to ref (14) and may need re-tuning elsewhere.
- The workflow applies MZmine with this fixed parameter set only; no benchmark against alternative parameterizations is reported.

## Evidence

- [methods] the centroid mass detector was used for mass detection with the noise level set to 1.0E4 for MS level set to 1: "the centroid mass detector was used for mass detection with the noise level set to 1.0E4 for MS level set to 1"
- [methods] The ADAP chromatogram builder was used and set to a minimum group size of scans of 5: "The ADAP chromatogram builder was used and set to a minimum group size of scans of 5, minimum group intensity threshold of 1.0E4, minimum highest intensity of 5.0E5"
- [methods] The intensity window S/N was used as an S/N estimator with a signal-to-noise ratio set at 10, a minimum feature height at 5.0E5: "The intensity window S/N was used as an S/N estimator with a signal-to-noise ratio set at 10, a minimum feature height at 5.0E5"
- [methods] Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min.: "Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min."
- [results] 788,623 and 364,967 features in positive ionization (PI) and negative ionization (NI) modes, respectively: "788,623 and 364,967 features in positive ionization (PI) and negative
ionization (NI) modes, respectively"
- [results] feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine: "the data for each sample are processed to perform feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine."
- [readme] From the root of the repository, run: sh workflow/00_workflow_all.sh: [evidence span withheld — anchor status: not_found]

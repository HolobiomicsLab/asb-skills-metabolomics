---
name: isotope-peak-grouping
description: Use when processing raw LC-HRMS2 chromatograms in MZmine (mzXML/mzML
  input) and adduct/isotope peaks must be collapsed into single features prior to
  filtering and .mgf export.
when_to_use_negative:
- Input is already a filtered, deconvoluted feature quantification table rather than
  raw MZmine feature lists
- Samples follow the ENPKG sample-centric design and were already isotope-grouped
  in a prior MZmine batch
- Data are not centroided LC-HRMS2 acquisitions processed with MZmine (e.g. proteomics
  or DIA workflows)
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Runs the isotope peaks grouper module that collapses isotopic peaks into single
    features with a representative isotope
  repo: https://github.com/enpkg/enpkg_workflow
- name: ProteoWizard
  role: Converts Thermo RAW LC-MS files to mzXML format, the entry point for MZmine
    processing prior to isotope grouping
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:c7c3a3fe14678b22e03dadf5c1b50386a3cc0b075d2a06d90d4e976204c6c666
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - MZmine
  - ProteoWizard
  - matchms
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# isotope-peak-grouping

## Summary

Group isotopic peaks of the same molecule into single features during MZmine-based feature detection in the ENPKG LC-HRMS2 workflow, so that each deconvoluted feature is represented by its most intense isotope before MS2 export.

## When to use

Use when processing raw LC-HRMS2 chromatograms in MZmine (mzXML/mzML input) and adduct/isotope peaks must be collapsed into single features prior to filtering and .mgf export.

## When NOT to use

- Input is already a filtered, deconvoluted feature quantification table rather than raw MZmine feature lists
- Samples follow the ENPKG sample-centric design and were already isotope-grouped in a prior MZmine batch
- Data are not centroided LC-HRMS2 acquisitions processed with MZmine (e.g. proteomics or DIA workflows)

## Inputs

- mzXML files converted from Thermo RAW via MSConvert (ProteoWizard)
- MZmine feature lists after ADAP chromatogram building and wavelets (ADAP) peak deconvolution

## Outputs

- Isotope-grouped feature lists with one representative (most intense) isotope per feature
- Deconvoluted feature quantification table (m/z, RT, area/intensity) and per-feature MS2 .mgf files exported via 'Export to GNPS'

## How to apply

After chromatogram building (ADAP, minimum group size 5 scans, m/z tolerance 12 ppm) and wavelets (ADAP) deconvolution (S/N 10, minimum feature height 5.0E5, peak duration 0.02-0.5 min), apply MZmine's isotope peaks grouper to each feature list. Use an m/z tolerance of 12 ppm and an absolute RT tolerance of 0.01 min, allow a maximum charge of 2, and set the most intense isotope as the representative peak. This keeps downstream feature filtering (features with MS2 scans, RT 0.5-8.0 min) and 'Export to GNPS' outputs consistent with the reported scale of ~788,623 features in PI and ~364,967 in NI.

## Related tools

- **MZmine** (Runs the isotope peaks grouper module that collapses isotopic peaks into single features with a representative isotope) — https://github.com/enpkg/enpkg_workflow
- **ProteoWizard** (Converts Thermo RAW LC-MS files to mzXML format, the entry point for MZmine processing prior to isotope grouping)

## Evaluation signals

- Isotope grouping uses m/z tolerance of 12 ppm, RT tolerance of 0.01 min (absolute), and maximum charge of 2, matching the ENPKG parameters
- Each resulting feature is represented by its most intense isotope peak
- Downstream feature quantification table exists with expected columns (m/z, RT, area/intensity) and exported .mgf files contain MS2 spectra per retained feature
- Feature counts after filtering are consistent with reported scale (~788,623 features in PI, ~364,967 in NI before per-sample filtering)

## Limitations

- Parameters (12 ppm m/z tolerance, 0.01 min RT tolerance, max charge 2) were tuned for the paper's UHPLC-HRMS2 setup; other instruments or chromatographic gradients may require re-optimization
- Only charge states up to 2 are grouped; higher-charge isotope clusters will not be collapsed
- PI-mode LC-MS2 acquisition parameters are deferred to ref (14), limiting transferability of the exact grouping context

## Evidence

- [methods] Isotopes were detected using the isotope peaks grouper with an m/z tolerance of 12 ppm: "Isotopes were detected using the isotope peaks grouper with an m/z tolerance of 12 ppm"
- [results] MZmine feature detection step: "feature detection and export the resulting files (Figure 2B, step 1), here, using MZmine"
- [intro] Sample-centric processing rationale: "Each sample is considered individually for taxonomic metadata standardization, feature detection, structural annotation, and FBMN."
- [readme] Workflow README: individual sample processing: [evidence span withheld — anchor status: external_doc]

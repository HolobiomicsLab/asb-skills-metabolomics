---
name: sirius-in-silico-compound-class-prediction
description: Use when unannotated MS/MS spectra from an untargeted metabolomics dataset
  sit inside a biologically coherent molecular-network subnetwork (e.g., a carnitine
  subnetwork) and need putative compound-class or structural assignment without a
  GNPS library match.
when_to_use_negative:
- The MS/MS spectrum already has a confident annotation via spectral matching against
  the public GNPS reference libraries (a Level 2 or 3 MSI annotation), which is preferable
  to an in-silico class prediction
- The feature does not belong to any biologically coherent molecular-network subnetwork,
  so there is no supporting context to interpret the in-silico class prediction
- The task is to quantify differentially abundant features rather than to putatively
  classify unannotated spectra
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS
  role: Predicts molecular formulae and putative compound classes (with the CSI:FingerID/compound-class
    module) for unannotated MS/MS spectra exported from the molecular network
- name: GNPS
  role: Provides the untargeted metabolomics MS/MS data (MassIVE repository) and the
    cross-tissue molecular network whose subnetworks localize the unannotated spectra
    before SIRIUS classification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:64c78f51ef1cf75a543f8fd0bb72917c62f5f49056580918a27f3d29c1891084
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sirius-in-silico-compound-class-prediction

## Summary

Uses SIRIUS with its CSI:FingerID/compound-class module to putatively assign compound classes (e.g., fatty-acyl carnitines) to unannotated MS/MS spectra from a molecular network, complementing GNPS spectral-library matching when no Level 2/3 library hit exists.

## When to use

Use when unannotated MS/MS spectra from an untargeted metabolomics dataset sit inside a biologically coherent molecular-network subnetwork (e.g., a carnitine subnetwork) and need putative compound-class or structural assignment without a GNPS library match.

## When NOT to use

- The MS/MS spectrum already has a confident annotation via spectral matching against the public GNPS reference libraries (a Level 2 or 3 MSI annotation), which is preferable to an in-silico class prediction
- The feature does not belong to any biologically coherent molecular-network subnetwork, so there is no supporting context to interpret the in-silico class prediction
- The task is to quantify differentially abundant features rather than to putatively classify unannotated spectra

## Inputs

- Consensus MS/MS spectra (precursor mass + peak list) for unannotated features from a cross-tissue molecular network, e.g. features at m/z 266.138, 280.153, and 294.169 in the carnitine subnetwork
- Untargeted metabolomics MS/MS data downloaded from the public GNPS/MassIVE repository
- Subnetwork membership assignments that place the unannotated features in a biologically relevant subnetwork (e.g., carnitine subnetwork)

## Outputs

- SIRIUS candidate lists with putative molecular formulae and structural candidates per precursor
- Compound-class assignments (e.g., fatty-acyl carnitine) from the SIRIUS compound-class module
- Summary table of precursor m/z, top formula, top compound class, and whether the class matches the expected class

## How to apply

First, retrieve the untargeted metabolomics MS/MS data from the public GNPS/MassIVE repository and locate the consensus MS/MS spectra for the target features inside the relevant subnetwork of the cross-tissue molecular network — for example, the three unannotated spectra at precursor masses m/z 266.138, 280.153, and 294.169 in the carnitine subnetwork of the 3xTg cross-tissue network. Export each consensus spectrum to a SIRIUS-compatible format (.mgf or individual .ms) carrying the precursor mass and MS/MS peak list. Run SIRIUS with the CSI:FingerID/compound-class module enabled to generate putative molecular formulae and structural candidates for each precursor. Inspect the candidate lists and record whether the top-ranked predictions fall within the expected compound class (fatty-acyl carnitine), then aggregate precursor m/z, top formula, top compound class, and class match into a summary table. Because these classifications are putative, treat them alongside the network membership evidence rather than as validated Level 1 annotations.

## Related tools

- **SIRIUS** (Predicts molecular formulae and putative compound classes (with the CSI:FingerID/compound-class module) for unannotated MS/MS spectra exported from the molecular network)
- **GNPS** (Provides the untargeted metabolomics MS/MS data (MassIVE repository) and the cross-tissue molecular network whose subnetworks localize the unannotated spectra before SIRIUS classification)

## Examples

```
sirius console --input spectra.mgf --project sirius_run --formula --fingerid --compound-class
```

## Evaluation signals

- SIRIUS returns a compound-class assignment consistent with fatty-acyl carnitine for the spectra at m/z 266.138, 280.153, and 294.169, matching the paper's statement that they were 'putatively classified as fatty-acyl carnitines via SIRIUS'
- Each exported spectrum carries the correct precursor mass and a non-empty MS/MS peak list before SIRIUS is run
- Top-ranked SIRIUS candidates for all three precursors fall in the same compound class, consistent with subnetwork co-membership
- The summary table contains one row per precursor with precursor m/z, top formula, top compound class, and a boolean class match, with no missing values
- The predicted classes are cross-checked against subnetwork membership (all three spectra fall within the carnitine subnetwork) rather than accepted in isolation

## Limitations

- In-silico compound-class predictions are putative; in the article, MS/MS spectra annotated via spectral matching against GNPS reference libraries only reach Level 2 or 3 annotation under the Metabolomics Standards Initiative, and SIRIUS-based classification is not a validated Level 1 structure
- Some spectral pairs may be missing from the cross-tissue molecular network due to poor fragmentation or missing fragments during MS/MS acquisition, which can exclude relevant spectra from SIRIUS analysis
- Feature detection in public reference datasets can be influenced by study-specific extraction methods, workflows, and instrumentation, so lack of detection does not necessarily mean absence in the tissue
- Hundreds of differentially abundant molecules remain non-annotated even after this workflow, so SIRIUS classification resolves only a subset of unknown features

## Evidence

- [results] The three unannotated spectra were identified as possible carnitines because they fell within the carnitine subnetwork and were putatively classified as fatty-acyl carnitines via SIRIUS.: [evidence span withheld — anchor status: not_found]
- [discussion] MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation according to the Metabolomics Standards Initiative"
- [discussion] it is possible that some spectral pairs are missing due to the poor fragmentation or missing fragments during the acquisition of the MS/MS spectra: "it is possible that some spectral pairs are missing due to the poor fragmentation or missing fragments during the acquisition of the MS/MS spectra"

---
name: mzmine3-workflow-configuration
description: Use when raw LC–MS/MS mzML files need feature extraction and per-feature
  representative MS2 spectra before formula/adduct assignment and molecular candidate
  ranking.
when_to_use_negative:
- Input is already a processed feature table with per-feature m/z and MS2 spectra
  — rerunning MZmine3 extraction is unnecessary.
- The goal is to skip feature extraction entirely and only rank molecular candidates
  from preprocessed spectra; the MVP pretrained model (test.py) consumes already-prepped
  spectra files, not raw mzML.
- Data are negative ion mode spectra when the analysis scope targets only positive
  ion mode data.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine3
  role: Primary tool for mass detection, chromatogram building, feature detection/deconvolution,
    and representative spectra selection on mzML inputs
- name: SIRIUS
  role: 'Downstream step: formula and adduct assignment on the MZmine3 feature list
    (v6.1.0)'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:e3cde63e947ebdf2913c359a942c06d2bb4e50ab54934ef63731c25a5ff3c643
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MZmine3
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mzmine3-workflow-configuration

## Summary

Configures MZmine3 to perform feature extraction (mass detection, chromatogram building, feature deconvolution) and representative spectra selection on LC–MS/MS mzML files, producing a feature table of m/z, retention time, and representative MS2 spectra for downstream annotation.

## When to use

Use when raw LC–MS/MS mzML files need feature extraction and per-feature representative MS2 spectra before formula/adduct assignment and molecular candidate ranking.

## When NOT to use

- Input is already a processed feature table with per-feature m/z and MS2 spectra — rerunning MZmine3 extraction is unnecessary.
- The goal is to skip feature extraction entirely and only rank molecular candidates from preprocessed spectra; the MVP pretrained model (test.py) consumes already-prepped spectra files, not raw mzML.
- Data are negative ion mode spectra when the analysis scope targets only positive ion mode data.

## Inputs

- LC–MS/MS raw files in mzML format (e.g. downloaded from the Metabolomics Workbench public deposit)
- Positive ion mode acquisition files

## Outputs

- feature_table.csv containing per-feature m/z, retention time, and linked representative MS2 spectra
- Chromatographic feature list with formula/adduct assignments ready for SIRIUS v6.1.0 (per article workflow)

## How to apply

Import positive ion mode mzML files into MZmine3 and run mass detection followed by chromatogram building to construct extracted ion chromatograms. Execute MZmine3 feature detection/deconvolution to obtain chromatographic features characterized by m/z and retention time, then apply MZmine3's representative spectra selection module to assign a single representative MS2 spectrum to each feature. Export the resulting feature list (m/z, retention time, and linked representative MS2 spectra per feature) to feature_table.csv. In the MVP paper this MZmine3 pipeline was followed by formula and adduct assignment with SIRIUS v6.1.0, so the exported table should preserve enough per-feature information to support that downstream step.

## Related tools

- **MZmine3** (Primary tool for mass detection, chromatogram building, feature detection/deconvolution, and representative spectra selection on mzML inputs)
- **SIRIUS** (Downstream step: formula and adduct assignment on the MZmine3 feature list (v6.1.0))

## Examples

```
Launch MZmine3 GUI: import mzML files → Mass detection → Chromatogram building → Feature detection → Representative spectra selection → Export feature_table.csv (m/z, RT, MS2 per feature)
```

## Evaluation signals

- feature_table.csv exists and parses as CSV
- Detected feature count is on the order of ~900 features (the meat study dataset detected 900 features, 866 of which received formula and adduct assignments from SIRIUS)
- Every row of feature_table.csv carries a single representative MS2 spectrum after representative spectra selection
- Each feature retains both m/z and retention-time values from chromatogram building/deconvolution
- The exported table retains sufficient per-feature detail for the subsequent SIRIUS v6.1.0 formula and adduct assignment step

## Limitations

- Neither the processed spectra nor the mapping to the identified metabolites was provided by the original meat study; the MZmine3 workflow was a replication, so reference outputs may be unavailable.
- Annotations resulting from the downstream pipeline are putative and require experimental validation.
- Peak labeling accuracy affects representation quality — more accurate peak labeling could improve the feature representations.

## Evidence

- [results] Feature extraction and representative spectra selection using MZmine3: "Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0"
- [results] Positive ion mode focus: "For this evaluation, we focus on the positive ion mode data"
- [results] Metabolomics Workbench data source: "LC–MS/MS data in mzML format are publicly available on the Metabolomics Workbench"
- [results] Candidate retrieval tolerance after feature extraction: "Candidate structures are retrieved within a 0.01 Da tolerance from FoodDB, 39 HMDB,40 MassSpecGym,32 NIST23,1 and the GNPS Multiplex library"
- [results] MZmine3 role in the workflow: "Feature extraction and representative spectra selection are performed using MZmine3"
- [results] Meat study replication context: "Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study"

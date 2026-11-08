---
name: collision-energy-optimization-configuration
description: Use when building a Skyline small-molecule transition list (MRM/SRM targets
  with heavy-labeled internal standards) whose exported instrument method needs per-transition
  predicted collision energies rather than fixed defaults.
when_to_use_negative:
- Input is HRMS or DIA data rather than scheduled MRM/SRM transitions, where per-transition
  CE prediction does not apply
- The goal is untargeted feature finding/alignment (XCMS, MZmine, MS-DIAL style workflows)
  rather than targeted method refinement
- A vendor-specific method already contains empirically tuned, validated collision
  energies and only needs data import/processing
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Skyline
  role: 'Targeted method development: hosts the transition list, enables collision
    energy optimization via Transition Settings -> Prediction, and exports the RT-scheduled
    MRM method'
  repo: https://github.com/ProteoWizard/pwiz-cpp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:dd157978e5a357e610e778339cfcd7ff4200a4baf6e5e67d929974d30a95a477
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Collision-energy-optimization-configuration

## Summary

Configures Skyline to predict/optimize collision energies for small-molecule SRM transitions before exporting a targeted MRM method, as part of the metabolomics targeted-quantification workflow.

## When to use

Use when building a Skyline small-molecule transition list (MRM/SRM targets with heavy-labeled internal standards) whose exported instrument method needs per-transition predicted collision energies rather than fixed defaults.

## When NOT to use

- Input is HRMS or DIA data rather than scheduled MRM/SRM transitions, where per-transition CE prediction does not apply
- The goal is untargeted feature finding/alignment (XCMS, MZmine, MS-DIAL style workflows) rather than targeted method refinement
- A vendor-specific method already contains empirically tuned, validated collision energies and only needs data import/processing

## Inputs

- Skyline small-molecule document (.sky) with imported transition list including compounds and heavy labeled internal standards
- Transition list of small-molecule targets with m/z, formula, and adduct information
- Prior-run measured retention times for RT scheduling (optional, for the subsequent method export)

## Outputs

- Transition settings with collision energy optimization enabled per transition
- Retention-time-scheduled instrument method export with optimized CE values (1-minute centered windows)
- Assembled Skyline document (e.g., small_molecule_workflow_template.sky)

## How to apply

In a small-molecule Skyline document that already has the imported transition list (e.g., arginine/polyamine pathway targets plus heavy labeled internal standards), open Settings -> Transition Settings -> Prediction and enable collision energy optimization so CE values are computed for each transition. Rationale: Skyline's targeted paradigm extracts known targets then detects and quantifies them, and CE optimization ensures each transition is acquired at near-optimal fragmentation energy across instruments. After data acquisition and peak review, proceed to export a retention-time-scheduled method using a 1-minute window centered on the measured retention times from the prior analysis, so the optimized CEs are delivered only in the scheduled windows. Verify by re-importing the vendor raw files (e.g., ADNI p180 serum amino acid files) and confirming each target's extracted ion chromatogram shows reasonable peak integration without manual boundary adjustment.

## Related tools

- **Skyline** (Targeted method development: hosts the transition list, enables collision energy optimization via Transition Settings -> Prediction, and exports the RT-scheduled MRM method) — https://github.com/ProteoWizard/pwiz-cpp

## Examples

```
Skyline GUI: Settings -> Transition Settings -> Prediction -> enable collision energy optimization (then File -> Export -> Method with 1-min RT window)
```

## Evaluation signals

- Transition Settings -> Prediction tab shows collision energy optimization enabled for all imported transitions
- Exported method contains per-transition CE values rather than a single instrument default
- Exported method uses a 1-minute RT scheduling window centered on measured retention times
- Re-imported raw files (e.g., ADNI p180) show satisfactory automatic peak integration with minimal manual boundary adjustment
- Skyline document round-trips with heavy-labeled internal standards paired to their endogenous analytes for calibration

## Limitations

- Fragmentation library workflows for Metlin, MassBank, HMDB, and NIST were not yet implemented in Skyline at publication time, limiting library-guided CE/transition refinement
- Skyline may lag behind commercial vendor software for latest-generation or vendor-specific workflows until support is implemented
- No single tool is best for all workflows; GC-MS and ion mobility small-molecule support was still ongoing

## Evidence

- [supplementary] Collision energy optimization enabled via Settings->Transition Settings->Prediction tab.: "Collision energy optimization enabled via Settings->Transition Settings->Prediction tab."
- [results] a method was exported with retention time scheduling based on the measured retention times from the prior analysis and a time window setting of 1-minute centered on these times: "a method was exported with retention time scheduling based on the measured retention times from the prior analysis and a time window setting of 1-minute centered on these times"
- [abstract] we describe the expansion of Skyline to data for small molecule analysis, including selected reaction monitoring, high-resolution mass spectrometry, and calibrated quantification: "we describe the expansion of Skyline to data for small molecule analysis, including selected reaction monitoring, high-resolution mass spectrometry, and calibrated quantification"
- [readme] Skyline lives under `pwiz_tools/Skyline` and depends on `pwiz_tools/Shared` and the pwiz library projects under `pwiz/src`.: "Skyline lives under `pwiz_tools/Skyline` and depends on `pwiz_tools/Shared` and the pwiz library projects under `pwiz/src`"

---
name: srm-targeted-quantification
description: Use when raw SRM/MRM acquisition files (e.g. Waters Xevo TQ-S) must be
  independently quantified against a vendor result such as TargetLynx, or when a targeted
  amino-acid/metabolite panel (e.g. the 19 amino acids from a Biocrates p180 kit)
  needs calibrated concentrations outside vendor software.
when_to_use_negative:
- The input is already a processed feature/peak table from untargeted tools such as
  XCMS, MZmine, or MS-DIAL — Skyline's SRM skill extracts known targets from raw data,
  not reprocessing aligned feature lists.
- The acquisition is untargeted or data-independent MS/MS requiring deconvolution
  and feature finding — the targeted approach needs a predefined transition list.
- GC-MS or full ion-mobility small-molecule workflows — the paper notes support for
  these data types is still ongoing.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Skyline
  role: Imports vendor SRM raw files, integrates scheduled transitions, applies the
    external calibration fit (linear with 1/x weighting), and exports calibrated concentrations
    for the 19 amino acids.
  repo: https://github.com/ProteoWizard/pwiz
- name: ProteoWizard/pwiz-cpp
  role: Vendor raw-file readers that let Skyline read Waters Xevo TQ-S and other instrument
    formats directly for SRM reanalysis.
  repo: https://github.com/ProteoWizard/pwiz-cpp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:b7066eaea74e66c8cbaf9b1f32acceeee0b5391a3f408471b90d1a13570e881f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz
  tools:
  - ProteoWizard/pwiz-cpp
  - Skyline
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# srm-targeted-quantification

## Summary

Reprocess SRM (triple-quadrupole MRM) small-molecule data in Skyline by importing raw vendor files, integrating scheduled transitions, and computing absolute concentrations from an external calibration curve — demonstrated here by reanalyzing 76 ADNI serum Xevo TQ-S files from a Biocrates AbsoluteIDQ p180 plate against TargetLynx results.

## When to use

Use when raw SRM/MRM acquisition files (e.g. Waters Xevo TQ-S) must be independently quantified against a vendor result such as TargetLynx, or when a targeted amino-acid/metabolite panel (e.g. the 19 amino acids from a Biocrates p180 kit) needs calibrated concentrations outside vendor software.

## When NOT to use

- The input is already a processed feature/peak table from untargeted tools such as XCMS, MZmine, or MS-DIAL — Skyline's SRM skill extracts known targets from raw data, not reprocessing aligned feature lists.
- The acquisition is untargeted or data-independent MS/MS requiring deconvolution and feature finding — the targeted approach needs a predefined transition list.
- GC-MS or full ion-mobility small-molecule workflows — the paper notes support for these data types is still ongoing.

## Inputs

- Raw SRM/MRM acquisition files from a supported vendor (e.g. Waters Xevo TQ-S .raw files; 76 serum samples from one Biocrates p180 plate)
- Transition list defining the small-molecule targets (precursor/product m/z, charge, retention time, e.g. the 19 amino acids from the AbsoluteIDQ p180 kit)
- Calibrator/blank injection files defining the external calibration curve
- Heavy-labeled internal standard definitions for isotope-dilution quantification

## Outputs

- Integrated chromatographic peak areas for each transition per injection
- Calibrated concentrations for each analyte per sample (exported from Skyline)
- Regression/correlation statistics comparing Skyline concentrations against vendor values (e.g. slope 0.9837, 95% CI 0.9824–0.9850 vs TargetLynx)

## How to apply

Open a Skyline document and import the transition list for the target panel (for the p180 demo, 19 amino acids with their quantifier/qualifier transitions and heavy-labeled internal standards), selecting the vendor raw reader that matches the instrument (Waters Xevo TQ-S files in the ADNI serum example). Schedule the method around measured retention times — export with a 1-minute window centered on each retention time — then let Skyline pick and integrate peaks in the extracted ion chromatograms, checking qualifier ions. Apply the calibration model in Transition Settings matching the vendor fit (the paper uses linear regression with 1/x weighting, as in TargetLynx) and let Skyline calculate sample concentrations from the calibrators. Export the concentration table and compare against the vendor (TargetLynx) values; in the published benchmark the Skyline-vs-TargetLynx regression for all 19 amino acids gave a slope of 0.9837 (95% CI 0.9824–0.9850), which is the accuracy bar to reproduce.

## Related tools

- **Skyline** (Imports vendor SRM raw files, integrates scheduled transitions, applies the external calibration fit (linear with 1/x weighting), and exports calibrated concentrations for the 19 amino acids.) — https://github.com/ProteoWizard/pwiz
- **ProteoWizard/pwiz-cpp** (Vendor raw-file readers that let Skyline read Waters Xevo TQ-S and other instrument formats directly for SRM reanalysis.) — https://github.com/ProteoWizard/pwiz-cpp

## Examples

```
pwiz_tools\Skyline\build.bat  # build Skyline; then File > Import Transition List (19 amino acids), File > Import > Results (76 Xevo TQ-S .raw files), and export concentrations
```

## Evaluation signals

- Linear regression of Skyline-calculated concentrations against TargetLynx values for the 19 amino acids should reproduce a slope near 1 (reported 0.9837, 95% CI 0.9824–0.9850).
- Qualifier ion ratios and retention-time agreement with the scheduled window (±1-minute windows centered on measured RTs) should match expected chromatographic behavior.
- Calibration curve residuals must be consistent with the chosen fit (linear regression with 1/x weighting) before concentrations are accepted.
- The full 76-sample plate imports cleanly with the correct vendor reader and every transition has an integrated peak.
- Exported concentration table contains values for all target analytes and all samples, ready for correlation analysis.

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented.
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST are not yet implemented.
- No single tool is best for all workflows; GC-MS and all types of ion-mobility small-molecule data support are still ongoing.
- Support for latest vendor workflows may trail instrument vendor software.

## Evidence

---
name: external-calibration-quantification-setup
description: Use when a targeted MRM/HRMS metabolomics assay (e.g. Biocrates p180
  serum amino acids or arginine/polyamine panels) must yield calibrated concentrations
  rather than only peak areas, using heavy-labeled internal standards or external
  calibrants in Skyline.
when_to_use_negative:
- The goal is untargeted feature finding, alignment, and identification — use XCMS,
  MZmine, or MS-DIAL instead of Skyline's extract-known-targets approach
- Quantification relies on vendor-software-specific workflows where Skyline support
  lags (e.g. latest-generation GC-MS or ion mobility data)
- Input is already a processed feature/concentration table requiring no chromatographic
  re-integration
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: Skyline
  role: Defines small-molecule targets/transitions, integrates extracted ion chromatograms,
    and computes concentrations via external calibration against isotope-labeled internal
    standards
  repo: github.com/ProteoWizard/pwiz
- name: Panorama
  role: Web-based repository for sharing the assembled Skyline documents and methods
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:79619245cbe64cd41a373e4f3a702caca6e24d4536f23da40c894fb9a379c16f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Panorama
  - Skyline
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# External-calibration quantification setup

## Summary

Configure Skyline small-molecule documents so compound concentrations are calculated by isotope-dilution/external calibration against heavy-labeled internal standards, then exported with flexible quantification reports.

## When to use

Use when a targeted MRM/HRMS metabolomics assay (e.g. Biocrates p180 serum amino acids or arginine/polyamine panels) must yield calibrated concentrations rather than only peak areas, using heavy-labeled internal standards or external calibrants in Skyline.

## When NOT to use

- The goal is untargeted feature finding, alignment, and identification — use XCMS, MZmine, or MS-DIAL instead of Skyline's extract-known-targets approach
- Quantification relies on vendor-software-specific workflows where Skyline support lags (e.g. latest-generation GC-MS or ion mobility data)
- Input is already a processed feature/concentration table requiring no chromatographic re-integration

## Inputs

- Vendor raw LC-MS/MS files (e.g. ADNI serum amino acids acquired with the Biocrates AbsoluteIDQ p180 kit)
- Transition list with m/z, formula, adducts, and heavy-labeled internal standards
- Calibration standards / isotope-labeled internal standard definitions
- Measured retention times from a prior analysis

## Outputs

- Skyline document with calibrated concentrations of compounds of interest (e.g. small_molecule_workflow_template.sky)
- Retention-time-scheduled acquisition method with 1-minute windows centered on measured RTs
- Exported quantification report containing compound concentrations

## How to apply

Create a small-molecule Skyline document defining targets with m/z, formula, and adducts, and import the transition list together with the heavy-labeled internal standards (e.g. arginine/polyamine pathway targets). Configure quantification so concentrations are calculated by external calibration against the isotope-labeled internal standards, enabling collision energy optimization under Settings -> Transition Settings -> Prediction. After importing vendor raw files (e.g. ADNI p180 data), inspect each target as an extracted ion chromatogram, keep automatic peak integration and manually adjust integration boundaries only where automatic picks are unsatisfactory, since integration quality directly feeds the calibrated concentration. Export a retention-time-scheduled method (e.g. a 1-minute window centered on measured retention times) and a quantification report with the calibrated concentrations. Validate by benchmarking against comparable vendor software, as done in the paper with the AbsoluteIDQ p180 kit data.

## Related tools

- **Skyline** (Defines small-molecule targets/transitions, integrates extracted ion chromatograms, and computes concentrations via external calibration against isotope-labeled internal standards) — github.com/ProteoWizard/pwiz
- **Panorama** (Web-based repository for sharing the assembled Skyline documents and methods) — https://panoramaweb.org/

## Examples

```
Skyline: File > Import > Transition List (arginine/polyamine targets + heavy standards); configure external calibration; import ADNI p180 raw files; File > Export > Method with 1-minute RT window; save small_molecule_workflow_template.sky
```

## Evaluation signals

- Calibrated concentrations are present for all targets in the exported report, not just raw peak areas
- Heavy-labeled internal standards are paired with every endogenous target in the transition list
- Retention-time-scheduled method uses a 1-minute window centered on the measured retention times from prior analysis
- Integration boundaries match or improve upon automatic peak picking; manual adjustments are only made where automatic picks are unsatisfactory
- Results benchmark comparably against the instrument vendor software tool, as demonstrated with the p180 kit data

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows; GC-MS and ion mobility small-molecule support is still ongoing
- Fragmentation library workflows for METLIN, MassBank, HMDB, and NIST are not yet implemented, limiting orthogonal identifiability checks

## Evidence

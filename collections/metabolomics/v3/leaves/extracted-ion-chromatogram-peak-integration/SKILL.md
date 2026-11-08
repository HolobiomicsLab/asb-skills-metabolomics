---
name: extracted-ion-chromatogram-peak-integration
description: Use when vendor raw targeted MS files (SRM/MRM or HRMS) must be inspected
  as XICs and integrated per compound — with heavy-labeled internal standards — to
  yield peak areas for calibrated concentration calculation.
when_to_use_negative:
- Untargeted profiling that requires MS1 feature-finding, nonlinear peak alignment
  between runs, and identification — use XCMS, MZmine, or MS-DIAL instead
- Input is already a processed feature table or peak-area report rather than raw chromatographic
  data — re-integration in Skyline is not applicable
- Latest-generation or vendor-specific workflows where Skyline support has not yet
  been implemented and commercial vendor software is required
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: Open-source tool used to build the small-molecule document, view raw LC-MS
    data as extracted ion chromatograms, perform automatic/manual peak integration,
    and export retention-time-scheduled methods and calibrated quantification reports
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: Panorama
  role: Web-based repository used to share the assembled Skyline documents and processed
    targeted quantification data
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:46d16e34deb744cbf40566654cd7856851bba5a9c47de72f8f047e9d8f72a71a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Panorama
  - Skyline
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# extracted-ion-chromatogram-peak-integration

## Summary

Targeted small-molecule quantification in Skyline by importing vendor raw files (e.g. ADNI serum Biocrates p180 SRM data), extracting each target as an extracted ion chromatogram (XIC), keeping automatic peak picking, and manually adjusting integration boundaries where automatic picks are unsatisfactory.

## When to use

Use when vendor raw targeted MS files (SRM/MRM or HRMS) must be inspected as XICs and integrated per compound — with heavy-labeled internal standards — to yield peak areas for calibrated concentration calculation.

## When NOT to use

- Untargeted profiling that requires MS1 feature-finding, nonlinear peak alignment between runs, and identification — use XCMS, MZmine, or MS-DIAL instead
- Input is already a processed feature table or peak-area report rather than raw chromatographic data — re-integration in Skyline is not applicable
- Latest-generation or vendor-specific workflows where Skyline support has not yet been implemented and commercial vendor software is required

## Inputs

- Vendor raw LC-MS/MS targeted files (e.g. ADNI serum amino acids from the Biocrates AbsoluteIDQ p180 kit)
- Transition list with small-molecule targets: m/z, formula, adducts, products, and heavy-labeled internal standards
- Skyline small-molecule document with Transition Settings (collision energy prediction) and quantification calibration configured

## Outputs

- Extracted ion chromatograms per target with automatic and manually adjusted integration boundaries
- Integrated peak areas per transition/replicate
- Calibrated compound concentrations via external calibration against isotope-labeled internal standards
- Exported retention-time-scheduled method (1-minute window centered on measured RTs) and Skyline report export
- Shareable Skyline document (e.g. small_molecule_workflow_template.sky) publishable to Panorama

## How to apply

Create a small-molecule Skyline document and import a transition list including arginine/polyamine pathway targets together with their heavy labeled internal standards; the Skyline targeted approach is 'extracting known targets followed by detection and quantification' rather than the nontargeted feature-finding-in-MS1 workflow of XCMS/MZmine/MS-DIAL. Import the vendor raw files (e.g. ADNI p180 serum amino acid SRM data) and inspect each target as an extracted ion chromatogram, evaluating separation and peak integration. Keep Skyline's automatic peak integration by default, but manually adjust integration boundaries where the automatic picks are unsatisfactory — this is the decision point for each transition/replicate. For refined assays, export a retention-time-scheduled method using a 1-minute window centered on measured retention times from a prior analysis, optionally with collision energy optimization enabled via Settings -> Transition Settings -> Prediction. Quantify by stable-isotope dilution/external calibration against the heavy standards to calculate compound concentrations, then export results via flexible reports; validate against the instrument vendor software benchmark.

## Related tools

- **Skyline** (Open-source tool used to build the small-molecule document, view raw LC-MS data as extracted ion chromatograms, perform automatic/manual peak integration, and export retention-time-scheduled methods and calibrated quantification reports) — https://github.com/ProteoWizard/pwiz-cpp
- **Panorama** (Web-based repository used to share the assembled Skyline documents and processed targeted quantification data) — https://panoramaweb.org/

## Evaluation signals

- Every target transition has an integrated peak in each replicate, with integration boundaries manually reviewed where automatic picks were unsatisfactory
- Skyline-integrated peak areas/concentrations are benchmarked against a comparable instrument vendor software tool and agree in ranking and magnitude
- Exported method uses a 1-minute retention-time window centered on the measured RTs from the prior analysis
- Quantified concentrations derive from external calibration against the isotope-labeled heavy internal standards, with isotope-dilution ratios present in the report
- The assembled Skyline document (small_molecule_workflow_template.sky) reopens with transitions, CE settings, and calibration intact and can be shared via Panorama

## Limitations

- For latest-generation or vendor-specific workflows, Skyline may lag behind commercial software for a time until support can be implemented
- No single tool will be best for all workflows; expansion of GC-MS and ion mobility small molecule data support is still ongoing
- Fragmentation library workflows for Metlin, MassBank, HMDB, and NIST are not yet implemented for small-molecule library matching

## Evidence

---
name: small-molecule-transition-list-authoring
description: Use when building a targeted SRM/MRM or HRMS assay for small molecules
  or metabolites — e.g. arginine/polyamine or Biocrates p180-style panels — where
  a transition list with heavy internal standards must be defined, optimized, and
  exported as a scheduled acquisition method.
when_to_use_negative:
- Input is untargeted metabolomics data requiring feature finding, alignment, and
  identification — use XCMS, MZmine, or MS-DIAL instead, since Skyline's approach
  is extracting known targets followed by detection and quantification
- The workflow is GC-MS or ion-mobility small-molecule data, where Skyline support
  was still ongoing and the authors caution 'no single tool will be best for all workflows'
- A latest-generation or vendor-specific acquisition workflow is required, where Skyline
  may lag behind commercial vendor software until support is implemented
edam_operation: http://edamontology.org/operation_3665
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Skyline
  role: 'Primary tool: imports small-molecule transition lists with heavy internal
    standards, optimizes collision energies via the Transition Settings Prediction
    tab, exports RT-scheduled methods, and performs calibrated quantification'
  repo: github.com/ProteoWizard/pwiz
- name: Panorama
  role: Web-based repository for sharing the assembled Skyline targeted methods and
    processed documents
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:08ac21349e1e66da4195118c95a712ed072d4d55451b07a9f8737631ed229180
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Panorama
  - Skyline
  edam_operation: http://edamontology.org/operation_3665
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# small-molecule-transition-list-authoring

## Summary

Author and refine targeted small-molecule transition lists in Skyline by defining targets with m/z, formulas, adducts, and heavy-labeled internal standards, then optimizing collision energies and retention-time scheduling before method export. This operationalizes Skyline's extension from peptides to molecule-centric small-molecule quantification.

## When to use

Use when building a targeted SRM/MRM or HRMS assay for small molecules or metabolites — e.g. arginine/polyamine or Biocrates p180-style panels — where a transition list with heavy internal standards must be defined, optimized, and exported as a scheduled acquisition method.

## When NOT to use

- Input is untargeted metabolomics data requiring feature finding, alignment, and identification — use XCMS, MZmine, or MS-DIAL instead, since Skyline's approach is extracting known targets followed by detection and quantification
- The workflow is GC-MS or ion-mobility small-molecule data, where Skyline support was still ongoing and the authors caution 'no single tool will be best for all workflows'
- A latest-generation or vendor-specific acquisition workflow is required, where Skyline may lag behind commercial vendor software until support is implemented

## Inputs

- Transition list of small-molecule targets with precursor/product m/z, formula, adduct, and heavy-labeled internal standards (e.g. arginine/polyamine pathway molecules)
- Published transition list for a commercial kit panel (AbsoluteIDQ p180)
- Vendor raw LC-MS/MS files (e.g. ADNI serum amino acids acquired on the Biocrates p180 kit)
- Measured retention times from a prior Skyline analysis

## Outputs

- Retention-time-scheduled transition list or native vendor method with a 1-minute window centered on measured retention times
- Skyline document with transition settings, collision-energy-optimized transitions, and calibrated quantification configured (e.g. small_molecule_workflow_template.sky)
- Calibrated compound concentrations computed against isotope-labeled internal standards

## How to apply

Start by importing the transition list of target compounds (e.g. arginine and polyamine pathway molecules) into a Skyline small-molecule document, pairing each analyte with its heavy-labeled internal standard and specifying precursor/product m/z, formulas, and adducts. Enable collision energy optimization under Settings -> Transition Settings -> Prediction so that per-transition CE values are refined rather than fixed. Acquire an initial unscheduled run, inspect each target as an extracted ion chromatogram, and adjust integration boundaries where automatic peak picking is unsatisfactory to confirm transition quality. Then export a retention-time-scheduled method with a 1-minute window centered on the measured retention times from the prior analysis, either as a transition list or a native instrument method. Finally, configure quantification to calculate concentrations by external calibration against the isotope-labeled internal standards, and save the assembled document (e.g. small_molecule_workflow_template.sky) for reuse and sharing.

## Related tools

- **Skyline** (Primary tool: imports small-molecule transition lists with heavy internal standards, optimizes collision energies via the Transition Settings Prediction tab, exports RT-scheduled methods, and performs calibrated quantification) — github.com/ProteoWizard/pwiz
- **Panorama** (Web-based repository for sharing the assembled Skyline targeted methods and processed documents) — https://panoramaweb.org/

## Evaluation signals

- Exported method uses retention-time scheduling with a 1-minute window centered on the measured retention times from the prior analysis
- Every quantified target has an accompanying heavy-labeled internal standard transition, and concentrations are computed by external calibration against those standards
- Collision energy optimization is active in the saved document (Settings -> Transition Settings -> Prediction tab)
- Manually adjusted integration boundaries exist only where automatic peak picks were unsatisfactory; the rest retain automatic integration
- Assembled document (e.g. small_molecule_workflow_template.sky) round-trips: reopening it reproduces the same transition list and quantification settings

## Limitations

- Fragmentation library workflows for small-molecule spectral databases such as Metlin, MassBank, HMDB, and NIST were not yet implemented at time of publication
- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support can be implemented
- Expansion of support for GC-MS and all types of ion mobility small molecule data is ongoing; no single tool is best for all workflows

## Evidence

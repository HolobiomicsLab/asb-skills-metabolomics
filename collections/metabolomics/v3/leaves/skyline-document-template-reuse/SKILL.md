---
name: skyline-document-template-reuse
description: Use when new targeted metabolomics raw files (e.g. Sciex, Waters, Agilent,
  Bruker, Shimadzu, or Thermo data) need to be processed with the exact transition
  list, retention-time windows, and calibrated quantification of an already-validated
  Skyline small-molecule assay template rather than rebuilt.
when_to_use_negative:
- When the goal is hypothesis-free untargeted feature finding and alignment between
  runs — the article contrasts Skyline's targeted extract-then-quantify approach with
  non-targeted workflows that start with feature-finding in MS1 spectra followed by
  alignment and identification (use XCMS, MZmine, or MS-DIAL instead).
- When the assay requires fragmentation-library-based identification from Metlin,
  MassBank, HMDB, or NIST — the paper notes these fragmentation library workflows
  are not yet implemented in Skyline for small molecules.
- When the raw data require support for GC-MS or ion mobility small molecule acquisitions
  — the article states expansion of support for these data types is ongoing, so the
  template-reuse skill may not yet function correctly.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3315
tools:
- name: Skyline
  role: Open the template, import vendor raw data, review/adjust peak integration,
    and export calibrated quantitative results
  repo: https://github.com/ProteoWizard/pwiz
- name: Panorama Public
  role: Repository from which the Skyline document templates and reference quantitative
    results are downloaded and to which new processed results can be shared
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:393d69ed4db6799a2986df98f1173be246f674a58f5ebb03aef0ffd61a18cd8e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz
  tools:
  - Panorama Public
  - Skyline
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# skyline-document-template-reuse

## Summary

Reuse a pre-built Skyline document template (e.g. the Biocrates p180 amino acid assay or polyamine panel templates deposited on Panorama Public) by opening it in Skyline and importing new vendor raw data files, so that transitions, retention-time scheduling, and calibrated quantification are applied unchanged to the new data.

## When to use

Use when new targeted metabolomics raw files (e.g. Sciex, Waters, Agilent, Bruker, Shimadzu, or Thermo data) need to be processed with the exact transition list, retention-time windows, and calibrated quantification of an already-validated Skyline small-molecule assay template rather than rebuilt from scratch.

## When NOT to use

- When the goal is hypothesis-free untargeted feature finding and alignment between runs — the article contrasts Skyline's targeted extract-then-quantify approach with non-targeted workflows that start with feature-finding in MS1 spectra followed by alignment and identification (use XCMS, MZmine, or MS-DIAL instead).
- When the assay requires fragmentation-library-based identification from Metlin, MassBank, HMDB, or NIST — the paper notes these fragmentation library workflows are not yet implemented in Skyline for small molecules.
- When the raw data require support for GC-MS or ion mobility small molecule acquisitions — the article states expansion of support for these data types is ongoing, so the template-reuse skill may not yet function correctly.

## Inputs

- Skyline document template (.sky) — e.g. p180 amino acid assay or polyamine panel template from Panorama Public
- Vendor raw data files (e.g. Sciex, Waters, Agilent, Bruker, Shimadzu, Thermo LC-MS raw files)
- Reference quantitative results from the template (CSV/text exported from Skyline)

## Outputs

- Quantitative results report (peak areas, concentrations where calibration is defined) exported from Skyline as CSV
- Validated Skyline document containing the new data's extracted ion chromatograms and integration results

## How to apply

Download the deposited Skyline document template (for example the p180 kit amino acid assay or the polyamine panel template) plus its reference quantitative results from Panorama Public (https://panoramaweb.org/). Open the template in Skyline so that the imported transition list, m/z and adduct definitions, heavy-labeled internal standards, and 1-minute retention-time windows centered on measured retention times are loaded intact. Import the new vendor raw data files via Skyline's data import dialog — the document is vendor-agnostic because Skyline was expanded to handle selected reaction monitoring and high-resolution data from Sciex, Waters, Agilent, Bruker, Shimadzu, and Thermo. Inspect each target's extracted ion chromatogram, keep automatic peak picking where satisfactory and manually adjust integration boundaries only where the automatic picks are unsatisfactory, mirroring the documented workflow for the ADNI p180 files. Export the quantitative results (peak areas and concentrations where external calibration against isotope-labeled internal standards is defined) as a CSV report, then compare them against the template's reference results to confirm the same transitions yield consistent values within expected cross-vendor variability.

## Related tools

- **Skyline** (Open the template, import vendor raw data, review/adjust peak integration, and export calibrated quantitative results) — https://github.com/ProteoWizard/pwiz
- **Panorama Public** (Repository from which the Skyline document templates and reference quantitative results are downloaded and to which new processed results can be shared) — https://panoramaweb.org/

## Examples

```
In Skyline (GUI): File → Open → small_molecule_workflow_template.sky, then File → Import → Results… → select vendor raw files (.wiff / .rawd / .mzML); after review: File → Export → Report → CSV.
```

## Evaluation signals

- The same transitions produce peak areas and, where calibration is defined, concentrations consistent with the template's reference results within expected instrument-specific variability for cross-vendor data.
- Extracted ion chromatograms for each target show correctly extracted transitions with peak boundaries matching those in the reference document.
- The retention-time-scheduled method in the reused template retains the 1-minute window centered on measured retention times (no re-tuning needed).
- Heavy-labeled internal standards and external calibration settings carry over intact — the exported CSV includes concentrations calculated against isotope-labeled internal standards.
- The reused document remains vendor-agnostic: raw data from a non-original vendor (e.g. Thermo or Bruker) import and process without modification of the transition list or scheduling.

## Limitations

- Skyline may lag behind commercial instrument vendor software for latest-generation or vendor-specific workflows until support is implemented.
- No single tool will be best for all workflows; template reuse is appropriate only where the target panel, scheduling, and calibration logic of an existing template already match the experimental question.
- Expansion of support for GC-MS and ion mobility small molecule data is still ongoing, so templates relying on those acquisition modes may not be reusable yet.
- Fragmentation library workflows (Metlin, MassBank, HMDB, NIST) are not yet implemented, limiting template reuse for discovery-driven identification rather than predefined-target quantification.

## Evidence

- [results] Manual integration adjustment is part of the reuse workflow: [evidence span withheld — anchor status: not_found]

---
name: cross-vendor-raw-data-import
description: Use when targeted small-molecule raw data were acquired on an instrument
  vendor other than the one a Skyline template or method was originally built for,
  and quantitative extraction must run on Agilent, Bruker, Shimadzu, Thermo, Sciex,
  or Waters files.
when_to_use_negative:
- The analysis is untargeted feature finding (MS1 feature detection, alignment, identification)
  rather than targeted extraction of known molecules — Skyline's targeted approach
  starts from a known transition list
- The workflow depends on GC-MS or ion mobility small-molecule data, whose Skyline
  support was still ongoing at publication
- The vendor's latest-generation proprietary acquisition workflow is required and
  commercial vendor software already supports it — Skyline may lag until support is
  implemented
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Skyline
  role: Targeted small-molecule import, transition extraction, RT-scheduled peak picking,
    and quantification across vendor raw formats
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: ProteoWizard
  role: Underlying library providing direct reading of many vendor raw data formats
    used by Skyline's import
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: Panorama
  role: Repository for sharing the Skyline document templates and deposited raw files
    used as reference inputs
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:c18a6451fa5ca28ddf2602c6ebf4bcdd04708ded5344605c7a28f94f04e4acdd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Panorama
  - ProteoWizard
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-vendor-raw-data-import

## Summary

Import vendor raw mass spectrometry files from six major MS instrument vendors (Agilent, Bruker, Shimadzu, Thermo, Sciex, Waters) directly into Skyline small-molecule targeted analysis workflows via the ProteoWizard vendor readers, eliminating per-vendor software lock-in.

## When to use

Use when targeted small-molecule raw data were acquired on an instrument vendor other than the one a Skyline template or method was originally built for, and quantitative extraction must run on Agilent, Bruker, Shimadzu, Thermo, Sciex, or Waters files.

## When NOT to use

- The analysis is untargeted feature finding (MS1 feature detection, alignment, identification) rather than targeted extraction of known molecules — Skyline's targeted approach starts from a known transition list
- The workflow depends on GC-MS or ion mobility small-molecule data, whose Skyline support was still ongoing at publication
- The vendor's latest-generation proprietary acquisition workflow is required and commercial vendor software already supports it — Skyline may lag until support is implemented

## Inputs

- Vendor raw MS data files from Agilent, Bruker, Shimadzu, Thermo, Sciex, or Waters instruments
- Skyline document template (.sky) with preconfigured small-molecule transition list and RT scheduling
- Reference quantitative results from Panorama Public for comparison

## Outputs

- Extracted ion chromatograms and integrated peak areas for each small-molecule transition in Skyline
- CSV report of peak areas and calibrated concentrations
- Cross-vendor validation comparison against reference document values

## How to apply

Obtain a Skyline document template (e.g., the Biocrates p180 amino acid kit assay or the polyamine-panel document) with its transition list, retention-time scheduling (e.g., 1-minute windows centered on measured retention times), and peak-picking settings already configured, deposited on Panorama Public. Open the template in Skyline and directly import the additional-vendor raw data files; Skyline reads vendor formats through ProteoWizard, so no per-vendor software is required. Inspect extracted ion chromatograms to confirm the small-molecule transitions are correctly extracted and that peak boundaries match the reference document. Export peak areas and calibrated concentrations as a CSV report, then compare against the reference document's values, accepting runs where the same transitions yield consistent results within instrument-specific cross-vendor variability. On Linux builds, note the vendor reader limitation described in the README (Thermo, Bruker, Waters plus mzML/mzXML/MGF); run Windows for the other vendor SDK readers.

## Related tools

- **Skyline** (Targeted small-molecule import, transition extraction, RT-scheduled peak picking, and quantification across vendor raw formats) — https://github.com/ProteoWizard/pwiz-cpp
- **ProteoWizard** (Underlying library providing direct reading of many vendor raw data formats used by Skyline's import) — https://github.com/ProteoWizard/pwiz-cpp
- **Panorama** (Repository for sharing the Skyline document templates and deposited raw files used as reference inputs) — https://panoramaweb.org/

## Examples

```
pwiz_tools\Skyline\build.bat
```

## Evaluation signals

- All transitions in the template's transition list are extracted with chromatograms present (no missing-transition errors during raw import)
- Peak boundaries in the imported vendor data match the peak boundaries in the reference Skyline document
- Exported CSV report contains peak areas (and concentrations where calibration is defined) for every transition without nulls
- Imported values are consistent with reference-document values within instrument-specific cross-vendor variability
- Template's transition list, retention-time scheduling, and peak-picking settings are preserved unchanged after import

## Limitations

- On Linux, ProteoWizard supports only Thermo, Bruker, and Waters vendor formats plus mzML/mzXML/MGF; other vendor SDK readers (Sciex, Agilent, Shimadzu) are Windows-only per the README
- For latest-generation or vendor-specific workflows, Skyline may lag behind commercial vendor software until support is implemented
- No single tool is best for all workflows; Skyline's targeted approach requires a priori transition lists rather than untargeted discovery

## Evidence

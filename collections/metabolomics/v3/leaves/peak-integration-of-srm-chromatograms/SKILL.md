---
name: peak-integration-of-srm-chromatograms
description: Use when raw targeted SRM/MRM vendor files (e.g., Waters Xevo TQ-S from
  a Biocrates AbsoluteIDQ p180 plate) must be integrated and converted to concentrations
  with an open-source tool.
when_to_use_negative:
- The analysis is untargeted feature-finding/peak alignment rather than targeted extraction
  — the paper points such workflows to XCMS, MZmine, and MS-DIAL
- The data are GC-MS or ion mobility small molecule acquisitions, for which Skyline
  support is still ongoing per the discussion
- A fragmentation library workflow (METLIN, MassBank, HMDB, NIST) is needed, which
  is not yet implemented in Skyline
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Skyline
  role: 'Primary tool: imports vendor SRM raw files, integrates peaks from the imported
    transition list, applies calibration curves, and exports concentrations'
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: TargetLynx
  role: Waters vendor quantification software used as the benchmark comparator whose
    curve fit (linear regression, 1/x weighting) and reported concentrations the Skyline
    results must match
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:d23ec56d9861b375e7903f208117594375b29c63bfd61e071b2137623ccb79f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - Skyline
  - TargetLynx
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# peak-integration-of-srm-chromatograms

## Summary

Targeted integration and quantification of SRM/MRM chromatograms in Skyline, importing vendor raw data (e.g., Waters Xevo TQ-S), defining transitions from a target list, and applying calibrated curve fits to obtain analyte concentrations.

## When to use

Use when raw targeted SRM/MRM vendor files (e.g., Waters Xevo TQ-S from a Biocrates AbsoluteIDQ p180 plate) must be integrated and converted to concentrations with an open-source tool.

## When NOT to use

- The analysis is untargeted feature-finding/peak alignment rather than targeted extraction — the paper points such workflows to XCMS, MZmine, and MS-DIAL
- The data are GC-MS or ion mobility small molecule acquisitions, for which Skyline support is still ongoing per the discussion
- A fragmentation library workflow (METLIN, MassBank, HMDB, NIST) is needed, which is not yet implemented in Skyline

## Inputs

- Raw vendor SRM/MRM data files (e.g., 76 Waters Xevo TQ-S .raw files from a single Biocrates AbsoluteIDQ p180 plate)
- Transition list defining target molecules and quantifier/qualifier transitions (e.g., 19 amino acids from the p180 kit)
- Calibration curve parameters (curve fit type and weighting, e.g., linear regression with 1/x weighting)

## Outputs

- Integrated peak areas per transition per sample
- Calculated analyte concentrations from the calibration curve fit
- Correlation/regression statistics comparing results against vendor-reported values (e.g., slope and 95% CI)

## How to apply

Import the vendor raw SRM files into Skyline using the appropriate vendor reader (e.g., the Waters reader), then define targets by importing the transition list (e.g., the 19 amino acids from the p180 kit with their transitions and heavy labeled internal standards). Integrate the extracted ion chromatograms for each transition, applying Skyline's small molecule peak picking. Configure quantification with the calibration curve fit matching the vendor parameters — for the p180 benchmark this was linear regression with 1/x weighting — and export the calculated concentrations. Validate by regressing Skyline concentrations against the vendor tool (TargetLynx) values; the published benchmark achieved a slope of 0.9837 (95% CI 0.9824–0.9850) over 19 amino acids in 76 ADNI serum samples.

## Related tools

- **Skyline** (Primary tool: imports vendor SRM raw files, integrates peaks from the imported transition list, applies calibration curves, and exports concentrations) — https://github.com/ProteoWizard/pwiz-cpp
- **TargetLynx** (Waters vendor quantification software used as the benchmark comparator whose curve fit (linear regression, 1/x weighting) and reported concentrations the Skyline results must match)

## Evaluation signals

- Regression of Skyline-derived concentrations against TargetLynx-reported values for all 19 amino acids yields a slope of ~0.9837 (95% CI 0.9824–0.9850)
- Skyline's calibration curve fit (linear regression with 1/x weighting) is configured to match the TargetLynx quantification parameters exactly
- Peaks for each amino acid transition are visually inspected in extracted ion chromatograms for correct integration and separation
- All 76 plate samples import and process successfully via the Waters vendor reader with retention-time-scheduled transitions
- Exported concentration tables contain one value per amino acid per serum sample with no missing integration results

## Limitations

- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows; expansion of GC-MS and ion mobility small molecule data support is ongoing
- Fragmentation library workflows for Metlin, MassBank, HMDB, and NIST are not yet implemented

## Evidence

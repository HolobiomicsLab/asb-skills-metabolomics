---
name: dda-dia-feature-table-merging
description: Use when you have paired DDA (pooled QC) and DIA (individual biological
  sample) mzXML files from an LC-MS untargeted metabolomics run and need one consolidated
  feature-intensity table plus source-attributed MS2 spectra.
when_to_use_negative:
- Input is already a merged feature table or annotation result — DaDIA.R expects raw
  mzXML acquisitions per mode
- Data were acquired in a single mode only (pure DDA or pure DIA with no paired counterpart),
  since the merge step requires both feature lists
- Running XCMS older than version 3.11.4, which the article states is required to
  run DaDIA.R
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DaDIA.R
  role: Orchestrates centWave feature detection in both modes, merges DDA and DIA
    feature lists, and exports the intensity table and source-tagged MGF
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Performs centWave peak picking on the DDA and DIA mzXML files under DaDIA.R
    control (version 3.11.4+)
  repo: https://github.com/Huan Lab/DaDIA.git
- name: R
  role: Runtime environment for DaDIA.R (version 4.0 or above, with metaMS 1.25.1)
  repo: https://github.com/Huan Lab/DaDIA.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:1fb7d74950461148ad7d2a4f5a0936207b5af4919fb0367c31275a0e84fccf68
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# dda-dia-feature-table-merging

## Summary

Merge DDA-derived and DIA-(SWATH)-derived metabolic feature lists into a single metabolite-intensity table and export combined MS2 spectra tagged by acquisition source, using DaDIA.R on top of XCMS peak detection.

## When to use

Use when you have paired DDA (pooled QC) and DIA (individual biological sample) mzXML files from an LC-MS untargeted metabolomics run and need one consolidated feature-intensity table plus source-attributed MS2 spectra.

## When NOT to use

- Input is already a merged feature table or annotation result — DaDIA.R expects raw mzXML acquisitions per mode
- Data were acquired in a single mode only (pure DDA or pure DIA with no paired counterpart), since the merge step requires both feature lists
- Running XCMS older than version 3.11.4, which the article states is required to run DaDIA.R

## Inputs

- DDA mzXML files from pooled QC samples (e.g., Bruker Impact II UHR-QqTOF with Agilent 1290 LC)
- DIA-(SWATH) mzXML files from individual biological samples (e.g., human urine extracts)
- Number of DIA samples declared in DaDIA.R (line 38 of the script)

## Outputs

- metabolite_intensity_table.csv — merged DDA+DIA metabolite-intensity feature table
- ms2_spectra.mgf — combined MS2 spectra with DDA/DIA origin tagged in each MGF record header

## How to apply

Load the DDA and DIA-(SWATH) mzXML files into R (R 4.0+, XCMS 3.11.4+, metaMS 1.25.1) and detect metabolic features in both datasets with XCMS centWave as orchestrated by DaDIA.R. Merge the DDA-derived and DIA-derived feature lists into a single metabolite-intensity table (metabolite_intensity_table.csv), using the unique metabolic feature information from the DDA data to align and consolidate cross-mode features; DIA rescues DDA features lacking MS2 coverage (~22% of DDA features rescued in urine RP(+) data). Export the combined MS2 spectra to ms2_spectra.mgf, tagging each spectrum record's header with its DDA or DIA origin. The resulting hybrid table typically exceeds either mode alone (e.g., 7040 DaDIA features vs 1591 DDA and 6680 DIA-SWATH in the urine study).

## Related tools

- **DaDIA.R** (Orchestrates centWave feature detection in both modes, merges DDA and DIA feature lists, and exports the intensity table and source-tagged MGF) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Performs centWave peak picking on the DDA and DIA mzXML files under DaDIA.R control (version 3.11.4+)) — https://github.com/Huan Lab/DaDIA.git
- **R** (Runtime environment for DaDIA.R (version 4.0 or above, with metaMS 1.25.1)) — https://github.com/Huan Lab/DaDIA.git

## Examples

```
Rscript DaDIA.R  # after setting the number of DIA mzXML samples in line 38; outputs metabolite_intensity_table.csv and ms2_spectra.mgf with DDA/DIA origin tags
```

## Evaluation signals

- ms2_spectra.mgf parses as valid MGF format and every spectrum record carries a DDA or DIA origin tag in its header
- metabolite_intensity_table.csv exists and contains the required feature-intensity columns
- DDA/DIA split of exported MS2 spectra is consistent with reported proportions (63% DDA / 37% DIA in the urine test data)
- Merged feature count exceeds both single-mode counts (DaDIA 7040 vs DDA 1591 vs DIA-SWATH 6680 in the urine study), with DIA rescuing ~22% of DDA features (~344 of 1591)
- The number of samples declared in line 38 of DaDIA.R matches the real number of DIA mzXML files

## Limitations

- Feature extraction requires XCMS Development Version 3.11.4 or above; the number of samples hardcoded in line 38 of DaDIA.R must be manually updated to match the real number of DIA mzXML files
- Rescued features are not all reliable: manual EIC inspection showed 71% true-positive, 17% false-positive, and 12% undetermined rates; features with intensity < 3× the method-blank signal are background noise and must be removed
- Demonstrated on Bruker Impact II UHR-QqTOF / Agilent 1290 LC data (C18 RP(+) urine and ZIC-pHILIC ESI-negative leukemia plasma); transfer to other instruments or modes may need revalidation

## Evidence

- [results] Merge workflow: "63% of the MS2 spectra were generated from the DDA data and the remaining 37% were generated from the DIA data"
- [results] Feature counts: [evidence span withheld — anchor status: not_found]
- [results] Rescue rate: "the average number of rescued features from DDA is 344, which is approximately 22% of the average number of features (1591 features) detected in DDA data"
- [intro] Tool requirements: "R Version 4.0 or above, XCMS Development Version 3.11.4 or above, and metaMS Version 1.25.1 are required"
- [readme] Sample count README: "The number of samples in line 38 has to agree with the real number of DIA mzXML files"
- [methods] Background filter: "Metabolic features with intensities less than 3 × the signal of method blank were considered as background noise and removed"
- [results] Rescue validation: [evidence span withheld — anchor status: not_found]

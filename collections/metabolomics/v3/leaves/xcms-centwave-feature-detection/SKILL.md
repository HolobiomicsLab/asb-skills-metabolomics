---
name: xcms-centwave-feature-detection
description: Use when raw LC-MS data in mzXML format (e.g. DDA and DIA-(SWATH) acquisitions
  from a Bruker Impact II UHR-QqTOF) need peak picking and metabolic feature detection
  prior to merging features or annotating metabolites via MS2 spectral matching.
when_to_use_negative:
- Input is already a feature-intensity table (metabolite_intensity_table.csv) or an
  MGF MS2 file — feature detection has already been performed.
- Data are not LC-MS metabolomics acquisitions (e.g. proteomics or genomics data)
  where XCMS centWave peak picking does not apply.
- The R environment lacks XCMS Development Version 3.11.4 or above — DaDIA.R explicitly
  requires this version or higher.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: XCMS
  role: Provides the centWave peak-picking function used for metabolic feature detection
    in both DDA and DIA data; version 3.11.4 or above is required
  repo: https://github.com/Huan Lab/DaDIA.git
- name: DaDIA.R
  role: R program that orchestrates centWave-based feature extraction from DDA and
    DIA data and merges the resulting feature lists
  repo: https://github.com/Huan Lab/DaDIA.git
- name: R
  role: Runtime environment; R Version 4.0 or above is required to run DaDIA.R with
    XCMS and metaMS
  repo: https://github.com/Huan Lab/DaDIA.git
- name: metaMS
  role: Required R package (Version 1.25.1) in the DaDIA.R processing environment
  repo: https://github.com/Huan Lab/DaDIA.git
merged_aliases:
- xcms-centwave-feature-extraction
merged_alias_records:
- alias: xcms-centwave-feature-extraction
  slug: xcms-centwave-feature-extraction
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:52950ab859d5910684b5e668df83fd0011b2b750c56c35ef60c9e5408e48c482
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - IPO
  - R
  - XCMS
  - XCMS CentWave
  - metaMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# xcms-centwave-feature-detection

## Summary

Detect metabolic features from LC-MS DDA and DIA-(SWATH) mzXML files using XCMS's centWave peak-picking function, orchestrated by DaDIA.R, as the feature-extraction step that feeds the merged metabolite-intensity table and MS2 export of the DaDIA workflow.

## When to use

Use when raw LC-MS data in mzXML format (e.g. DDA and DIA-(SWATH) acquisitions from a Bruker Impact II UHR-QqTOF) need peak picking and metabolic feature detection prior to merging features or annotating metabolites via MS2 spectral matching.

## When NOT to use

- Input is already a feature-intensity table (metabolite_intensity_table.csv) or an MGF MS2 file — feature detection has already been performed.
- Data are not LC-MS metabolomics acquisitions (e.g. proteomics or genomics data) where XCMS centWave peak picking does not apply.
- The R environment lacks XCMS Development Version 3.11.4 or above — DaDIA.R explicitly requires this version or higher.

## Inputs

- DDA mzXML files (e.g. pooled QC samples analyzed in DDA mode)
- DIA-(SWATH) mzXML files (e.g. individual biological samples analyzed in DIA mode)
- DaDIA.R script from https://github.com/Huan Lab/DaDIA.git

## Outputs

- metabolite_intensity_table.csv (merged DDA/DIA metabolic feature intensity table)
- ms2_spectra.mgf (combined MS2 spectra from DDA and DIA data, tagged with acquisition source)
- xcmsSet/feature list objects from centWave peak picking for each acquisition mode

## How to apply

Load the DDA and DIA-(SWATH) mzXML files into R using XCMS version 3.11.4 or higher (with R 4.0+, metaMS 1.25.1, and all other packages updated to their newest versions) and run the centWave peak-picking function on both acquisition modes as orchestrated by DaDIA.R. This produces feature lists for the DDA data and the DIA data, which DaDIA.R then aligns and consolidates — using the unique metabolic feature information from the DDA data to guide cross-mode merging — into a single metabolite-intensity table, and the detected features supply the MS2 feature set exported as ms2_spectra.mgf. Downstream, filter detected features against method blanks (intensities below 3x the method-blank signal are treated as background noise and removed). Verify extraction succeeded by checking that the metabolite_intensity_table.csv has the required columns and that the MS2 provenance split is plausible; in the urine test data, 63% of exported MS2 spectra originated from the DDA data and 37% from the DIA data. Note that the number of samples at line 38 of the DaDIA.R script must agree with the real number of DIA mzXML files.

## Related tools

- **XCMS** (Provides the centWave peak-picking function used for metabolic feature detection in both DDA and DIA data; version 3.11.4 or above is required) — https://github.com/Huan Lab/DaDIA.git
- **DaDIA.R** (R program that orchestrates centWave-based feature extraction from DDA and DIA data and merges the resulting feature lists) — https://github.com/Huan Lab/DaDIA.git
- **R** (Runtime environment; R Version 4.0 or above is required to run DaDIA.R with XCMS and metaMS) — https://github.com/Huan Lab/DaDIA.git
- **metaMS** (Required R package (Version 1.25.1) in the DaDIA.R processing environment) — https://github.com/Huan Lab/DaDIA.git

## Examples

```
Rscript DaDIA.R  # after editing line 38 to set the number of DIA mzXML files; requires R >= 4.0 and XCMS >= 3.11.4
```

## Evaluation signals

- metabolite_intensity_table.csv exists and contains the required feature columns after the merged DDA/DIA feature lists are consolidated
- ms2_spectra.mgf parses as valid MGF format and each spectrum record is tagged with its DDA or DIA origin in the MGF header
- Spot-check the DDA/DIA source split of exported MS2 spectra against the reported 63% DDA / 37% DIA proportions from the urine test data
- Compare detected feature counts against the reported averages (1591 features for DDA, 6680 for DIA(SWATH), 7040 for DaDIA in urine RP(+) mode) as an order-of-magnitude sanity check
- Confirm the number of samples set at line 38 of DaDIA.R matches the real number of DIA mzXML files supplied

## Limitations

- Requires XCMS Development Version 3.11.4 or above; older XCMS versions will not run DaDIA.R.
- The number of samples in line 38 of the DaDIA.R script must agree with the real number of DIA mzXML files, or feature extraction will be misconfigured.
- Rescued features from DDA include false positives — manual EIC inspection of rescued features reported a 71% true-positive rate, 17% false-positive rate, and 12% undetermined, so feature lists should not be treated as ground truth without inspection.

## Evidence

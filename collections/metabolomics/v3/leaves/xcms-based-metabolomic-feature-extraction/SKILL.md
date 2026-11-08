---
name: xcms-based-metabolomic-feature-extraction
description: Use when raw LC-MS mzXML files from a hybrid DDA (pooled QC) / DIA (individual
  samples) metabolomics run need feature extraction before rescue and annotation,
  or when centWave-based peak-picking is the desired engine.
when_to_use_negative:
- Input is already a peak/feature table (e.g. CSV of m/z, RT, intensities) rather
  than raw LC-MS mzXML — no peak-picking is needed.
- Data are not untargeted LC-MS metabolomics (e.g. proteomics DDA search, targeted
  assays) where centWave-style centroided peak-picking is not the appropriate extraction
  mode.
- The environment cannot provide XCMS 3.11.4+ with R 4.0+; DaDIA.R explicitly requires
  these versions.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: XCMS
  role: Provides the centWave peak-picking function that DaDIA.R calls to extract
    metabolic features from both DDA and DIA mzXML files
  repo: https://github.com/Huan Lab/DaDIA.git
- name: DaDIA.R
  role: R driver script that wraps centWave-based feature extraction and links it
    to EIC-based feature rescue and MS2-based annotation
  repo: https://github.com/Huan Lab/DaDIA.git
- name: metaMS
  role: Required R package (version 1.25.1) listed among DaDIA.R's dependencies for
    the feature extraction/annotation workflow
- name: R
  role: Runtime environment (version 4.0 or above) in which the XCMS-based extraction
    is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:2659eb08c83ea1eac73d5fe3367521f3c690af1138af4ab09a6f9cfdc6d61709
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  - metaMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# xcms-based-metabolomic-feature-extraction

## Summary

Peak-picking of metabolic features from LC-MS raw data (mzXML from DDA and DIA/SWATH acquisitions) using the centWave function in XCMS, as implemented inside the DaDIA.R R workflow for hybrid DDA/DIA metabolomics.

## When to use

Use when raw LC-MS mzXML files from a hybrid DDA (pooled QC) / DIA (individual samples) metabolomics run need feature extraction before rescue and annotation, or when centWave-based peak-picking is the desired engine.

## When NOT to use

- Input is already a peak/feature table (e.g. CSV of m/z, RT, intensities) rather than raw LC-MS mzXML — no peak-picking is needed.
- Data are not untargeted LC-MS metabolomics (e.g. proteomics DDA search, targeted assays) where centWave-style centroided peak-picking is not the appropriate extraction mode.
- The environment cannot provide XCMS 3.11.4+ with R 4.0+; DaDIA.R explicitly requires these versions.

## Inputs

- mzXML files of DDA-mode runs on pooled QC samples
- mzXML files of DIA (SWATH)-mode runs on individual biological samples
- R environment with XCMS >= 3.11.4 and metaMS 1.25.1

## Outputs

- Metabolic feature lists (m/z, retention time, intensity per sample) extracted from DDA and DIA data
- Feature-intensity table consumed by DaDIA.R's rescue step and downstream metabolite annotation

## How to apply

Set up the R environment with R 4.0 or above, XCMS development version 3.11.4 or higher, and metaMS 1.25.1, plus up-to-date dependency packages, since DaDIA.R relies on these specific versions. Prepare the mzXML files for both the DDA runs on pooled QC samples and the DIA (SWATH) runs on individual biological samples, and make sure the number of samples declared at line 38 of the DaDIA.R script matches the real number of DIA mzXML files. Run the peak-picking step of DaDIA.R, which internally calls the centWave function of XCMS to extract metabolic features from both DDA and DIA mzXML files. The resulting feature lists (with m/z and retention time) feed DaDIA.R's rescue step, where DDA-unique features missing in DIA runs get intensities re-determined by EIC matching against the DIA raw data. Correct application is judged by the script running end-to-end on the demo mzXML files and by producing a feature/intensity table containing m/z, retention time, and intensity fields for downstream rescued-feature export.

## Related tools

- **XCMS** (Provides the centWave peak-picking function that DaDIA.R calls to extract metabolic features from both DDA and DIA mzXML files) — https://github.com/Huan Lab/DaDIA.git
- **DaDIA.R** (R driver script that wraps centWave-based feature extraction and links it to EIC-based feature rescue and MS2-based annotation) — https://github.com/Huan Lab/DaDIA.git
- **metaMS** (Required R package (version 1.25.1) listed among DaDIA.R's dependencies for the feature extraction/annotation workflow)
- **R** (Runtime environment (version 4.0 or above) in which the XCMS-based extraction is executed)

## Examples

```
Rscript DaDIA.R  # run on the demo DDA/DIA mzXML files from https://github.com/Huan Lab/DaDIA.git; edit line 38 to set the real number of DIA mzXML files
```

## Evaluation signals

- DaDIA.R must run end-to-end without errors on the authors' demo mzXML files (DDA + DIA/SWATH).
- The output feature table must contain m/z, retention time, and intensity fields per sample.
- The number of samples declared at line 38 of DaDIA.R must match the real count of DIA mzXML files; a mismatch breaks the extraction.
- As a sanity reference on urine RP(+) data, ~1591 features per DDA run and ~6680 per DIA(SWATH) run were detected downstream of this extraction step, with ~344 features rescued from DDA.
- Rescued EICs manually inspected in the paper showed 71% true-positive and 17% false-positive rates, giving an expected quality range for features fed by this extraction.

## Limitations

- Strict version requirements: R 4.0 or above, XCMS development version 3.11.4 or above, and metaMS 1.25.1; older or newer untested combinations may break the script.
- The script hard-codes the number of samples at line 38, which must be edited to match the actual number of DIA mzXML files.
- Extraction only yields feature tables; rescue of DDA-unique features and MS2-based annotation are separate downstream steps of DaDIA.R.

## Evidence

---
name: lc-hrms-roi-extraction-from-mzml
description: 'Use when building an LC-HRMS benchmark from raw *_POS.mzML files: an
  isotopologue prediction table (get_mz_table output) and target-molecule RT boundaries
  exist and chromatographic regions of interest must be extracted before peak validation.'
when_to_use_negative:
- Input is already a processed feature/peak table from XCMS, MZmine 2, MS-DIAL or
  similar — mzRAPP's ROI extraction works on raw centroided mzML files to build an
  independent benchmark, not on NPP outputs
- Files are profile-mode or non-mzML vendor formats — mzRAPP requires centroided mzML
  (convert/centroid via ProteoWizard MSconvert)
- No target molecules with known molecular composition and RT boundaries are available
  — the ROI extraction is driven entirely by the target file and enviPat predictions
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: mzRAPP
  role: Provides the get_ROIs function that performs the ROI extraction as the second
    step of the benchmark generation cascade
  repo: https://github.com/YasinEl/mzRAPP
- name: MSnbase
  role: Underlying functionality used to read mzML files (readMSData) and extract
    chromatograms for the target isotopologues
- name: XCMS
  role: Provides findmzROI and xcmsRaw functionality used to identify chromatographic
    regions of interest within the extracted ion chromatograms
- name: enviPat
  role: 'Upstream: predicts the isotopologue patterns (per molecule formula, adducts,
    instrument resolution) that define which m/z traces get ROI-extracted'
- name: data.table
  role: Data structure for the ROI table passed between the benchmark-generation functions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab231
    title: 'mzRAPP: a tool for reliability assessment of data pre-processing in non-targeted
      metabolomics'
  source_content_hash: sha256:fb4ced1b654560a77016c32fdf17a6fcb76898bcc04b0edaa6e8281131c9344e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YasinEl/mzRAPP
  tools:
  - MSnbase
  - R
  - XCMS
  - data.table
  - enviPat
  - mzRAPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lc-hrms-roi-extraction-from-mzml

## Summary

Extracts chromatographic regions of interest (ROIs) for enviPat-predicted isotopologues of target molecules directly from centroided mzML files using mzRAPP's get_ROIs function (built on MSnbase chromatogram extraction and XCMS findmzROI), as the intermediate step of benchmark generation.

## When to use

Use when building an LC-HRMS benchmark from raw *_POS.mzML files: an isotopologue prediction table (get_mz_table output) and target-molecule RT boundaries exist and chromatographic regions of interest must be extracted before peak validation.

## When NOT to use

- Input is already a processed feature/peak table from XCMS, MZmine 2, MS-DIAL or similar — mzRAPP's ROI extraction works on raw centroided mzML files to build an independent benchmark, not on NPP outputs
- Files are profile-mode or non-mzML vendor formats — mzRAPP requires centroided mzML (convert/centroid via ProteoWizard MSconvert)
- No target molecules with known molecular composition and RT boundaries are available — the ROI extraction is driven entirely by the target file and enviPat predictions

## Inputs

- data.table of enviPat-predicted isotopologue patterns from get_mz_table (target file with molecule, SumForm_c, main_adduct, user.rtmin, user.rtmax)
- centroided mzML files (e.g. 30, or at least 5, MTBLS267 *_POS.mzML files)
- sample-group csv (sample_name, sample_group)
- parameters: Min.PointsperPeak=6 scans, mz precision=6 ppm, mz accuracy=5 ppm

## Outputs

- data.table of chromatographic regions of interest (ROIs) for all predicted isotopologues across the input mzML files, serving as argument to find_bench_peaks

## How to apply

Run mzRAPP's get_ROIs on the data.table returned by get_mz_table, which contains enviPat-predicted isotopologue patterns for the target molecules (e.g. MTBLS267 targets with adducts M+NH4, M+Na, M+K and instrument resolution OrbitrapXL/Velos R60000@400). get_ROIs reads the centroided mzML files via MSnbase (readMSData) and extracts chromatograms, using XCMS findmzROI logic to delimit chromatographic regions. Set Min.PointsperPeak = 6 (minimum scans for a chromatographic peak), mz precision = 6 ppm (maximum spread of mass peaks in the mz dimension within one chromatogram), and mz accuracy = 5 ppm. Restrict extraction to the supplied user.rtmin/user.rtmax boundaries per molecule (optionally per FileName), with optional StartTime.EIC/EndTime.EIC overrides. Return the ROI data.table as input to find_bench_peaks for peak extraction and consistency filtering.

## Related tools

- **mzRAPP** (Provides the get_ROIs function that performs the ROI extraction as the second step of the benchmark generation cascade) — https://github.com/YasinEl/mzRAPP
- **MSnbase** (Underlying functionality used to read mzML files (readMSData) and extract chromatograms for the target isotopologues)
- **XCMS** (Provides findmzROI and xcmsRaw functionality used to identify chromatographic regions of interest within the extracted ion chromatograms)
- **enviPat** (Upstream: predicts the isotopologue patterns (per molecule formula, adducts, instrument resolution) that define which m/z traces get ROI-extracted)
- **data.table** (Data structure for the ROI table passed between the benchmark-generation functions)

## Examples

```
library(mzRAPP); callmzRAPP()  # then in the Generate Benchmark tab: select *_POS.mzML files, SampleGroups_MTBLS267.csv, Target_File_MTBLS267.csv, set Min. scans per peak = 6, mz precision = 6 ppm, mz accuracy = 5 ppm
```

## Evaluation signals

- Every ROI corresponds to an enviPat-predicted isotopologue of a target molecule and lies within the provided user.rtmin/user.rtmax boundaries
- Extracted chromatograms contain at least Min.PointsperPeak = 6 scans per candidate peak, with mass peaks within 6 ppm mz precision and 5 ppm mz accuracy
- The ROI data.table feeds find_bench_peaks without schema breaks — each cascade function returns a data.table consumed by the next
- End-to-end check: the benchmark generated from all 30 MTBLS267 mzML files contains 47 molecules, 157 features (all adducts and isotopologues) and 2870 peaks, matching the reference Benchmark.csv on ucloud
- ROIs are extracted only for molecules/adducts whose main adduct was detected (at least two isotopologues required)

## Limitations

- ROI extraction is bounded by user-supplied RT boundaries; peaks will still be rejected later if isotopic information does not fit, even with provided boundaries
- At least two isotopologues (including the theoretically most abundant) must be findable for a molecule/adduct to contribute to the final benchmark
- Requires centroided mzML input; profile-mode or vendor formats must first be converted and centroided via ProteoWizard MSconvert

## Evidence

- [other] ROI extraction cascade: [evidence span withheld — anchor status: not_found]
- [readme] README extraction description: "mzRAPP extracts and validates chromatographic peaks for which boundaries are provided for all (enviPat predicted) isotopologues of those target molecules directly from mzML files."
- [readme] README parameter definition: [evidence span withheld — anchor status: not_found]
- [methods] Methods file download step: "Download all 30 mzML files (at least five if you do not want to process that many) ending on "_POS.mzML" from the repository MTBLS267 on"
- [methods] Methods benchmark tab: [evidence span withheld — anchor status: not_found]

---
name: lc-hrms-peak-feature-processing
description: Use when raw LC-HRMS profiles from xenobiotic biotransformation assays
  (e.g. human liver S9 incubations) must be deconvolved into an annotated m/z–RT–MS2
  feature table before downstream CMDN analysis.
when_to_use_negative:
- Input is already a processed feature table with m/z, RT, and MS2 spectra attached
  — skip to CMDN's RCS/SSN steps instead
- Data are GC-MS, direct-infusion MS, or NMR rather than LC-HRMS — the feature-detection
  and RT-alignment logic of this pipeline does not apply
- The goal is targeted quantification of a known analyte list rather than untargeted
  discovery of unknown biotransformation products
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-Dial
  role: Peak detection, feature alignment, deconvolution, and MS2-spectrum extraction
    from raw LC-HRMS files to build the input feature table for CMDN
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: CMDN (R scripts)
  role: Downstream consumer of the processed feature table; R packages tidyverse,
    readxl, msentropy, MSMSsim, CluMSID, OrgMassSpecR are used in the pipeline
  repo: https://github.com/LeaveMeNotTonight/CMDN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:c8ae8492df518ff7009e89594c9dcfc77a667d73b463c3be7ef2060ee460612d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - CMDN (R scripts)
  - LC-HRMS
  - MS-Dial
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8260869565217391
n_claims: 23
---

# LC-HRMS peak/feature processing

## Summary

Converts raw LC-HRMS data into a feature table with accurate m/z, retention time (RT), and MS2 spectra — the prerequisite feature representation for CMDN's reaction-guided metabolite annotation pipeline.

## When to use

Use when raw LC-HRMS profiles from xenobiotic biotransformation assays (e.g. human liver S9 incubations) must be deconvolved into an annotated m/z–RT–MS2 feature table before downstream CMDN analysis.

## When NOT to use

- Input is already a processed feature table with m/z, RT, and MS2 spectra attached — skip to CMDN's RCS/SSN steps instead
- Data are GC-MS, direct-infusion MS, or NMR rather than LC-HRMS — the feature-detection and RT-alignment logic of this pipeline does not apply
- The goal is targeted quantification of a known analyte list rather than untargeted discovery of unknown biotransformation products

## Inputs

- Raw LC-HRMS data files (MS1 + MS2) from human liver S9 incubation samples (e.g. batch 1 malathion/sulprofos incubations) and matched blank/controls

## Outputs

- Aligned feature table with accurate m/z, retention time (RT), and MS2 spectra per feature
- Differential metabolite space (DMS): sample-enriched features after blank/differential filtering

## How to apply

Run MS-Dial (ver. 4.80) over the raw LC-HRMS files from the enzymatic S9 incubation to detect features, align them across conditions, and export a feature table carrying accurate m/z, retention time, and associated MS2 spectra per feature. Perform blank-subtraction/differential filtering to isolate sample-enriched features (the differential metabolite space, DMS) so that incubation-derived metabolite features are retained while background contaminants are removed. Keep MS2 spectra paired to each precursor feature since downstream CMDN computes pairwise spectral-entropy similarities on them to build the MS2 spectrum similarity network (SSN). Export the aligned feature table (e.g. to CSV/Excel readable with R's readxl/tidyverse) so it can be consumed directly by the R-based CMDN scripts. Verify that reported metabolites carry plausible RT values and that parent and metabolite features co-occur in the same aligned table before proceeding to RCS/SSN.

## Related tools

- **MS-Dial** (Peak detection, feature alignment, deconvolution, and MS2-spectrum extraction from raw LC-HRMS files to build the input feature table for CMDN) — https://github.com/LeaveMeNotTonight/CMDN
- **CMDN (R scripts)** (Downstream consumer of the processed feature table; R packages tidyverse, readxl, msentropy, MSMSsim, CluMSID, OrgMassSpecR are used in the pipeline) — https://github.com/LeaveMeNotTonight/CMDN

## Examples

```
msdial --mode DIAMetA --input raw_s9_batch1.mzML --output features_malathion_sulprofos.csv
```

## Evaluation signals

- Every exported feature row carries a non-missing accurate m/z and RT value
- MS2 spectra are attached to (or retrievable for) precursor features destined for SSN similarity computation
- Sample-enriched features persist after blank/differential filtering (DMS) while background features are suppressed
- The feature table is loadable in R via readxl/tidyverse and row/feature counts match MS-Dial alignment output
- Parent-pesticide features and their reaction-derived candidate features (e.g. malathion −CH2, −S+O variants) co-occur in the aligned table

## Limitations

- MS-Dial ver. 4.80 is stated as the compatible version; other processing software may produce feature tables incompatible with the CMDN R scripts
- R packages (tidyverse, CluMSID, CluMSIDdata, OrgMassSpecR, pheatmap, reshape2, MSMSsim, msentropy, readxl) must all be installed before running the CMDN scripts
- Peak picking/alignment parameters are not specified in the provided context; results depend on chosen RT-tolerance and m/z-tolerance settings
- Traditional bottom-up library-matching cannot find previously unknown metabolites; CMDN's reaction-based candidature space (RCS) is required after this step for truly novel annotations

## Evidence

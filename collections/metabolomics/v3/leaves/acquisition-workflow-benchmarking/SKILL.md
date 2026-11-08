---
name: acquisition-workflow-benchmarking
description: Use when comparing LC-MS data acquisition workflows (e.g. DDA vs DIA/SWATH
  vs DaDIA) using detected-feature and MS2-feature counts from the same sample set.
when_to_use_negative:
- When only one acquisition mode was run and no comparative workflow data exists —
  the skill requires a side-by-side comparison.
- When the goal is metabolite identification or annotation quality (use a spectral-matching
  skill instead) — this skill measures acquisition coverage, not identification success.
- When the input is already a published feature-count summary; instead, use it to
  independently recompute relative increases from raw per-sample counts.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: XCMS
  role: Feature extraction (peak picking/alignment) on the DDA, DIA(SWATH), and DaDIA
    datasets so per-workflow feature counts are comparable.
  repo: https://github.com/Huan Lab/DaDIA.git
- name: DaDIA.R
  role: Automated feature extraction and metabolite annotation of the combined DDA/DIA
    dataset used for the DaDIA arm of the benchmark.
  repo: https://github.com/Huan Lab/DaDIA.git
- name: metaMS
  role: Supporting R package for the feature-extraction environment (Version 1.25.1
    required).
- name: Bruker Impact II UHR-QqTOF
  role: Mass spectrometer used for the urine RP(+) benchmarking runs under each acquisition
    mode.
- name: Agilent 1290 Infinity II LC system
  role: Chromatographic platform coupled to the MS for the compared acquisition workflows.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:5953dd497b41d00ac2382b7a375e686df73f9d082af3b884f88aa7d760c499d0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - Agilent 1290 Infinity II LC system
  - Bruker Impact II UHR-QqTOF
  - DaDIA.R
  - XCMS
  - metaMS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# acquisition-workflow-benchmarking

## Summary

Benchmarks LC-MS acquisition workflows (DDA, DIA/SWATH, DaDIA) side-by-side by extracting detected-feature and MS2-feature counts, then reporting relative increases so a hybrid acquisition can be judged against conventional modes.

## When to use

Use when comparing LC-MS data acquisition workflows (e.g. DDA vs DIA/SWATH vs DaDIA) using detected-feature and MS2-feature counts from the same sample set.

## When NOT to use

- When only one acquisition mode was run and no comparative workflow data exists — the skill requires a side-by-side comparison.
- When the goal is metabolite identification or annotation quality (use a spectral-matching skill instead) — this skill measures acquisition coverage, not identification success.
- When the input is already a published feature-count summary; instead, use it to independently recompute relative increases from raw per-sample counts.

## Inputs

- Feature extraction outputs (detected feature counts and MS2 feature counts) from each acquisition workflow on the same sample set, e.g. mzXML files for DDA, DIA(SWATH), and DaDIA
- Per-workflow feature tables filtered against method blanks (3× blank signal threshold)

## Outputs

- Structured comparison table (CSV) with one row per acquisition workflow and columns for detected features, MS2 features, and relative increases
- Supplemental audit metrics: share of MS2 spectra from DDA vs DIA data, rescue-feature true-positive/false-positive rates, and paired-test p-values

## How to apply

Run the competing acquisition workflows (e.g. DDA, DIA/SWATH, and DaDIA) on the same samples (human urine RP(+) in the paper) processed identically: feature extraction with XCMS 3.11.4 or above and DaDIA.R for the hybrid data, after filtering metabolic features whose intensities are below 3× the method blank signal. Extract per-workflow averages for total detected features and MS2 features, then recompute relative increases (e.g. DaDIA raised total features by 342% over DDA and 5% over DIA(SWATH); MS2 features by 413% and 22%) and check them against paired statistical tests (p ≤ 0.001). Report a structured comparison table (one row per workflow) with columns for detected features, MS2 features, and relative increases, plus supplemental metrics like the share of MS2 spectra from DDA vs DIA data and rescue-feature true-positive/false-positive rates (71%/17%) to support the feature-rescue audit.

## Related tools

- **XCMS** (Feature extraction (peak picking/alignment) on the DDA, DIA(SWATH), and DaDIA datasets so per-workflow feature counts are comparable.) — https://github.com/Huan Lab/DaDIA.git
- **DaDIA.R** (Automated feature extraction and metabolite annotation of the combined DDA/DIA dataset used for the DaDIA arm of the benchmark.) — https://github.com/Huan Lab/DaDIA.git
- **metaMS** (Supporting R package for the feature-extraction environment (Version 1.25.1 required).)
- **Bruker Impact II UHR-QqTOF** (Mass spectrometer used for the urine RP(+) benchmarking runs under each acquisition mode.)
- **Agilent 1290 Infinity II LC system** (Chromatographic platform coupled to the MS for the compared acquisition workflows.)

## Examples

```
Rscript DaDIA.R feature_extraction.R --input urine_dadia_mzxml/ --xcms_version 3.11.4 --metaMS_version 1.25.1 --blank_filter 3
```

## Evaluation signals

- Recomputed relative increases (e.g. total features 342% over DDA, 5% over DIA(SWATH); MS2 features 413% and 22%) must match the article's reported percentages.
- Every cell in the comparison table must reproduce the article's reported averages: 1591/6680/7040 detected features and 956/4024/4901 MS2 features for DDA/DIA(SWATH)/DaDIA.
- MS2-spectrum provenance split should sum to 100% (63% DDA + 37% DIA in the urine data).
- Feature-rescue audit should show ~22% of DDA features rescued (344 of 1591) with 71% true-positive, 17% false-positive, and 12% undetermined EICs.
- Paired-test significance must hold (p ≤ 0.001 for total features, p < 0.001 for MS2 features).

## Limitations

- Benchmark metrics assume all workflows were run on the same LC-MS platform and sample set; cross-instrument comparisons are not supported by the article.
- Rescue-feature true/false-positive rates were determined by manual EIC inspection (12% undetermined), so the audit is not fully automated.
- Feature counts depend on blank filtering (3× method blank) and dot-product thresholds (>0.7), so changing these cutoffs breaks comparability with the published numbers.

## Evidence

- [results] The convertMSP.R is currently available on Github: "The convertMSP.R is currently available on Github (https://github.com/Huan Lab/DaDIA.git) with a user manual"
- [results] average number of detected features: [evidence span withheld — anchor status: not_found]
- [methods] Metabolic features with intensities less than 3 × the signal of method blank were considered as background noise and removed: "Metabolic features with intensities less than 3 × the signal of method blank were considered as background noise and removed"
- [methods] Only the annotated metabolites with a dot product score higher than 0.7 were kept for further analysis: "Only the annotated metabolites with a dot product score higher than 0.7 were kept for further analysis"
- [results] 63% of the MS2 spectra were generated from the DDA data: "63% of the MS2 spectra were generated from the DDA data and the remaining 37% were generated from the DIA data"
- [results] The average true-positive rate: [evidence span withheld — anchor status: not_found]

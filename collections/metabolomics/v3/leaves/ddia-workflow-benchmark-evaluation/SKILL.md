---
name: ddia-workflow-benchmark-evaluation
description: Use when evaluating whether a hybrid DDA/DIA metabolomic acquisition
  strategy improves feature detection, quantification, and annotation over conventional
  DDA or DIA(SWATH) workflows, especially for urine or plasma LC-MS datasets on QqTOF
  instruments.
when_to_use_negative:
- Input data is already a post-acquisition feature table (no need to re-run acquisition-benchmarking);
  use a standard feature-summarization skill instead
- The instrument mode is already DaDIA and the goal is downstream metabolite annotation
  only, not workflow-level benchmarking against DDA/DIA baselines
- Data is generated on a non-QqTOF instrument without matching DDA/DIA(SWATH) reference
  runs, so the paper's rescue-rate baselines (344 rescued features, 71% TPR) are not
  comparable
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DaDIA.R
  role: R program that processes DDA and DIA mzXML files to extract metabolic features
    and perform MS2-based metabolite annotation; used to generate the combined DaDIA
    feature table for benchmarking
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: Feature-detection engine called by DaDIA.R to extract metabolic peaks from
    raw mzXML files; version 3.11.4 or higher required
- name: metaMS
  role: R package required alongside XCMS for metabolite annotation within DaDIA.R;
    version 1.25.1
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:a6dacd0ce542b71dfa511eefff8c763fbcb35665fe4beed5fec7665c91ca177e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  - metaMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ddia-workflow-benchmark-evaluation

## Summary

A skill for benchmarking and validating the DaDIA hybrid DDA/DIA workflow's performance: comparing feature counts, MS2 spectra counts, and rescued-feature reliability across DDA, DIA(SWATH), and DaDIA modes in LC-MS metabolomics.

## When to use

Use when evaluating whether a hybrid DDA/DIA metabolomic acquisition strategy improves feature detection, quantification, and annotation over conventional DDA or DIA(SWATH) workflows, especially for urine or plasma LC-MS datasets on QqTOF instruments.

## When NOT to use

- Input data is already a post-acquisition feature table (no need to re-run acquisition-benchmarking); use a standard feature-summarization skill instead
- The instrument mode is already DaDIA and the goal is downstream metabolite annotation only, not workflow-level benchmarking against DDA/DIA baselines
- Data is generated on a non-QqTOF instrument without matching DDA/DIA(SWATH) reference runs, so the paper's rescue-rate baselines (344 rescued features, 71% TPR) are not comparable

## Inputs

- DDA-mode mzXML files from pooled QC samples
- DIA/SWATH-mode mzXML files from individual biological samples
- DDA feature list (mz values from pooled QC runs)
- Method blank mzXML files for background filtering

## Outputs

- JSON record of urine_rescue_metrics with average rescued features (344), true-positive rate (0.71), false-positive rate (0.17)
- Per-workflow feature-count table comparing DDA, DIA(SWATH), and DaDIA
- Per-workflow MS2 feature-count table
- Annotated metabolite count with dot-product score > 0.7 threshold
- EIC inspection logs for manually validated rescued features

## How to apply

Load DDA and DIA(SWATH) mzXML files (e.g., the DaDIA repository demo urine samples) and process them with DaDIA.R in R using XCMS ≥3.11.4 and metaMS 1.25.1 to extract metabolic features. Compare total feature counts and MS2 feature counts across DDA, DIA(SWATH), and DaDIA workflows — the paper reports averages of 1591/6680/7040 features and 956/4024/4901 MS2 features, representing a 342% and 413% increase over DDA respectively. Identify features present in the DDA feature list that are rescued in the combined DaDIA data and compute the average number of rescued features per sample (344 in the urine study, ~22% of DDA features). Validate rescue reliability by manual extracted-ion chromatogram (EIC) inspection, recording the true-positive rate (71% good peak shapes) and false-positive rate (17% poor/noisy shapes). Filter features against method blanks (intensity < 3× blank signal removed), apply significance criteria (FC > 1.5 or < 0.67, p < 0.05), and retain only metabolite annotations with dot-product scores > 0.7 when comparing annotated metabolite counts across workflows.

## Related tools

- **DaDIA.R** (R program that processes DDA and DIA mzXML files to extract metabolic features and perform MS2-based metabolite annotation; used to generate the combined DaDIA feature table for benchmarking) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (Feature-detection engine called by DaDIA.R to extract metabolic peaks from raw mzXML files; version 3.11.4 or higher required)
- **metaMS** (R package required alongside XCMS for metabolite annotation within DaDIA.R; version 1.25.1)

## Examples

```
Rscript DaDIA.R --input DDA_and_DIA_mzXMLs --xcms_version 3.11.4
```

## Evaluation signals

- Verify the reported urine-study rescue metrics JSON: average rescued features == 344, true-positive rate == 0.71, false-positive rate == 0.17 (12% undetermined)
- Feature-count sanity check: DaDIA total feature count must exceed both DDA and DIA(SWATH) counts (DDA 1591 < DIA(SWATH) 6680 < DaDIA 7040)
- MS2-feature count ratio: DaDIA MS2 features (4901) should be > DDA (956) by ~413% and > DIA(SWATH) (4024) by ~22%
- Confirm every background feature removed has intensity < 3× the method-blank signal before benchmarking refined counts
- Manually inspect a random subset of rescued-feature EICs and confirm ≥ 60% good chromatographic peak shapes (consistent with reported 71% TPR)

## Limitations

- Rescue-reliability metrics are from manual EIC inspection of a specific urine study in RP(+) mode; generalization to other matrices, ionization modes, or instruments requires re-derivation
- The DaDIA.R workflow requires matching sample counts in the mzXML list (line 38) and specific R/XCMS/metaMS versions; version mismatches or wrong sample counts break feature extraction
- The paper's DDA-vs-DIA comparison depends on pooled-QC DDA runs for MS2 rescue; workflows without pooled-QC design cannot be benchmarked with these metrics

## Evidence

- [results] Rescue reliability: [evidence span withheld — anchor status: not_found]
- [results] MS2 benchmark: [evidence span withheld — anchor status: not_found]
- [methods] Instrument and LC context: [evidence span withheld — anchor status: not_found]

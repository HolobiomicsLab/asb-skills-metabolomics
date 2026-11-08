---
name: total-ion-chromatogram-processing
description: Use when raw untargeted metabolomics samples (.mzML/.mzXML/.CDF, qTOF/orbitrap/FT-ICR)
  must be characterized by their TIC peak structure before parameter optimization,
  e.g. to establish per-sample peak scan bounds or estimate the Group difference parameter.
when_to_use_negative:
- The input is already a processed feature table or peak list — no TIC re-analysis
  is needed
- The task requires comparing EIC peaks across samples to estimate retention time
  correction; the divide-and-conquer TIC stage explicitly does not support that
- The user expects fully automated group assignment — the Group difference parameter
  estimated during TIC peak detection requires a non-automatable cross sample comparison
  and user inspection of the sliding window output
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoTuner
  role: R package implementing the sliding-window TIC peak detection and linear-model
    boundary refinement
  repo: https://github.com/crmclean/Autotuner
- name: XCMS
  role: Downstream consumer of the per-sample TIC-derived peak bounds for centWave
    parameter estimation
- name: MZmine2
  role: Downstream consumer of the per-sample TIC-derived peak bounds for centWave
    parameter estimation
- name: R
  role: Execution environment; AutoTuner requires R ≥ 3.6
  repo: https://github.com/crmclean/Autotuner
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/812370
    title: 'AutoTuner: High fidelity, robust, and rapid parameter selection for metabolomics
      data processing'
  source_content_hash: sha256:fee7d2338b49a1835f2586dac48fc34092763a524b00ebde3d90b5595d5eebf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/crmclean/Autotuner
  tools:
  - AutoTuner
  - MZmine2
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# total-ion-chromatogram-processing

## Summary

Detects dataset-specific peaks in each sample's total ion chromatogram (TIC) via a sliding-window algorithm, then refines peak boundaries by iteratively fitting linear models until R2 ≥ 0.8 — the first stage of AutoTuner that establishes per-sample TIC peak scan bounds used to estimate metabolomics peak-picking parameters.

## When to use

Use when raw untargeted metabolomics samples (.mzML/.mzXML/.CDF, qTOF/orbitrap/FT-ICR) must be characterized by their TIC peak structure before parameter optimization, e.g. to establish per-sample peak scan bounds or estimate the Group difference parameter.

## When NOT to use

- The input is already a processed feature table or peak list — no TIC re-analysis is needed
- The task requires comparing EIC peaks across samples to estimate retention time correction; the divide-and-conquer TIC stage explicitly does not support that
- The user expects fully automated group assignment — the Group difference parameter estimated during TIC peak detection requires a non-automatable cross sample comparison and user inspection of the sliding window output

## Inputs

- Raw untargeted metabolomics data files converted to .mzML, .mzXML, or .CDF (≥3 samples required by AutoTuner)
- Per-sample total ion chromatogram (TIC) traces extracted from raw files
- Metadata spreadsheet mapping sample names to experimental factors

## Outputs

- Per-sample TIC peak boundary table (start and end scan indices per accepted peak)
- Linear-model fit statistics (R2) for each refined peak bound
- Per-sample peak structure used downstream for centWave parameter estimation and the Group difference estimate

## How to apply

Load each sample's raw converted file and extract the total ion chromatogram (TIC). Run the ThresholdingAlgo sliding-window analysis (parameterized by lag, threshold, and influence) over each TIC to flag local deviations as candidate peaks. For each candidate, fit a linear model to the first and last three points bounding the peak and refine its boundaries; if the model's R2 falls below 0.8 or the endpoint is not a local maximum, expand the ending bound by one scan and rerun the model until the criterion is satisfied. Record the accepted per-sample start/end scan indices. This divide-and-conquer stage deliberately does not compare extracted-ion-chromatogram (EIC) peaks across samples, which is why retention time correction cannot be estimated here; the non-automatable cross-sample comparison during TIC peak detection is where the Group difference parameter is estimated.

## Related tools

- **AutoTuner** (R package implementing the sliding-window TIC peak detection and linear-model boundary refinement) — https://github.com/crmclean/Autotuner
- **XCMS** (Downstream consumer of the per-sample TIC-derived peak bounds for centWave parameter estimation)
- **MZmine2** (Downstream consumer of the per-sample TIC-derived peak bounds for centWave parameter estimation)
- **R** (Execution environment; AutoTuner requires R ≥ 3.6) — https://github.com/crmclean/Autotuner

## Examples

```
Rscript -e 'library(Autotuner); at <- createAutotuner("mzml_dir", "metadata.csv"); peaks <- ThresholdingAlgo(at, lag=5, threshold=1, influence=0.2)'}
```

## Evaluation signals

- Every retained TIC peak entry carries a linear-model R2 value ≥ 0.8, per the bound-expansion acceptance criterion
- Each sample's accepted peak table contains explicit start and end scan indices (not just peak centers)
- The sliding-window parameters (lag, threshold, influence) are recorded so the TIC detection is reproducible
- Peak detection runs on the TIC trace only — no EIC cross-sample comparison was performed at this stage
- Output validates that the input raw data were pre-converted from proprietary instrument formats (.mzML, .mzXML, or .CDF) rather than vendor files

## Limitations

- The sliding window analysis requires user input during the first portion of AutoTuner; replacing it with a more sensitive peak detector is proposed as future work
- Robustness of the Group difference parameter estimate could not be statistically verified because it is estimated through a non-automatable cross sample comparison during TIC peak detection
- AutoTuner is not parallelized, so TIC peak detection over large sample sets is single-threaded
- The divide-and-conquer design does not compare EIC peaks across samples, so retention time correction algorithms cannot be estimated from this stage

## Evidence

- [other] AutoTuner detects peaks with a sliding window analysis: [evidence span withheld — anchor status: not_found]
- [other] if the model's R2 is below 0.8: [evidence span withheld — anchor status: not_found]
- [readme] requires at least 3 samples of raw data: [evidence span withheld — anchor status: external_doc]
- [discussion] Group difference parameter estimated during TIC peak detection: "this parameter is estimated through a non-automatable cross sample comparison during the TIC peak detection step of the algorithm."
- [discussion] loss of comparing EIC peaks across samples: [evidence span withheld — anchor status: not_found]

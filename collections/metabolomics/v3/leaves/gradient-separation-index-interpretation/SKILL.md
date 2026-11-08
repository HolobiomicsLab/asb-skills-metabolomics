---
name: gradient-separation-index-interpretation
description: Use when evaluating or comparing LC-MS gradient performance across runs
  — e.g. reporting GSI before/after gradient optimization, or selecting the next gradient
  in a Bayesian-optimization loop for small-molecule LC-MS(/MS) analysis.
when_to_use_negative:
- The input is already a processed feature table (mz/rt/intensity matrix) with no
  raw chromatographic data — sepEfficiency needs retention times of top MS1 signals
  from the raw run, not pre-summarized features.
- Analyzing GC-MS or non-LC separation where an elution gradient (and hence BAGO's
  gradient search space) does not apply.
- A single gradient run with no baseline or alternative-gradient comparison — the
  article's workflow uses GSI to compare original vs. optimized gradients, so an isolated
  value has limited interpretive value.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: BAGO
  role: Python package providing the separation-efficiency computation (`sepEfficiency`),
    MS1/MS2 spectrum extraction, Gaussian-process fit, and acquisition-function-driven
    next-gradient proposal
  repo: https://github.com/HuanLab/bago
- name: pyopenms
  role: Underlying module used by BAGO to read mzML/mzXML raw files and extract spectra
- name: scikit-learn
  role: GaussianProcessRegressor used to fit the separation-efficiency objective during
    gradient optimization
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:a684d149ce8f9bbd98692f44e43810ecd909560cfcf0bb770d313e9b1bc28ac1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/huaxuyu/bago
  repo_url: https://github.com/HuanLab/bago
  tools:
  - BAGO
  - Python
  - pyopenms
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gradient-separation-index-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets BAGO's separation efficiency metric (GSI) for untargeted LC-MS gradients: an omics-scale, normalized [0,1] score computed from retention times of top MS1 signals, used to compare original vs. BAGO-optimized gradients and to guide Bayesian-optimization iterations.

## When to use

Use when evaluating or comparing LC-MS gradient performance across runs — e.g. reporting GSI before/after gradient optimization, or selecting the next gradient in a Bayesian-optimization loop for small-molecule LC-MS(/MS) analysis.

## When NOT to use

- The input is already a processed feature table (mz/rt/intensity matrix) with no raw chromatographic data — sepEfficiency needs retention times of top MS1 signals from the raw run, not pre-summarized features.
- Analyzing GC-MS or non-LC separation where an elution gradient (and hence BAGO's gradient search space) does not apply.
- A single gradient run with no baseline or alternative-gradient comparison — the article's workflow uses GSI to compare original vs. optimized gradients, so an isolated value has limited interpretive value.

## Inputs

- Raw LC-MS/MS data in mzML or mzXML format (one file per gradient run)
- Retention-time sequence of top MS1 signals (rtSeq) and the expected retention-time window (rtRange)
- Gradient parameters / search space for the Bayesian-optimization model

## Outputs

- Separation efficiency / global separation index (GSI) value in [0, 1] per run
- Per-run GSI comparison table (original vs. optimized gradient) with feature counts, MS/MS counts, and annotated-compound counts (e.g. bago_optimization_results.csv)
- Next proposed gradient from the Gaussian-process acquisition function

## How to apply

Extract MS1 scans from raw mzML/mzXML files with the BAGO package (backed by pyopenms), identify the top signals in the base-peak chromatogram via `getBPCData`/`findTopSignals`, and obtain their retention times. Feed the retention-time series into `sepEfficiency(rtSeq, rtRange)` to compute the separation efficiency, which yields a normalized global separation index (GSI) in [0, 1]; a low GSI (e.g. 0.0345 for the mouse cecum HILIC study's original gradient) indicates poor compound separation, while an optimized gradient can raise it substantially (to 0.171 in that same study). Report GSI alongside downstream metrics — metabolic feature counts, unique MS/MS spectra, and annotated compounds meeting quantification criteria — because the article's validation shows GSI gains track feature discovery (e.g. +11.0% features, +26.0% high-quality features in mouse cecum HILIC). In the optimization loop, the GSI (or per-signal separation efficiency) is the objective fitted to a `sklearn` GaussianProcessRegressor and maximized via an acquisition function (default 'ei') in `computeNextGradient` to propose the next gradient to run.

## Related tools

- **BAGO** (Python package providing the separation-efficiency computation (`sepEfficiency`), MS1/MS2 spectrum extraction, Gaussian-process fit, and acquisition-function-driven next-gradient proposal) — https://github.com/HuanLab/bago
- **pyopenms** (Underlying module used by BAGO to read mzML/mzXML raw files and extract spectra)
- **scikit-learn** (GaussianProcessRegressor used to fit the separation-efficiency objective during gradient optimization)

## Examples

```
from bago import sepEfficiency; gsi = sepEfficiency(rtSeq, rtRange)
```

## Evaluation signals

- Every extracted GSI value falls within [0, 1], consistent with its definition as a normalized separation score.
- Reported percentage gains (e.g. mouse cecum HILIC: GSI 0.0345 → 0.171, features 1660 → 1894, +11.0%; high-quality features 1000 → 1260, +26.0%) are arithmetically consistent with the absolute before/after counts.
- Per-study rows (six validated studies: mouse cecum HILIC, human urine, serum metabolomics, serum lipidomics, Drosophila HILIC and reversed-phase) match article-reported reference values in a consolidated table (e.g. bago_optimization_results.csv).
- The optimized-gradient GSI is strictly greater than the original-gradient GSI in each validated study, and the number of annotated compounds meeting quantification criteria increases (up to +48.5% reported).
- Reproducibility: rerunning `sepEfficiency` on the same raw mzML/mzXML file yields the same GSI for a fixed rtRange.

## Limitations

- GSI is defined for LC gradient-based separations of small molecules; it is not a general-purpose chromatographic quality metric outside the BAGO gradient-optimization context.
- The Python package and workflow are tied to Python 3.8+ and mzML/mzXML input; other raw formats are not supported by the described pipeline.
- Raw MS data used for the six validation studies are only available on demand.
- Windows software and the Python package are maintained in separate releases; the Python package is the route for programmatic GSI interpretation and pipeline extension.

## Evidence

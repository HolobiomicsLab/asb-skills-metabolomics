---
name: smoothed-trace-local-maxima-peak-candidate-detection
description: Use when preprocessing LC-QQQ-MS MRM raw data (mzML) and candidate chromatographic
  peaks must be detected and scored before ML-based or averaged peak classification,
  e.g. inside automRm's process_batch() workflow.
when_to_use_negative:
- The data are not MRM mode LC-MS (e.g. full-scan/DDA) — automRm's peak picking operates
  on quantifier/qualifier MRM traces and for optimal performance requires at least
  one qualifier in addition to the quantifier
- A validated peak table already exists and only the ML peak-reporting step is needed
  — this skill covers candidate generation, not the reporting model
- Samples are excessively dirty, very-low-input, or from a new LC method, where the
  authors retreat to manual peak review for selected metabolites
tools:
- name: automRm
  role: R package implementing smoothed-trace local-maxima peak candidate detection
    and 20 quality-score computation for MRM chromatograms
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses the mzML files to extract quantifier and qualifier MRM chromatograms
    per metabolite per sample
- name: openxlsx
  role: Reads the metabolite database (xlsx) used to map transitions and expected
    RTs during peak picking
- name: R
  role: Implementation language (developed/tested in R 3.6.0 and 4.0.2) for the peak-picking
    and quality-score code
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:23201ff6cb92ad1fc3f29429f56fd4dcb66db45f72ed1458c559f15fa1d2c9e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - mzR
  - openxlsx
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# smoothed-trace-local-maxima-peak-candidate-detection

## Summary

automRm's peak-picking stage: local maxima are detected in a smoothed version of the quantifier MRM trace, candidate peaks are ranked by peak height, and each candidate is described by 20 quality scores (peak shape, quantifier/qualifier agreement, RT deviation) for downstream ML evaluation or averaging.

## When to use

Use when preprocessing LC-QQQ-MS MRM raw data (mzML) and candidate chromatographic peaks must be detected and scored before ML-based or averaged peak classification, e.g. inside automRm's process_batch() workflow.

## When NOT to use

- The data are not MRM mode LC-MS (e.g. full-scan/DDA) — automRm's peak picking operates on quantifier/qualifier MRM traces and for optimal performance requires at least one qualifier in addition to the quantifier
- A validated peak table already exists and only the ML peak-reporting step is needed — this skill covers candidate generation, not the reporting model
- Samples are excessively dirty, very-low-input, or from a new LC method, where the authors retreat to manual peak review for selected metabolites

## Inputs

- Batch of LC-QQQ-MS raw data files in mzML format (e.g. the automRm HILIC demodata batch)
- Metabolite database (metabdb.xlsx) defining MRM quantifier/qualifier transitions and expected RT
- Parameter file (update_prm.tsv) specifying smoothing degree and number of candidate peaks to evaluate

## Outputs

- qslog_initial.tsv: per-transition, per-sample table of candidate peaks with all 20 quality scores
- Per-candidate peak classification scores (averaged quality scores when no ML model is supplied)

## How to apply

Parse the mzML batch with mzR, extracting quantifier and qualifier MRM chromatograms for each metabolite in each sample of the batch. Smooth each quantifier trace and detect local maxima in the smoothed trace — the smoothing degree and the number of candidate peaks evaluated per metabolite per sample are taken from the parameter file (update_prm.tsv). Rank candidate peaks by peak height and compute 20 quality scores per candidate characterizing peak shape, quantifier/qualifier agreement, and deviation from expected RT, including QS_T.o (quantifier height ratio to highest outside point), QS_To2 (same on qualifier trace), QS_T.h/QS_T.l (qualifier height ratios to upper/lower peak borders), QS_cor123 (quantifier–qualifier correlation), QS_gauss (correlation with a Gaussian peak), and QS_dRT/QS_dRTs (RT deviation with and without predicted RT shift relative to the reference sample). These are the same quality scores later consumed by the peak reporting ML model; where no ML model is supplied, average the quality scores to obtain each candidate's peak classification score. Write the per-transition, per-sample table of candidates and all quality scores to qslog_initial.tsv.

## Related tools

- **automRm** (R package implementing smoothed-trace local-maxima peak candidate detection and 20 quality-score computation for MRM chromatograms) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses the mzML files to extract quantifier and qualifier MRM chromatograms per metabolite per sample)
- **openxlsx** (Reads the metabolite database (xlsx) used to map transitions and expected RTs during peak picking)
- **R** (Implementation language (developed/tested in R 3.6.0 and 4.0.2) for the peak-picking and quality-score code) — https://gitlab.gwdg.de/joerg.buescher/automrm

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')
```

## Evaluation signals

- qslog_initial.tsv contains quality-score fields (QS_T.o, QS_To2, QS_T.h, QS_T.l, QS_cor123, QS_gauss, QS_dRT, QS_dRTs, etc.) for every candidate peak of every transition and sample
- With no ML model supplied, each candidate's classification score equals the average of its quality scores (the paper's stated no-ML-model fallback)
- Candidate peaks are ranked by peak height and the configured number of candidates per metabolite per sample (from update_prm.tsv) is evaluated
- QS_dRTs values are consistent with the metabolite RT shifts computed by cross-correlation against the reference sample
- Each local maximum found in the smoothed quantifier trace yields exactly one scored candidate entry

## Limitations

- For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier; adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode
- Very dirty or very-low-input samples, or new LC methods, may still require retreat to manual peak review for selected metabolites
- Smoothing degree and candidate count are user-set parameters (update_prm.tsv); poor settings can miss true peaks or flood the candidate list
- Reaching quality similar to manual peak review requires sufficient training on relevant raw data downstream

## Evidence

- [methods] local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model: "the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [discussion] For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier.: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [methods] automRm depends on several packages such as mzR7 for parsing of mzML files: "automRm depends on several packages such as mzR7 for parsing of mzML files"

---
name: quantifier-qualifier-transition-agreement-characterization
description: Use when MRM raw data (mzML) must be scored per peak candidate on quantifier/qualifier
  transition agreement and RT deviation — e.g., to rank peak candidates or feed an
  ML peak-reporting model.
when_to_use_negative:
- The acquisition method contains only quantifier transitions with no qualifier —
  automRm requires at least one qualifier in addition to the quantifier for optimal
  performance, so transition-agreement scores are not meaningful.
- Data are full-scan/DDA LC-MS rather than MRM-mode QQQ chromatograms; the skill is
  built around quantifier/qualifier MRM traces.
- Peak candidates already carry curated quality scores and only manual peak review
  is intended — the article notes manual review remains the fallback for excessively
  dirty samples, very-low-input samples, or new LC methods.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: automRm
  role: R package whose peak-picking module smooths the quantifier trace, ranks candidate
    peaks by height, and computes the 20 quality scores including quantifier/qualifier
    agreement metrics
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses the mzML raw files to extract quantifier and qualifier MRM chromatograms
    for each metabolite in each sample
- name: openxlsx
  role: Reads/writes the xlsx metabolite database (e.g., metabdb.xlsx) that defines
    transitions and expected RTs used in scoring
- name: R
  role: Runtime environment for automRm (developed and tested in R 3.6.0 and 4.0.2)
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:fb8f80ece1e00b5bb030f9d2a60f423e12cddc765f27553b0c12668e05112c4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - mzR
  - openxlsx
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# quantifier-qualifier-transition-agreement-characterization

## Summary

Computes per-candidate MRM peak quality scores that quantify agreement between the quantifier transition and its qualifier transitions (QS_To2, QS_T.h, QS_T.l, QS_cor123), together with peak-shape (QS_gauss) and retention-time deviation (QS_dRT, QS_dRTs) scores, as implemented in the automRm peak-picking module for LC-QQQ-MS data.

## When to use

Use when MRM raw data (mzML) must be scored per peak candidate on quantifier/qualifier transition agreement and RT deviation — e.g., to rank peak candidates or feed an ML peak-reporting model.

## When NOT to use

- The acquisition method contains only quantifier transitions with no qualifier — automRm requires at least one qualifier in addition to the quantifier for optimal performance, so transition-agreement scores are not meaningful.
- Data are full-scan/DDA LC-MS rather than MRM-mode QQQ chromatograms; the skill is built around quantifier/qualifier MRM traces.
- Peak candidates already carry curated quality scores and only manual peak review is intended — the article notes manual review remains the fallback for excessively dirty samples, very-low-input samples, or new LC methods.

## Inputs

- mzML files of an LC-QQQ-MS MRM batch (e.g., the automRm demo data set from gitlab.gwdg.de/joerg.buescher/demodata)
- Metabolite database (metabdb.xlsx) defining quantifier/qualifier transitions and expected retention times
- Parameter file (update_prm.tsv) specifying smoothing degree and number of candidate peaks evaluated per metabolite

## Outputs

- Per-transition, per-sample table of candidate peaks with all 20 quality scores (QS_To2, QS_T.h, QS_T.l, QS_cor123, QS_gauss, QS_dRT, QS_dRTs, etc.) written to qslog_initial.tsv
- Peak classification score per candidate (average of the quality scores when no ML model is supplied)

## How to apply

Parse quantifier and qualifier MRM chromatograms for each metabolite in each sample from mzML files using mzR, then smooth each quantifier trace, detect local maxima in the smoothed trace, and rank candidate peaks by peak height (smoothing degree and number of candidates evaluated per metabolite are read from update_prm.tsv). For each candidate, compute the 20 automRm quality scores characterizing peak shape, quantifier/qualifier agreement, and deviation from the expected RT — including QS_To2 (qualifier height ratio to the highest outside point), QS_T.h and QS_T.l (qualifier height ratios to upper and lower peak borders), QS_cor123 (quantifier–qualifier correlation), QS_gauss (correlation with a Gaussian peak), and QS_dRT/QS_dRTs (deviation from expected RT, with and without the RT shift predicted by cross-correlation against the reference sample). If no ML model is supplied, average the quality scores to obtain each candidate's peak classification score, and write the per-transition, per-sample candidate table with all scores to qslog_initial.tsv. The rationale is that ML-based peak picking and peak reporting rely on these same quality scores, so their consistent computation is the basis of both candidate detection and downstream reporting.

## Related tools

- **automRm** (R package whose peak-picking module smooths the quantifier trace, ranks candidate peaks by height, and computes the 20 quality scores including quantifier/qualifier agreement metrics) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses the mzML raw files to extract quantifier and qualifier MRM chromatograms for each metabolite in each sample)
- **openxlsx** (Reads/writes the xlsx metabolite database (e.g., metabdb.xlsx) that defines transitions and expected RTs used in scoring)
- **R** (Runtime environment for automRm (developed and tested in R 3.6.0 and 4.0.2)) — https://gitlab.gwdg.de/joerg.buescher/automrm

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')
```

## Evaluation signals

- qslog_initial.tsv contains quality-score fields for every candidate peak of every metabolite in every sample.
- Candidate peaks are ranked by peak height, matching the paper's description of peak picking.
- With no ML model supplied, each candidate's classification score is derivable as the average of its quality scores (the paper's stated no-ML fallback).
- QS_dRTs values reflect the RT shift predicted by simultaneous cross-correlation against the reference sample, so samples with large global RT shifts still score near zero after shift correction.
- QS_cor123 is computable for every candidate because automRm requires at least one qualifier in addition to the quantifier.

## Limitations

- For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier; adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode.
- Sufficient training on relevant raw data is required for automRm to reach a quality similar to manual peak review; results depend on the choice of training data set and ML algorithm.
- Smoothing degree and the number of evaluated candidates per metabolite are taken from update_prm.tsv, so misconfigured parameters directly affect which candidates get scored.

## Evidence

- [methods] local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] peak reporting ML model uses same quality scores: "the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [discussion] automRm requires at least one qualifier: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [abstract] ML for detection of chromatographic peaks and quality control: "Using machine learning (ML) for detection of chromatographic peaks and quality control of reported results enables the automatic recognition of complex patterns in raw data."

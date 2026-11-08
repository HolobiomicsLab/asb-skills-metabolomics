---
name: chromatographic-peak-quality-score-computation
description: Use when preprocessing LC-QQQ-MS raw data in MRM mode and you need per-candidate
  peak quality metrics (shape, quantifier/qualifier coelution, RT deviation) before
  ML-based peak reporting.
when_to_use_negative:
- Data are not MRM-mode LC-MS with quantifier/qualifier transitions — automRm requires
  at least one qualifier in addition to the quantifier for optimal performance
- The goal is final peak reporting on new or very-low-input samples without relevant
  training data; the paper notes retreat to manual peak review may be needed for excessively
  dirty samples, very-low-input samples, or new LC methods
- Raw files are not parseable as mzML (e.g. vendor-native formats not converted)
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: automRm
  role: R package implementing smoothing, local-maximum peak-picking, and quality-score
    computation for MRM chromatograms
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses mzML files to extract quantifier and qualifier MRM chromatograms for
    each metabolite in each sample
- name: openxlsx
  role: Reads the metabolite database (xlsx) defining transitions and expected RTs
- name: caret
  role: Machine learning backend that consumes the quality scores for peak picking
    and peak reporting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:262f2b2b0a58e6b00b661a47894808044949891741435dcd36483678f2e7bca7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - caret
  - mzR
  - openxlsx
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chromatographic-peak-quality-score-computation

## Summary

Computation of 20 chromatographic quality scores for candidate MRM peaks detected by automRm, characterizing peak shape, quantifier/qualifier agreement, and RT deviation, which feed the downstream ML peak-reporting model (or are averaged when no ML model is supplied).

## When to use

Use when preprocessing LC-QQQ-MS raw data in MRM mode and you need per-candidate peak quality metrics (shape, quantifier/qualifier coelution, RT deviation) before ML-based peak reporting.

## When NOT to use

- Data are not MRM-mode LC-MS with quantifier/qualifier transitions — automRm requires at least one qualifier in addition to the quantifier for optimal performance
- The goal is final peak reporting on new or very-low-input samples without relevant training data; the paper notes retreat to manual peak review may be needed for excessively dirty samples, very-low-input samples, or new LC methods
- Raw files are not parseable as mzML (e.g. vendor-native formats not converted)

## Inputs

- mzML files of an LC-QQQ-MS batch recorded in MRM mode (e.g. from an Agilent 1290 Infinity II UHPLC coupled to an Agilent 6495 QQQ-MS)
- Metabolite database (metabdb.xlsx) defining quantifier/qualifier transitions and expected RTs
- Parameter file (update_prm.tsv) specifying smoothing degree and number of candidate peaks evaluated per metabolite per sample

## Outputs

- qslog_initial.tsv: per-transition, per-sample table of candidate peaks with all 20 quality scores
- Peak classification score per candidate (average of the quality scores when no ML model is supplied)

## How to apply

Parse quantifier and qualifier MRM chromatograms from mzML files with mzR for each metabolite in each sample of the batch. Smooth each quantifier trace (degree from update_prm.tsv), detect local maxima in the smoothed trace, rank candidate peaks by peak height, and evaluate the configured number of candidates per metabolite per sample. For each candidate compute 20 quality scores including QS_T.o (quantifier height ratio to highest outside point), QS_To2 (same on the qualifier trace), QS_T.h/QS_T.l (qualifier height ratios to upper/lower peak borders), QS_cor123 (quantifier–qualifier correlation), QS_gauss (correlation with a Gaussian peak), and QS_dRT/QS_dRTs (deviation from expected RT, with and without predicted RT shift relative to the reference sample). Where no ML model is supplied, average the quality scores to obtain each candidate's peak classification score; otherwise the same scores feed the peak-picking and peak-reporting ML models built with caret.

## Related tools

- **automRm** (R package implementing smoothing, local-maximum peak-picking, and quality-score computation for MRM chromatograms) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses mzML files to extract quantifier and qualifier MRM chromatograms for each metabolite in each sample)
- **openxlsx** (Reads the metabolite database (xlsx) defining transitions and expected RTs)
- **caret** (Machine learning backend that consumes the quality scores for peak picking and peak reporting)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de'); automRm::process_batch()
```

## Evaluation signals

- qslog_initial.tsv contains quality-score fields for every candidate peak, including QS_T.o, QS_To2, QS_T.h, QS_T.l, QS_cor123, QS_gauss, QS_dRT, and QS_dRTs
- Classification scores are derivable as averages of the quality scores (the paper's stated no-ML-model fallback)
- Candidate peaks are ranked by peak height as local maxima detected in the smoothed quantifier trace
- Number of candidates evaluated per metabolite per sample matches the parameter in update_prm.tsv
- QS_dRT values reflect deviation from the expected RT with and without the predicted RT shift relative to the reference sample

## Limitations

- For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier; adding qualifiers reduces the number of analytes detectable simultaneously in MRM mode
- With insufficient relevant training data, quality similar to manual peak review is not reached; manual review may be needed for excessively dirty samples, very-low-input samples, or new LC methods
- Smoothing degree and candidate count must be configured per method via update_prm.tsv; default settings may not suit all chromatography

## Evidence

- [methods] Peak picking with quality scores: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] Quality-score-bearing candidates feed ML reporting: "the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [methods] Dependency on mzR for mzML parsing: "automRm depends on several packages such as mzR7 for parsing of mzML files"
- [discussion] Qualifier requirement: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
- [discussion] Quality parity with manual review depends on training data: "With sufficient training on relevant raw data, results from automRm can reach a similar level of quality as manual peak review."

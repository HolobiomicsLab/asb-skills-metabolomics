---
name: retention-time-deviation-scoring-against-expected-rt
description: Use when preprocessing LC-QQQ-MS MRM batch data (mzML) and you need to
  score candidate chromatographic peaks by their deviation from expected RT, including
  alignment-corrected deviation, for downstream ML peak classification.
when_to_use_negative:
- No expected RT is available for the analyte in the metabolite database, so deviation
  cannot be defined
- Raw data are not MRM-mode LC-MS chromatograms (e.g. full-scan DDA/DIA), where quantifier/qualifier
  transition logic and RT-shift cross-correlation do not apply
- You only need final reported peaks, not per-candidate RT-deviation diagnostics —
  use the full process_batch() reporting step instead
tools:
- name: automRm
  role: R package implementing RT-shift estimation by cross-correlation, quadratic
    RT-shift fitting, and QS_dRT/QS_dRTs computation within its peak-picking pipeline
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: mzR
  role: Parses mzML files to extract the quantifier and qualifier MRM chromatograms
    on which RT deviation is measured
- name: openxlsx
  role: Reads the metabolite database (metabdb.xlsx) supplying expected RT values
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:fb6ac935562f86c96fc476f427b328c92b911eb72b1082b9c9c9bd998579c887
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
yes_rate: 1.0
n_claims: 11
---

# Retention-time deviation scoring against expected RT

## Summary

Computes the QS_dRT and QS_dRTs quality scores in automRm, which quantify how far a candidate MRM chromatographic peak deviates from the metabolite's expected retention time, with and without the sample-specific predicted RT shift relative to a reference sample. These scores feed peak picking and peak reporting ML models as part of the 20-score QS system.

## When to use

Use when preprocessing LC-QQQ-MS MRM batch data (mzML) and you need to score candidate chromatographic peaks by their deviation from expected RT, including alignment-corrected deviation, for downstream ML peak classification.

## When NOT to use

- No expected RT is available for the analyte in the metabolite database, so deviation cannot be defined
- Raw data are not MRM-mode LC-MS chromatograms (e.g. full-scan DDA/DIA), where quantifier/qualifier transition logic and RT-shift cross-correlation do not apply
- You only need final reported peaks, not per-candidate RT-deviation diagnostics — use the full process_batch() reporting step instead

## Inputs

- Quantifier and qualifier MRM chromatograms parsed from mzML files with mzR (one per metabolite per sample)
- Metabolite database (metabdb.xlsx) containing expected RTs per transition
- Parameter file (update_prm.tsv) defining smoothing degree and number of candidate peaks to evaluate

## Outputs

- Per-transition, per-sample table of candidate peaks with all 20 quality scores including QS_dRT and QS_dRTs (qslog_initial.tsv)
- Sample-level RT shifts and their quadratic fit used to derive the drift-corrected QS_dRTs

## How to apply

During peak picking, smooth each metabolite's quantifier trace (smoothing degree from update_prm.tsv), detect local maxima ranked by peak height, and for each candidate compute QS_dRT, the deviation of the candidate's apex from the expected RT stored in the metabolite database (metabdb.xlsx). Before scoring, calculate RT shifts separately for each metabolite in each sample by simultaneous cross-correlation, and fit the RT shifts of all metabolites in a sample as a quadratic function of RT; QS_dRTs then reports the same deviation after correcting for this predicted RT shift relative to the reference sample. QS_dRT and QS_dRTs are two of the 20 quality scores characterizing peak shape, quantifier/qualifier agreement, and RT deviation; when no ML model is supplied, they are averaged with the other QS values to obtain the peak classification score. The RT alignment rationale is that chromatographic drift differs per metabolite per sample, so absolute deviation (QS_dRT) and drift-corrected deviation (QS_dRTs) must both be available to the peak picking and peak reporting ML models, which use the same quality scores.

## Related tools

- **automRm** (R package implementing RT-shift estimation by cross-correlation, quadratic RT-shift fitting, and QS_dRT/QS_dRTs computation within its peak-picking pipeline) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **mzR** (Parses mzML files to extract the quantifier and qualifier MRM chromatograms on which RT deviation is measured)
- **openxlsx** (Reads the metabolite database (metabdb.xlsx) supplying expected RT values)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')
```

## Evaluation signals

- qslog_initial.tsv contains QS_dRT and QS_dRTs fields for every candidate peak of every metabolite in every sample
- QS_dRT is near zero for candidates whose apex coincides with the expected RT in metabdb.xlsx, and grows with apex displacement
- QS_dRTs tracks QS_dRT in samples whose fitted quadratic RT shift is ~0, and differs from it proportionally to the sample's predicted shift relative to the reference sample
- Classification scores are derivable as averages of the 20 QS values (the no-ML-model fallback), including the RT-deviation scores
- Fitted RT shifts of all metabolites in a sample follow a quadratic function of RT as stated in the methods

## Limitations

- RT deviation scoring presumes a reference sample exists from which a predicted RT shift can be estimated by cross-correlation; without reliable alignment the drift-corrected QS_dRTs is unreliable
- For optimal performance automRm requires at least one qualifier in addition to the quantifier; scoring on quantifier traces alone degrades quality estimation
- automRm results reach manual-review quality only with sufficient training on relevant raw data; very new LC methods may show systematic RT deviations until retrained

## Evidence

- [methods] Peak picking with quality scores: "local maxima are detected in a smoothed version of the quantifier trace and ranked by peak height"
- [methods] RT alignment via cross-correlation: "calculate RT shifts separately for each metabolite in each sample by the simultaneous cross-correlation"
- [methods] Filter steps (candidate fine-grained sub-tasks): RT shift fitting as quadratic function: "RT shifts of all metabolites in a sample are fitted as a quadratic function of RT"
- [methods] Peak reporting with ML model: "the peak reporting ML model uses the same quality scores that were previously used by the peak picking ML model"
- [abstract] ML-based peak detection rationale: "Using machine learning (ML) for detection of chromatographic peaks and quality control of reported results enables the automatic recognition of complex patterns in raw data."

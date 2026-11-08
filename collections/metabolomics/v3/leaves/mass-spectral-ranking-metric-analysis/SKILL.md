---
name: mass-spectral-ranking-metric-analysis
description: Use when ranked candidate lists (e.g., the top-100 ranked compounds PS2MS
  yields per analyte) exist for the same test substances under two or more similarity
  metrics, and you must quantify which metric puts the true compound at rank 1.
when_to_use_negative:
- The analyte mass spectrum already matches an established reference spectrum library
  (NIST, Wiley, or SWGDRUG), where conventional screening is sufficient and no ranking-metric
  comparison is needed
- The chemical structure of the NPS is already known, since in silico spectral prediction
  such as NEIMS is limited to known structures and no metric re-ranking is required
- The task is to build or filter the synthetic NPS database (enumeration and rule-based
  filtering of derivatives) rather than evaluate match metrics
tools:
- name: PS2MS
  role: Produces the ranked candidate lists (top-100 similar compounds ranked by integrated
    similarity score) whose top-1 accuracy is compared across metrics
  repo: https://github.com/jhhung/PS2MS
- name: NEIMS
  role: Predicts the mass spectra of compounds in the synthetic database, providing
    the spectral component of the cosine similarity being evaluated
  repo: https://github.com/jhhung/PS2MS
- name: DeepEI
  role: Predicts fingerprints of the unknown analyte and computes fingerprints for
    synthetic-database compounds, supplying the Jaccard fingerprint component of the
    integrated metric
  repo: https://github.com/jhhung/PS2MS
- name: python
  role: Runtime environment (3.6.9 or higher) for loading ranked outputs, computing
    top-1 proportions, and writing the comparison CSV
  repo: https://www.python.org/downloads/
- name: rdkit
  role: Chemistry toolkit used in the pipeline for handling fingerprints during matching
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:720f4e23518610b3ca66abc6466819c5a0bfe33f1641bd35b8283351f136bc8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - NEIMS
  - PS2MS
  - python
  - rdkit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-spectral-ranking-metric-analysis

## Summary

A benchmarking skill for comparing similarity metrics used to rank candidate compound identities in mass-spectral matching, as in PS2MS where an integrated SMSF (mass spectrum + molecular fingerprint) metric is evaluated against spectrum-only matching on GC/EI-MS data.

## When to use

Use when ranked candidate lists (e.g., the top-100 ranked compounds PS2MS yields per analyte) exist for the same test substances under two or more similarity metrics, and you must quantify which metric puts the true compound at rank 1.

## When NOT to use

- The analyte mass spectrum already matches an established reference spectrum library (NIST, Wiley, or SWGDRUG), where conventional screening is sufficient and no ranking-metric comparison is needed
- The chemical structure of the NPS is already known, since in silico spectral prediction such as NEIMS is limited to known structures and no metric re-ranking is required
- The task is to build or filter the synthetic NPS database (enumeration and rule-based filtering of derivatives) rather than evaluate match metrics

## Inputs

- Ranked-match output files for SWGDRUG test substances, one per similarity metric (cosine-only and cosine+Jaccard)
- SWGDRUG ground-truth compound identities for the test substances
- Ranked candidate lists (hundred most similar compounds per analyte, ranked by integrated SMSF similarity score)

## Outputs

- top1_metric_comparison.csv: side-by-side table with one row per metric and columns for metric name and top-1 correct-proportion
- Per-metric top-1 accuracy proportions

## How to apply

Load the ranked-match output files produced for the SWGDRUG test substances, one ranking from the cosine-similarity-only metric and one from the integrated cosine+Jaccard metric. For each test substance, match its known SWGDRUG ground-truth identity and flag a correct prediction when the true compound appears at rank 1 of the ranked candidate list. Compute the proportion of correct top-1 predictions separately per metric, then assemble a side-by-side comparison table (one row per metric, columns for metric name and top-1 proportion) saved as top1_metric_comparison.csv. The rationale is that adding Jaccard fingerprint similarity to the spectral cosine metric reorders candidates so the true identity ranks higher; in the source work this raised top-1 accuracy from below 40% to above 50% on SWGDRUG test substances.

## Related tools

- **PS2MS** (Produces the ranked candidate lists (top-100 similar compounds ranked by integrated similarity score) whose top-1 accuracy is compared across metrics) — https://github.com/jhhung/PS2MS
- **NEIMS** (Predicts the mass spectra of compounds in the synthetic database, providing the spectral component of the cosine similarity being evaluated) — https://github.com/jhhung/PS2MS
- **DeepEI** (Predicts fingerprints of the unknown analyte and computes fingerprints for synthetic-database compounds, supplying the Jaccard fingerprint component of the integrated metric) — https://github.com/jhhung/PS2MS
- **python** (Runtime environment (3.6.9 or higher) for loading ranked outputs, computing top-1 proportions, and writing the comparison CSV) — https://www.python.org/downloads/
- **rdkit** (Chemistry toolkit used in the pipeline for handling fingerprints during matching) — https://www.rdkit.org/docs/Install.html

## Examples

```
python -c "import pandas as pd; pd.DataFrame({'metric':['cosine','cosine+jaccard'],'top1_proportion':[p_cos,p_jac]}).to_csv('top1_metric_comparison.csv',index=False)"
```

## Evaluation signals

- Cosine-only top-1 proportion falls below 0.40 on SWGDRUG test substances
- Jaccard-integrated top-1 proportion exceeds 0.50 on SWGDRUG test substances
- The comparison table contains exactly one row per metric with a valid proportion in [0,1] per row
- Correct-prediction flags are consistent with the SWGDRUG ground truth appearing at rank 1 of the candidate list for each test substance
- Sanity check: using SMSF as the metric, over 85% of SWGDRUG test substances rank within the top 20 predictions

## Limitations

- Reported improvement is a range ('below 40%' to 'more than 50%'), not an exact threshold, so borderline results may be ambiguous
- Metrics were validated on GC/EI-MS spectra at 70 eV electron energy against NIST/Wiley/SWGDRUG libraries; transferability to other acquisition settings is not established
- Additional postprocessing may be needed beyond the ranking to determine precise identities of the analyte

## Evidence

- [methods] The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity.: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [readme] The system will compare the spectrum and chemical fingerprint between compounds and generate a list of the hundred most similar compounds which are ranked by similarity score.: "The system will compare the spectrum and chemical fingerprint between compounds and generate a list of the hundred most similar compounds which are ranked by similarity score."
- [methods] over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"

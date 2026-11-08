---
name: top-k-retrieval-metric-comparison
description: Use when two ranked-match output files over the same labeled test substances
  exist and you need to decide whether integrating a second similarity component (e.g.,
  Jaccard fingerprint similarity with cosine spectral similarity) moves correct hits
  into the top ranks.
when_to_use_negative:
- The analyte spectra already match established reference spectra in conventional
  library screening (NIST/Wiley/SWGDRUG), so no re-ranking of synthetic NPS candidates
  is needed.
- The chemical structure of the novel substance is already known, since in silico
  prediction (e.g., NEIMS) can be applied directly and metric comparison of anonymous
  retrieval is moot.
- Rankings come from different candidate sets or different test substances per metric,
  making the top-1 proportions not directly comparable.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: PS2MS
  role: Produces the ranked candidate lists (top 100 most similar compounds ranked
    by integrated SMSF similarity score) that are scored at top-1 and top-20 against
    ground truth
  repo: https://github.com/jhhung/PS2MS
- name: DeepEI
  role: Predicts the molecular fingerprint of the unknown analyte that enables the
    Jaccard fingerprint similarity component being evaluated
  repo: https://github.com/jhhung/PS2MS
- name: NEIMS
  role: Predicts the EI mass spectra of synthetic database compounds used as the cosine-similarity
    reference side of the comparison
  repo: https://github.com/jhhung/PS2MS
- name: python
  role: Scripted computation of top-1 proportions and assembly of the comparison CSV
- name: rdkit
  role: Provides molecular fingerprint representations used for the Jaccard similarity
    calculations
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:a612170d9a536e6e548bc2ee5aa02fea6133546553c25e4081b944fed8c66866
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# top-k-retrieval-metric-comparison

## Summary

Quantifies whether adding a fingerprint-based similarity (Jaccard) to a spectral similarity metric (cosine/SMSF) improves top-k retrieval accuracy, by recomputing the proportion of correct compounds ranked at position 1 (and within top 20) for a labeled test set such as SWGDRUG substances.

## When to use

Use when two ranked-match output files over the same labeled test substances exist and you need to decide whether integrating a second similarity component (e.g., Jaccard fingerprint similarity with cosine spectral similarity) moves correct hits into the top ranks.

## When NOT to use

- The analyte spectra already match established reference spectra in conventional library screening (NIST/Wiley/SWGDRUG), so no re-ranking of synthetic NPS candidates is needed.
- The chemical structure of the novel substance is already known, since in silico prediction (e.g., NEIMS) can be applied directly and metric comparison of anonymous retrieval is moot.
- Rankings come from different candidate sets or different test substances per metric, making the top-1 proportions not directly comparable.

## Inputs

- Ranked-match output files for SWGDRUG test substances, one ranking per similarity metric (cosine-only and cosine+Jaccard)
- Ground-truth compound identities for each SWGDRUG test substance

## Outputs

- top1_metric_comparison.csv — side-by-side table with one row per metric and columns for metric name and top-1 correct-prediction proportion
- Optional supplementary top-20 (top-k) hit-rate proportions per metric per dataset (NIST, SWGDRUG)

## How to apply

Load the ranked candidate lists produced by the matching step (here, PS2MS drug-detection output) for each test substance under both metrics — cosine-spectrum-only and the integrated cosine+Jaccard score. Determine each substance's known ground-truth identity from the SWGDRUG library and flag a hit when the true compound appears at rank 1 in the ranked list. Compute the proportion of correct top-1 predictions per metric and assemble a side-by-side table with one row per metric (metric name, top-1 proportion). As supporting evidence for retrieval quality, also report the proportion of test substances ranked within the top 20 predictions (the paper reports >90% for NIST and >85% for SWGDRUG with SMSF). The expected improvement signal in this study is top-1 accuracy rising from below 40% (cosine-only) to above 50% after incorporating Jaccard similarity, so save the comparison as top1_metric_comparison.csv and validate against those thresholds.

## Related tools

- **PS2MS** (Produces the ranked candidate lists (top 100 most similar compounds ranked by integrated SMSF similarity score) that are scored at top-1 and top-20 against ground truth) — https://github.com/jhhung/PS2MS
- **DeepEI** (Predicts the molecular fingerprint of the unknown analyte that enables the Jaccard fingerprint similarity component being evaluated) — https://github.com/jhhung/PS2MS
- **NEIMS** (Predicts the EI mass spectra of synthetic database compounds used as the cosine-similarity reference side of the comparison) — https://github.com/jhhung/PS2MS
- **python** (Scripted computation of top-1 proportions and assembly of the comparison CSV)
- **rdkit** (Provides molecular fingerprint representations used for the Jaccard similarity calculations) — https://www.rdkit.org/docs/Install.html

## Examples

```
import pandas as pd; df = pd.read_csv('ranked_matches.csv'); top1 = df.groupby('metric').apply(lambda g: (g['rank']==1).mean()); top1.to_csv('top1_metric_comparison.csv')
```

## Evaluation signals

- Cosine-only top-1 correct-prediction proportion falls below 0.40 on SWGDRUG test substances
- Jaccard-integrated top-1 proportion exceeds 0.50, consistent with the reported improvement
- The comparison CSV contains exactly one row per metric with a metric-name column and a top-1-proportion column
- With SMSF as the metric, over 90% (NIST) and over 85% (SWGDRUG) of test substances rank within the top 20 predictions
- Each test substance is scored against its known SWGDRUG ground-truth identity, and only rank-1 occurrences count as correct

## Limitations

- Top-1 accuracy was reported only as proportions below 40% and above 50%, not exact counts or confidence intervals; exact thresholds may vary with the test split.
- Improvement was demonstrated for the SWGDRUG/NIST EI-MS test settings with cathinone-like enumeration; generalization to other libraries or ionization methods is not established.
- The integrated metric depends on DeepEI-predicted fingerprints, so fingerprint prediction error propagates into the Jaccard component.

## Evidence

- [methods] Top-1 accuracy improves from cosine-only to cosine+Jaccard: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [methods] Top-20 retrieval rates for NIST and SWGDRUG: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [intro] Ranking mechanism being evaluated: "The integrated similarity metric between the analyte and enumerated compounds in the NPS database based on the mass spectrum and fingerprint are calculated, summarized, and ranked."
- [readme] Ranking output format of the drug-detection step: "generate a list of the hundred most similar compounds which are ranked by similarity score."
- [readme] Role of DeepEI fingerprints enabling the Jaccard component: "PS<sup>2</sup>MS employs DeepEI to predict the fingerprint of the unknown analyte."

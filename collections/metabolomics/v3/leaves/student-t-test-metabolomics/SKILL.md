---
name: student-t-test-metabolomics
description: Use when an LC/MS untargeted metabolomics feature table (m/z, retention
  time, intensity per sample) must be reduced to a significant feature list by univariate
  statistics before mummichog pathway analysis.
when_to_use_negative:
- Input is raw spectral data or MS/MS fragmentation rather than an aligned, quantified
  feature table — mummichog and this selection step require the feature table upstream
- The analysis goal is metabolite identification from fragmentation patterns, which
  the paper explicitly says requires tandem mass spectrometry instead of mummichog
- Multivariate in vivo study design where the paper notes validation is particularly
  challenging and a simple two-group t-test is inadequate
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: mummichog
  role: Downstream consumer of the significant feature list F_s and the full reference
    feature list for pathway/network activity prediction
  repo: http://atcg.googlecode.com
- name: LC/MS metabolomics
  role: Source of the 7,995 spectral features (m/z, retention time, intensity) that
    the t-tests are computed over
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:6f78207e8b1f79d9c08493f2bc30129687b297624d6fd0ec3ef28c8c7eb2b9c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LC/MS metabolomics
  - mummichog
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# student-t-test-metabolomics

## Summary

Per-feature two-sample Student's t-test over an LC/MS metabolomics feature table to select a significant feature list (F_s) feeding downstream pathway/network analysis such as mummichog. In the moDC YF17D activation experiment this reduced 7,995 QC-passing spectral features to exactly 601 significant features.

## When to use

Use when an LC/MS untargeted metabolomics feature table (m/z, retention time, intensity per sample) must be reduced to a significant feature list by univariate statistics before mummichog pathway analysis.

## When NOT to use

- Input is raw spectral data or MS/MS fragmentation rather than an aligned, quantified feature table — mummichog and this selection step require the feature table upstream
- The analysis goal is metabolite identification from fragmentation patterns, which the paper explicitly says requires tandem mass spectrometry instead of mummichog
- Multivariate in vivo study design where the paper notes validation is particularly challenging and a simple two-group t-test is inadequate

## Inputs

- LC/MS metabolomics feature table (Dataset S1) with 7,995 spectral features: m/z, retention time, intensity values across moDC YF17D-infected and control samples
- Sample-group definitions (YF17D-infected 6 hr, mock 6 hr, 0 hr baseline)
- P-value cutoff used in the moDC analysis

## Outputs

- significant_features.tsv listing F_s: feature identifiers, m/z, retention time, and t-test p-values (expected row count: 601)
- Reference list of all 7,995 detected features for downstream mummichog input

## How to apply

Load the feature table (e.g. Dataset S1 of the moDC experiment) containing 7,995 QC-passed spectral features with m/z, retention time and intensity across infected and control samples. For each feature, run two two-sample Student's t-tests: (1) YF17D-infected 6 hr vs time-matched mock 6 hr, and (2) YF17D-infected 6 hr vs the 0 hr baseline. Select features as significant using the paper's p-value cutoff, which the article states was chosen 'by a cutoff value' for the moDC data. Validate the result by confirming the significant list contains exactly 601 features, matching the published count. Write F_s as a table of feature identifiers with m/z, retention time and t-test p-values, since mummichog consumes exactly this pair of significant and reference m/z feature lists.

## Related tools

- **mummichog** (Downstream consumer of the significant feature list F_s and the full reference feature list for pathway/network activity prediction) — http://atcg.googlecode.com
- **LC/MS metabolomics** (Source of the 7,995 spectral features (m/z, retention time, intensity) that the t-tests are computed over)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(infected_6hr, mock_6hr, axis=1)
```

## Evaluation signals

- Row-count invariant: the output significant feature list F_s must contain exactly 601 features (row_count_equals 601)
- Every feature in F_s carries a two-sided Student's t-test p-value computed from both comparisons (infected 6 hr vs mock 6 hr and infected 6 hr vs 0 hr baseline)
- Output schema check: significant_features.tsv contains feature identifier, m/z, retention time, and p-value columns
- Reference list check: the full feature table retains all 7,995 QC-passing features so mummichog receives both the significant and reference m/z lists
- P-values of features written to F_s must satisfy the moDC cutoff used in the paper

## Limitations

- Quantification and feature selection upstream of mummichog 'can still be problematic' per the authors
- The current mummichog version does not incorporate a statistical matrix from the feature selection step, so ranking information from the t-test is not carried into pathway analysis
- A single m/z feature may match several metabolites in the same pathway, and database annotation limits cause a de facto filtering of features during mummichog analysis

## Evidence

- [results] Among them, 601 features were significantly different between the infected samples: "yielded 7,995 spectral features (denoted as ) after quality control. Among them, 601 features were significantly different between the infected samples"
- [discussion] In the moDC data, we chose  by a cutoff value .: "In the moDC data, we chose  by a cutoff value ."
- [results] mummichog requires two lists of m/z features, the significant list F_s and the reference list of all features: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [readme] User supplied features with m/z, rtime, p-value from a statistical test.: "User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID."

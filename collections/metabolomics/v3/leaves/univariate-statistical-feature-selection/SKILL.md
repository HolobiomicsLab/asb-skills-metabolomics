---
name: univariate-statistical-feature-selection
description: Use when you have a QC-passed LC/MS feature table (m/z, retention time,
  intensities) with infected vs. control/baseline sample groups and need to select
  significantly altered features before pathway analysis.
when_to_use_negative:
- You already have a statistically tested feature table with p-values that mummichog
  can consume directly (re-running selection would duplicate work)
- The analysis goal is metabolite identification from fragmentation spectra, which
  this skill does not address
- The data require multivariate or in vivo study design analysis, where the paper
  notes validation is particularly challenging and this simple univariate cutoff approach
  is insufficient
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mummichog
  role: 'Downstream consumer: takes the significant feature list F_s (selected by
    univariate statistics) together with the reference list of all detected m/z features
    for pathway/network activity prediction'
  repo: http://atcg.googlecode.com
- name: LC/MS
  role: Data-generating platform whose QC-passed feature table (7,995 spectral features
    in the moDC experiment) is the input to the t-test selection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:0059acf4e02921e8cbbbd2a915d420d94ac5779300e9d948cd7f834331620c87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LC/MS
  - mummichog
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# univariate-statistical-feature-selection

## Summary

Per-feature univariate statistical testing (Student's t-test) on an LC/MS metabolomics feature table to select the significant feature list F_s (e.g., 601 of 7,995 features in the moDC YF17D activation experiment) for downstream mummichog pathway/network analysis.

## When to use

Use when you have a QC-passed LC/MS feature table (m/z, retention time, intensities) with infected vs. control/baseline sample groups and need to select significantly altered features before pathway analysis.

## When NOT to use

- You already have a statistically tested feature table with p-values that mummichog can consume directly (re-running selection would duplicate work)
- The analysis goal is metabolite identification from fragmentation spectra, which this skill does not address
- The data require multivariate or in vivo study design analysis, where the paper notes validation is particularly challenging and this simple univariate cutoff approach is insufficient

## Inputs

- LC/MS metabolomics feature table (Dataset S1) with 7,995 QC-passed spectral features: m/z, retention time, and intensities across moDC YF17D-infected, mock 6 hr, and 0 hr baseline samples
- Sample-group assignment (infected 6 hr, mock 6 hr, 0 hr baseline)

## Outputs

- significant_features.tsv with feature identifiers, m/z, retention time, and t-test p-values (exactly 601 rows for the moDC experiment)
- Significant feature list F_s plus full reference list of all detected features, structured for mummichog input

## How to apply

Load the Dataset S1 feature table containing 7,995 spectral features with m/z, retention time, and intensity values across YF17D-infected and control moDC samples. For each of the 7,995 features, perform a two-sample Student's t-test comparing YF17D-infected 6 hr vs mock 6 hr intensities, and a second t-test comparing YF17D-infected 6 hr vs the 0 hr baseline. Select features as significant by the cutoff value used in the moDC analysis (the paper states features were chosen by a cutoff value) to yield the significant list F_s. Write the result with feature identifiers, m/z, retention time, and t-test p-values; note that mummichog expects 'user supplied features with m/z, rtime, p-value from a statistical test'.

## Related tools

- **mummichog** (Downstream consumer: takes the significant feature list F_s (selected by univariate statistics) together with the reference list of all detected m/z features for pathway/network activity prediction) — http://atcg.googlecode.com
- **LC/MS** (Data-generating platform whose QC-passed feature table (7,995 spectral features in the moDC experiment) is the input to the t-test selection)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(infected_6hr_intensities, mock_6hr_intensities, axis=1)
```

## Evaluation signals

- Row-count invariant: the output significant feature list F_s contains exactly 601 features, matching the published count for the moDC YF17D dataset
- Completeness check: both t-tests were run for every one of the 7,995 features (6 hr infected vs mock 6 hr, and 6 hr infected vs 0 hr baseline), not just one comparison
- Schema check: significant_features.tsv includes feature identifiers, m/z, retention time, and t-test p-values in the format mummichog expects (features with m/z, rtime, p-value)
- Consistency check: the reference list retains all features detected in the MS experiment, since mummichog requires both the significant list and the full reference list
- Feature count before selection equals 7,995, confirming QC-passed input was used without additional de facto filtering

## Limitations

- Quantification and feature selection, the steps upstream of mummichog, can still be problematic, so the significance cutoff choice directly shapes downstream pathway/network results
- A de facto filtering occurs later when data are run on mummichog due to limited metabolite database annotations, so F_s size does not equal metabolites analyzed
- The current mummichog version does not incorporate a statistical matrix from the feature selection step, so only binary significant/reference membership is carried forward

## Evidence

- [results] LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control.: "LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control."
- [results] Among them, 601 features were significantly different between the infected samples: "Among them, 601 features were significantly different between the infected samples"
- [results] mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment).: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics)"
- [discussion] Quantification and feature selection, the steps upstream of mummichog, can still be problematic.: "Quantification and feature selection, the steps upstream of mummichog, can still be problematic."
- [readme] User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID.: "User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID."

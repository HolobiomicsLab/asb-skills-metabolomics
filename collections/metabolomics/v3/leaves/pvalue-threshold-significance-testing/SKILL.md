---
name: pvalue-threshold-significance-testing
description: Use when an untargeted LC/MS metabolomics feature table (e.g. 7,995 spectral
  features with m/z, retention time and intensities across infected vs. mock/baseline
  samples) must be reduced to a significant feature list F_s for downstream network-activity
  analysis.
when_to_use_negative:
- The features are already annotated metabolites with identities — this skill selects
  univariate significance among raw spectral features, not interpret annotated metabolite
  tables.
- Only a significant feature list is available with no reference list of all detected
  features — mummichog requires both F_s and F_all for pathway and module activity
  estimation.
- The experiment requires multivariate modeling of in vivo data where 'validation
  is particularly challenging' per the discussion — univariate t-test cutoffs alone
  may not suffice.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mummichog
  role: Downstream consumer of the significant list F_s and reference list F_all;
    it performs pathway enrichment (Fisher's exact test) and network module activity
    prediction on the feature lists produced by this skill
  repo: http://atcg.googlecode.com
- name: LC/MS
  role: Instrumentation platform producing the 7,995-feature spectral table that is
    the input to the t-test-based feature selection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:5644adc05e92d69bb3ab35cda499dde4d38a8431a4e69b6e4b7eaba7a26cf272
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LC/MS
  - mummichog
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pvalue-threshold-significance-testing

## Summary

Selects significantly altered spectral features from a metabolomics feature table by applying Student's t-tests at a p-value cutoff, producing the significant feature list F_s that feeds mummichog's pathway and network activity prediction.

## When to use

Use when an untargeted LC/MS metabolomics feature table (e.g. 7,995 spectral features with m/z, retention time and intensities across infected vs. mock/baseline samples) must be reduced to a significant feature list F_s for downstream network-activity analysis.

## When NOT to use

- The features are already annotated metabolites with identities — this skill selects univariate significance among raw spectral features, not interpret annotated metabolite tables.
- Only a significant feature list is available with no reference list of all detected features — mummichog requires both F_s and F_all for pathway and module activity estimation.
- The experiment requires multivariate modeling of in vivo data where 'validation is particularly challenging' per the discussion — univariate t-test cutoffs alone may not suffice.

## Inputs

- LC/MS metabolomics feature table (e.g. Dataset S1) with 7,995 spectral features carrying m/z, retention time and intensity values across YF17D-infected, mock-control and 0 hr baseline moDC samples
- Grouping/condition labels for samples (infected vs. mock vs. baseline) to define the t-test comparisons

## Outputs

- Significant feature list F_s (e.g. significant_features.tsv) with feature identifiers, m/z, retention time and t-test p-values; exactly 601 features in the moDC case
- Reference feature list F_all of all detected features (7,995 in the moDC case) retained for mummichog's pathway and module analysis

## How to apply

Load the QC-passed LC/MS feature table (m/z, retention time, intensity per sample) and perform a two-sample Student's t-test for each feature, comparing, as in the moDC YF17D activation experiment, YF17D-infected 6 hr intensities against mock 6 hr controls and against the 0 hr baseline. Apply the chosen p-value cutoff to the t-test results to select features into the significant list F_s; in the moDC data this cutoff reduced 7,995 features to exactly 601. The rationale, per the paper, is that mummichog requires two m/z feature lists — the significant list F_s and the reference list F_all of all detected features — so feature selection by univariate statistics is the standard entry point before pathway (Fisher's exact test) and module analysis. Quantification and feature selection upstream of mummichog 'can still be problematic', so the choice of cutoff is a documented decision point: the paper notes 'In the moDC data, we chose [F_s] by a cutoff value [p]'. Write the selected features with identifiers, m/z, retention time and t-test p-values to a file such as significant_features.tsv, and retain the full list as the reference input.

## Related tools

- **mummichog** (Downstream consumer of the significant list F_s and reference list F_all; it performs pathway enrichment (Fisher's exact test) and network module activity prediction on the feature lists produced by this skill) — http://atcg.googlecode.com
- **LC/MS** (Instrumentation platform producing the 7,995-feature spectral table that is the input to the t-test-based feature selection)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Row count of the significant feature list F_s equals exactly 601 when reproducing the moDC YF17D dataset (row_count_equals 601).
- Every selected feature carries a finite m/z, retention time and t-test p-value consistent with the input table schema (feature_id, m/z, rtime, p-value columns).
- The reference list F_all retains all 7,995 QC-passed features, i.e. no features are dropped before mummichog sees them.
- Both output lists are m/z-based and in the form mummichog requires: 'mummichog requires two lists of m/z features, the significant list [F_s] and the reference list [F_all]'.
- Comparing the two t-test comparisons (infected vs. mock 6 hr and infected vs. 0 hr baseline) confirms the chosen p-value cutoff reproduces the 601-feature count rather than an arbitrary threshold.

## Limitations

- The p-value cutoff for choosing F_s is a judgment call: 'In the moDC data, we chose [F_s] by a cutoff value [p]' — different cutoffs change the size of F_s and downstream module activity.
- Quantification and feature selection upstream of mummichog 'can still be problematic', so t-test significance may be sensitive to upstream peak extraction and alignment.
- This skill does not perform metabolite identification; mummichog is explicitly not designed to replace tandem mass spectrometry for that purpose, and a single m/z feature may match several metabolites, inflating apparent pathway overlap.

## Evidence

- [results] mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment).: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [discussion] In the moDC data, we chose  by a cutoff value .: "In the moDC data, we chose  by a cutoff value ."
- [discussion] Quantification and feature selection, the steps upstream of mummichog, can still be problematic.: "Quantification and feature selection, the steps upstream of mummichog, can still be problematic."
- [results] The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control.: "The cell extracts from our activation experiment were analyzed by LC/MS metabolomics, and yielded 7,995 spectral features (denoted as ) after quality control."
- [readme] User supplied features with m/z, rtime, p-value from a statistical test.: "User supplied features with m/z, rtime, p-value from a statistical test."

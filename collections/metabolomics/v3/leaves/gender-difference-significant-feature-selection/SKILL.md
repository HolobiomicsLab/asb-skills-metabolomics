---
name: gender-difference-significant-feature-selection
description: Use when an untargeted LC/MS feature table (m/z, retention time, intensity
  per sample) has two biological groups, e.g. male vs female urine, and a significant
  feature list plus full reference list must be derived before mummichog pathway/network
  analysis.
when_to_use_negative:
- The dataset is a targeted assay or already contains confident MS/MS-based metabolite
  identifications, so univariate feature selection for tentative network prediction
  is unnecessary
- The experiment has no defined two-group contrast (e.g. multivariate in vivo design),
  which the paper notes makes validation particularly challenging
- Features have not been quality-controlled/quantified — the paper warns 'Quantification
  and feature selection, the steps upstream of mummichog, can still be problematic'
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mummichog
  role: Downstream consumer of the significant and reference m/z feature lists; its
    two-list input requirement defines the output format of this feature-selection
    step
  repo: http://atcg.googlecode.com
- name: Azimuth DB
  role: Chemical/metabolic model database backing mummichog's integrated human metabolic
    model used after feature selection
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:e148c9e63efc8eeee537ee92367953be81de5c5bd7daa74451673808fa349113
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - mummichog
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gender-difference-significant-feature-selection

## Summary

Selection of significant and reference m/z feature lists from an untargeted urinary LC/MS feature table for a male-vs-female differential analysis, as the upstream input required by mummichog's network-activity prediction.

## When to use

Use when an untargeted LC/MS feature table (m/z, retention time, intensity per sample) has two biological groups, e.g. male vs female urine, and a significant feature list plus full reference list must be derived before mummichog pathway/network analysis.

## When NOT to use

- The dataset is a targeted assay or already contains confident MS/MS-based metabolite identifications, so univariate feature selection for tentative network prediction is unnecessary
- The experiment has no defined two-group contrast (e.g. multivariate in vivo design), which the paper notes makes validation particularly challenging
- Features have not been quality-controlled/quantified — the paper warns 'Quantification and feature selection, the steps upstream of mummichog, can still be problematic'

## Inputs

- Untargeted LC/MS feature table (m/z, retention time, per-sample intensities), e.g. positive-ion table from MetaboLights study 439020
- Sample metadata with group labels (male vs female)
- Reference annotation of known metabolites for validation (e.g. original authors' annotations, testosterone glucuronide)

## Outputs

- Reference feature list (all features detected; 11,086 after intensity >100,000 in >=5 samples filtering)
- Significant feature list (524 gender-difference features with m/z, rtime, p-value and fold-change direction)
- Directionality sanity check (testosterone glucuronide low in females, high in males)

## How to apply

Load the feature table (e.g. the positive-ion table from MetaboLights study 439020) into a pandas DataFrame and first apply a quality/intensity filter — features with intensity >100,000 in at least 5 samples, which retains 11,086 features in the human urine study — to form the reference list. Run a univariate two-group statistical test (e.g. t-test on male vs female intensities) over the reference features and select features passing the significance cutoff to form the significant list (524 gender-difference features in the urine study). Verify directionality of a known biomarker such as testosterone glucuronide (low in females, high in males) as a sanity check on group labeling. mummichog then consumes both lists: 'mummichog requires two lists of m/z features, the significant list (e.g. selected by univariate statistics) and the reference list (all features detected in the MS experiment)'. Note that feature selection and quantification upstream of mummichog are the most fragile steps, so cutoffs should be documented and reproducible.

## Related tools

- **mummichog** (Downstream consumer of the significant and reference m/z feature lists; its two-list input requirement defines the output format of this feature-selection step) — http://atcg.googlecode.com
- **Azimuth DB** (Chemical/metabolic model database backing mummichog's integrated human metabolic model used after feature selection)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- After the intensity filter (intensity >100,000 in at least 5 samples), the reference list contains the expected 11,086 features
- The significant list contains the expected ~524 gender-difference features for the urinary dataset
- Known biomarker directionality holds: testosterone glucuronide is low in females and high in males
- Downstream mummichog run reproduces a 45-metabolite activity network with >=97% (31/32) agreement against the original authors' annotations

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating downstream overlap numbers
- Quantification and feature selection upstream of mummichog can still be problematic; cutoff choices strongly affect the significant list
- Features from high-resolution profiling far exceed metabolite database annotations, so a de facto filtering occurs when data are run on mummichog

## Evidence

- [results] mummichog requires two lists of m/z features: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list"
- [discussion] upstream fragility: "Quantification and feature selection, the steps upstream of mummichog, can still be problematic."
- [readme] input format expected by mummichog: "User supplied features with m/z, rtime, p-value from a statistical test."
- [supplementary] urine validation context: "Validation of mummichog on the human urine data set (Roux et al 2012), using metabolic model from BioCyc database."

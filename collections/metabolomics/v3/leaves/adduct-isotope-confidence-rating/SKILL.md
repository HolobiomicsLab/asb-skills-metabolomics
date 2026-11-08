---
name: adduct-isotope-confidence-rating
description: 'Use when interpreting mummichog metabolite predictions from an untargeted
  LC/MS feature table: predicted compounds must be rated by the presence of expected
  ion forms (M+H[1+], M−H[−], adducts, C13 isotopes) before counting or reporting
  them.'
when_to_use_negative:
- Metabolite identities are already confirmed by tandem mass spectrometry fragmentation
  or coelution with isotopically labeled references — mummichog is not designed to
  replace MS/MS identification
- The feature table lacks isotope/adduct partner features (e.g. no co-occurring M+H[1+]
  and M(C13)+H[1+] forms), so the rating has no evidence to score
- The compound formulas in the model are salt formats rather than neutral formulas,
  breaking adduct mass calculation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mummichog
  role: Core pipeline that collects predicted metabolites, looks up their isotopic
    derivatives and adducts, and applies the confidence rating system to filter qualified
    metabolites; also exposes the option to enforce M+H[+] or M−H[−] presence
  repo: https://github.com/shuzhao-li/mummichog
- name: JMS
  role: Provides annotation data structures (empirical compounds with MS1_pseudo_Spectra
    carrying ion_relation fields such as 'M+H[1+]') used to structure annotation feeding
    the rating step
  repo: https://github.com/shuzhao-li-lab/JMS
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:72b4eb168f22fbc7f44b2b5dcd7e7ac891f170cf289a3e080798c446379bc1da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/mummichog
  tools:
  - JMS
  - mummichog
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# adduct-isotope-confidence-rating

## Summary

A confidence rating scheme in mummichog that filters predicted metabolites by checking co-occurrence of their isotopic derivatives and adducts (e.g. M+H[1+] together with M(C13)+H[1+]) among matched m/z features, so that only qualified metabolite predictions feed the activity network.

## When to use

Use when interpreting mummichog metabolite predictions from an untargeted LC/MS feature table: predicted compounds must be rated by the presence of expected ion forms (M+H[1+], M−H[−], adducts, C13 isotopes) before counting or reporting them.

## When NOT to use

- Metabolite identities are already confirmed by tandem mass spectrometry fragmentation or coelution with isotopically labeled references — mummichog is not designed to replace MS/MS identification
- The feature table lacks isotope/adduct partner features (e.g. no co-occurring M+H[1+] and M(C13)+H[1+] forms), so the rating has no evidence to score
- The compound formulas in the model are salt formats rather than neutral formulas, breaking adduct mass calculation

## Inputs

- mummichog predicted metabolite list from significant pathway/network module activities
- LC/MS feature table (m/z, retention time, p-value) containing significant and reference feature lists
- Optional annotation table on the features (e.g. empCpds_with_annotations.json) with empirical compounds and ion_relation entries
- Metabolic model with compound neutral formulas (e.g. BioCyc human model, KEGG/Recon1/Edinburgh composite model)

## Outputs

- Filtered set of qualified metabolite predictions with confidence ratings
- Predicted activity network restricted to qualified metabolites and their adduct/isotope forms
- Agreement summary against reference annotations (e.g. urine_mh_agreement.csv: 41/41 = 100%)

## How to apply

After mummichog computes possibly matched metabolites from the significant m/z list and collects metabolites from significant pathway/network activities, look up each metabolite's isotopic derivatives and adducts in the feature table. Apply the confidence rating: if both the single-charged M+H[1+] form and the C13 isotope form M(C13)+H[1+] are present among the matched features, the metabolite prediction carries high confidence; metabolites lacking such qualifying ion-form evidence are filtered out before the predicted-activity network is assembled. Optionally, enforce the presence of M+H[+] (positive mode) or M−H[−] (negative mode) outright via mummichog's enforcement option, which the paper validated on the Roux et al. 2012 human urine dataset (41/41 agreement with the original annotation). When using mummichog 3 with annotation tables, neutral formulas (not salt formats) must be used to compute adducts, and JMS structures features with explicit 'ion_relation' fields such as 'M+H[1+]' and 'M+Na[1+]'.

## Related tools

- **mummichog** (Core pipeline that collects predicted metabolites, looks up their isotopic derivatives and adducts, and applies the confidence rating system to filter qualified metabolites; also exposes the option to enforce M+H[+] or M−H[−] presence) — https://github.com/shuzhao-li/mummichog
- **JMS** (Provides annotation data structures (empirical compounds with MS1_pseudo_Spectra carrying ion_relation fields such as 'M+H[1+]') used to structure annotation feeding the rating step) — https://github.com/shuzhao-li-lab/JMS

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Qualifying metabolite predictions show co-occurrence of the expected ion forms: M+H[1+] plus M(C13)+H[1+] for high-confidence calls
- With M+H[+] enforcement on the Roux et al. 2012 human urine data (BioCyc model), predicted metabolites agree with the original annotation at 41/41 (100%), i.e. 3 of 44 predictions are extra
- The moDC network of ~60 predicted metabolites (19 not in the original annotation) remains stable when the rating is applied across feature-selection p-value cutoffs in the Figure S7 range
- The filtered qualified-metabolite list excludes compounds whose only matched features lack adduct/isotope corroboration
- JSON outputs of pathway and network module analysis plus the agreement summary table are written and well-formed

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating overlap numbers
- Features from high-resolution profiling experiments far exceed current metabolite-database annotations, causing de facto filtering during mummichog analysis
- mummichog is not designed to replace tandem mass spectrometry in metabolite identification; ratings are heuristic, not structural proof

## Evidence

- [results] if both the single-charged form M+H[1+] and the form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence.: "if both the single-charged form M+H[1+] and the form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
- [results] confidence rating filters qualified metabolites: "Mummichog collects these metabolites, and look up all their isotopic derivatives and adducts in . A confidence rating system is applied to filter for qualified metabolites."
- [results] optional enforcement of M+H[+] form: "There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction."
- [readme] neutral formula used to calculate adducts: "We use neutral formula to calculate adducts in mass spec, which includes salts."
- [readme] JMS ion_relation field in empirical compound spectra: [evidence span withheld — anchor status: external_doc]

---
name: predicted-vs-annotated-metabolite-validation
description: Use when benchmarking mummichog metabolite prediction against a published,
  experimentally annotated metabolite list for the same MS dataset, e.g. the human
  urine set of Roux et al. 2012 with a BioCyc model.
when_to_use_negative:
- When no published annotation list exists for the dataset — mummichog is 'not designed
  to replace tandem mass spectrometry in metabolite identification', so this validation
  cannot stand in for MS/MS-level confirmation.
- When the input is raw spectra rather than a processed feature table — mummichog
  requires two lists of m/z features (significant list and reference list), not spectral
  peaks.
- When a negative-mode dataset is being evaluated without switching the enforcement
  to the M−H[−] option, since the M+H[+] enforcement is specific to positive mode.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mummichog
  role: Core CLI that predicts possibly matched metabolites from m/z features, performs
    pathway and network module analysis, and supplies the predicted-metabolite list
    compared against the original annotation
  repo: https://github.com/shuzhao-li/mummichog
- name: metDataModel
  role: Data models for metabolomics used by mummichog to structure the metabolic
    model and annotation inputs in this validation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:7f68e6b033f26744c5f9f5a0ae2c4391dc3f8afb77302bb5f2c119f923e5c90f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/mummichog
  tools:
  - metDataModel
  - mummichog
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# predicted-vs-annotated-metabolite-validation

## Summary

A validation skill that reruns mummichog metabolite prediction on a published urine metabolomics feature table (Roux et al. 2012) with the M+H[+] ion-form enforcement enabled, then compares the predicted metabolites against the paper's original annotation to quantify agreement (41/41, 100%).

## When to use

Use when benchmarking mummichog metabolite prediction against a published, experimentally annotated metabolite list for the same MS dataset, e.g. the human urine set of Roux et al. 2012 with a BioCyc model.

## When NOT to use

- When no published annotation list exists for the dataset — mummichog is 'not designed to replace tandem mass spectrometry in metabolite identification', so this validation cannot stand in for MS/MS-level confirmation.
- When the input is raw spectra rather than a processed feature table — mummichog requires two lists of m/z features (significant list and reference list), not spectral peaks.
- When a negative-mode dataset is being evaluated without switching the enforcement to the M−H[−] option, since the M+H[+] enforcement is specific to positive mode.

## Inputs

- Filtered human urine feature table from Roux et al. 2012 (significant m/z features plus the reference list of all detected features)
- BioCyc human metabolic model (list_of_pathways, list_of_reactions, list_of_compounds)
- Original list of annotated metabolites from the urine study
- mummichog configuration enabling M+H[+] enforcement (positive mode)

## Outputs

- Predicted-vs-annotated metabolite agreement summary (urine_mh_agreement.csv)
- JSON outputs of pathway analysis (Fisher's exact test, EASE score, Gamma-adjusted p-values)
- JSON outputs of network module analysis (subnetworks of 1-4 reactions, Newman's spectral split)
- Agreement counts (e.g. 41/41 = 100%; 3 of 44 predicted metabolites not in the original annotation)

## How to apply

Load the filtered human urine feature table from Roux et al. 2012, keeping both the significant m/z feature list and the reference list of all detected features, since mummichog requires both. Invoke the mummichog CLI (python3 -m mummichog.main) with the BioCyc human metabolic model and the option enforcing the M+H[+] form (positive mode) in metabolite prediction, which restricts predictions to metabolites whose single-charged protonated form is present. Let mummichog compute all possibly matched metabolites from the m/z features, run Fisher's exact test pathway enrichment with the EASE score against a Gamma-modeled permutation null, and run network module analysis extracting subnetworks of one to four reactions scored against the permutation null. Collect the predicted metabolites from significant activities and compare them one-to-one against the originally annotated metabolites from the urine study. The comparison is the validation: 41 of 44 predicted metabolites matched the original annotation, giving a 41/41 (100%) agreement on the matched set, saved as urine_mh_agreement.csv.

## Related tools

- **mummichog** (Core CLI that predicts possibly matched metabolites from m/z features, performs pathway and network module analysis, and supplies the predicted-metabolite list compared against the original annotation) — https://github.com/shuzhao-li/mummichog
- **metDataModel** (Data models for metabolomics used by mummichog to structure the metabolic model and annotation inputs in this validation)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- The agreement count between predicted and originally annotated metabolites must equal 41/41 (100%), matching the reported urine-dataset validation result.
- Exactly 3 of the 44 metabolites predicted by mummichog (with M+H[+] enforced) fall outside the original annotation.
- Both JSON outputs (pathway analysis and network module analysis) and the urine_mh_agreement.csv summary are produced by the run.
- The run must use the BioCyc human metabolic model and positive-mode M+H[+] enforcement; switching models or ion modes is expected to change the comparison (e.g. the alternate-model result of 19 of 60 metabolites not found in the original annotation).

## Limitations

- A single m/z feature can match multiple metabolites, 'inflating the overlap number' in module/pathway comparisons.
- Features from high-resolution profiling experiments 'by far exceed the current annotations in metabolite databases', causing de facto filtering during mummichog runs.
- Quantification and feature selection, the steps upstream of mummichog, 'can still be problematic' and affect the validation.
- Some metabolites in metabolomics databases are not present in any of the genome-scale metabolic models, capping achievable agreement.

## Evidence

- [results] There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction.: "There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction."
- [supplementary] Validation of mummichog on the human urine data set (Roux et al 2012), using metabolic model from BioCyc database.: "Validation of mummichog on the human urine data set (Roux et al 2012), using metabolic model from BioCyc database."
- [results] mummichog requires two lists of m/z features, the significant list and the reference list.: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list"
- [readme] Mummichog is a Python program for analyzing data from high throughput, untargeted metabolomics.: "Mummichog is a Python program for analyzing data from high throughput, untargeted metabolomics."
- [discussion] Mummichog is not designed to replace tandem mass spectrometry in metabolite identification.: "Mummichog is not designed to replace tandem mass spectrometry in metabolite identification."

---
name: metabolic-subnetwork-module-detection
description: 'Use when interpreting an LC/MS feature table (m/z, rtime, p-value) via
  network-level analysis: matching features to tentative metabolites and scoring metabolic
  subnetwork modules instead of relying on confirmed metabolite IDs.'
when_to_use_negative:
- Targeted metabolomics where metabolite identities are already confirmed — mummichog
  does not replace tandem mass spectrometry in metabolite identification
- Quantification or feature selection has not been done upstream; these steps remain
  problematic and mummichog expects a curated feature table
- In vivo multivariate studies where serum/urine mass balance is impossible because
  the reactions producing these metabolites occur in other tissues
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: mummichog
  role: 'Core algorithm package: m/z-to-metabolite matching, subnetwork module detection
    via Newman''s spectral split, pathway/activity network scoring'
  repo: http://atcg.googlecode.com
- name: metDataModel
  role: Data models for metabolomics used by mummichog to structure annotations and
    the metabolic model input
  repo: https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- name: mass2chem
  role: Common utilities in interpreting mass spectrometry data, annotation, used
    for enumerating isotopic derivatives and adducts from neutral formulas
- name: JMS
  role: Efficient mass and empirical compound search functions; perform annotation
    on a dataset before mummichog analysis
  repo: https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:67218ed9a5810464187da24420e58efa594d8d3b9381fac7fa6927d460bee299
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - JMS
  - mass2chem
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# metabolic-subnetwork-module-detection

## Summary

mummichog's subnetwork module detection: from a list of significant m/z features, compute all possibly matched metabolites, extract subnetworks within one to four reactions, and identify structural modules with Newman's spectral split algorithm to predict network activity without metabolite identification.

## When to use

Use when interpreting an LC/MS feature table (m/z, rtime, p-value) via network-level analysis: matching features to tentative metabolites and scoring metabolic subnetwork modules instead of relying on confirmed metabolite IDs.

## When NOT to use

- Targeted metabolomics where metabolite identities are already confirmed — mummichog does not replace tandem mass spectrometry in metabolite identification
- Quantification or feature selection has not been done upstream; these steps remain problematic and mummichog expects a curated feature table
- In vivo multivariate studies where serum/urine mass balance is impossible because the reactions producing these metabolites occur in other tissues

## Inputs

- Feature table of m/z, retention time, and p-value (TSV, e.g. tests/ineurons_ttest_1127.tsv); row_number used as ID if none supplied
- Optional annotation table on the features (from authentic standards / MS/MS, structured via metDataModel, e.g. empCpds_with_annotations.json)
- Metabolic model in JSON containing list_of_pathways, list_of_reactions, list_of_compounds with neutral (non-salt) chemical formulas; defaults supplied by mummichog
- Two feature lists: the significant list selected by univariate statistics and the reference list of all detected features

## Outputs

- Result tables and figures with result.html, metabolites colored by log2 fold change with connecting EC numbers
- JSON strings from pathway analysis and network module analysis for programmatic use
- Predicted activity network of metabolites (e.g. 60 metabolites in the moDC/YF17D analysis, 19 not in original annotation)
- Significant pathway/module table with adjusted p-values

## How to apply

Load the user-supplied feature table (m/z, rtime, p-value) plus a metabolic model (list_of_pathways, list_of_reactions, list_of_compounds) with neutral compound formulas. Match all possible metabolites to each m/z feature by enumerating isotopic derivatives and adducts from neutral formulas, optionally enforcing M+H[1+] (positive) or M−H[−] (negative) ion forms. Extract subnetworks within one to four reactions of matched metabolites, then clean each graph of protruding edges that do not connect input metabolites so they do not inflate the activity score. Identify structural modules using Newman's spectral split algorithm with extensive sampling of subnetwork structures. Score module activity, estimate pathway enrichment with Fisher's exact test using EASE score and a smaller-of overlap correction, model the permutation-based null p-value distribution as Gamma, and adjust per pathway. Rationale: the collective power in metabolic networks resolves the ambiguity of a single m/z feature matching multiple metabolites, bypassing the metabolite identification bottleneck.

## Related tools

- **mummichog** (Core algorithm package: m/z-to-metabolite matching, subnetwork module detection via Newman's spectral split, pathway/activity network scoring) — http://atcg.googlecode.com
- **metDataModel** (Data models for metabolomics used by mummichog to structure annotations and the metabolic model input) — https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- **mass2chem** (Common utilities in interpreting mass spectrometry data, annotation, used for enumerating isotopic derivatives and adducts from neutral formulas)
- **JMS** (Efficient mass and empirical compound search functions; perform annotation on a dataset before mummichog analysis) — https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Predicted activity network size is in a plausible range for the input — e.g. the moDC YF17D activation data with 7,995 features yielded a network of 60 metabolites, of which 19 were not in the original annotation
- Significant pathways reported in result.html match the published moDC analysis
- Every retained graph has no protruding edges that fail to connect input metabolites (edges not contributing to the activity score must be removed)
- Module p-values are compared against a Gamma-modeled permutation null and adjusted per pathway; overlap counts use the smaller-of correction
- m/z features matched to metabolites span 1–4 reaction distances in the reference network; matches are scored with the confidence system (e.g. M+H[1+] + M(C13)+H[1+] co-presence = high confidence)

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number
- Current mummichog versions do not incorporate a statistical matrix from the feature selection step nor mass flow balance of reactions
- A number of metabolites in metabolomics databases are not present in any of the genome-scale metabolic models; features from high-resolution profiling far exceed current annotations
- Metabolic models need improvement on lipid annotation, physiological context and tissue specificity
- Quantification and feature selection upstream of mummichog can still be problematic
- Metabolite identification is still needed to fully resolve compound IDs — the skill is not a replacement for MS/MS identification

## Evidence

- [results] mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites.: "mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these"
- [results] Structural modules are identified within these subnetworks according to Newman's spectral split algorithm [43].: "Structural modules are identified within these subnetworks according to Newman's spectral split algorithm [43]."
- [results] Each graph is cleaned of protruding edges if such edges do not connect input metabolites therefore not contributing to the activity score.: "Each graph is cleaned of protruding edges if such edges do not connect input metabolites therefore not contributing to the activity"
- [results] if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence.: "if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high"
- [supplementary] Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation.: "Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation."
- [readme] User supplied features with m/z, rtime, p-value from a statistical test.: "User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID."
- [results] mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment).: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list"

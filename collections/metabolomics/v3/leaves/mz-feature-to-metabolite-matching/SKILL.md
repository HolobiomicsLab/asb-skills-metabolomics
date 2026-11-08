---
name: mz-feature-to-metabolite-matching
description: Use when you have an untargeted LC/MS feature table (m/z, retention time,
  p-value) and need to map m/z features to candidate metabolites — including ambiguous
  one-to-many matches — as input to pathway/network activity prediction.
when_to_use_negative:
- The goal is definitive metabolite identification — mummichog's m/z matching is not
  designed to replace tandem mass spectrometry fragmentation-based identification
- Input is already a list of identified metabolites rather than raw m/z features (no
  matching step is needed)
- Serum or urine samples where mass flow balance matters — mass balance is almost
  impossible because the reactions producing these metabolites occur in other tissues
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: mummichog
  role: Core package performing the m/z-to-metabolite matching via adduct/isotope
    enumeration and confidence rating
  repo: https://github.com/shuzhao-li/mummichog
- name: metDataModel
  role: Data models structuring compounds (neutral formula, neutral formula mass,
    identifiers) used for matching
- name: Azimuth DB
  role: Chemical database supplying metabolic models and compound formulas for matching
- name: JMS
  role: Mass and empirical compound search functions; can perform annotation on a
    dataset to produce the optional annotation table
  repo: https://github.com/shuzhao-li-lab/JMS
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:14d29c1b4f80db951d1b1612f3d178d0a16b9615b828c7623d42058d9f1892b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/mummichog
  tools:
  - Azimuth DB
  - JMS
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mz-feature-to-metabolite-matching

## Summary

Enumerates all metabolites that could match each m/z feature in an untargeted LC/MS feature table by generating isotopic derivatives and adducts from neutral compound formulas, then applies a confidence-rating filter to retain qualified tentative metabolite matches without MS/MS identification.

## When to use

Use when you have an untargeted LC/MS feature table (m/z, retention time, p-value) and need to map m/z features to candidate metabolites — including ambiguous one-to-many matches — as input to pathway/network activity prediction.

## When NOT to use

- The goal is definitive metabolite identification — mummichog's m/z matching is not designed to replace tandem mass spectrometry fragmentation-based identification
- Input is already a list of identified metabolites rather than raw m/z features (no matching step is needed)
- Serum or urine samples where mass flow balance matters — mass balance is almost impossible because the reactions producing these metabolites occur in other tissues

## Inputs

- Untargeted LC/MS feature table with m/z, retention time, and p-value from a statistical test (e.g., significant list of 601 features and reference list of 7,995 features from the moDC Dataset S1)
- Metabolic model with list_of_pathways, list_of_reactions, list_of_compounds including neutral chemical formulas (e.g., KEGG/UCSD Recon1/Edinburgh human model, BioCyc yeast model)
- Optional annotation table on the features (from authentic standards or MS/MS), structured via metDataModel

## Outputs

- Per-feature lists of possibly matched metabolites with isotopic derivatives and adducts
- Confidence-rated set of qualified metabolite predictions feeding the activity-network step
- Pathway enrichment and network module results written as result tables/figures (result.html) and JSON strings

## How to apply

Load the feature table (m/z, retention time, p-value) and a metabolic model containing list_of_pathways, list_of_reactions, and list_of_compounds with neutral formulas (via metDataModel structures from Azimuth DB). For each m/z feature, enumerate isotopic derivatives and adducts from each candidate compound's neutral formula mass — never salt formulas, since neutral formulas are required to calculate adducts including salts. Apply the confidence rating system: e.g., co-presence of the single-charged M+H[1+] form and its M(C13)+H[1+] 13C isotope form confers high confidence, and qualified metabolites are retained while others are filtered. Optionally enforce the M+H[+] form (positive ion mode) or M−H[−] form (negative ion mode) to restrict predictions — e.g., enforcing M−H[−] on the yeast data had little effect on the 24/28 (86%) agreement with targeted annotation. Accept that a single m/z feature may match multiple metabolites; the downstream network-module step relies on the collective power of the metabolic network to resolve this ambiguity.

## Related tools

- **mummichog** (Core package performing the m/z-to-metabolite matching via adduct/isotope enumeration and confidence rating) — https://github.com/shuzhao-li/mummichog
- **metDataModel** (Data models structuring compounds (neutral formula, neutral formula mass, identifiers) used for matching)
- **Azimuth DB** (Chemical database supplying metabolic models and compound formulas for matching)
- **JMS** (Mass and empirical compound search functions; can perform annotation on a dataset to produce the optional annotation table) — https://github.com/shuzhao-li-lab/JMS

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- moDC/YF17D validation: mummichog predicts a network of 60 metabolites, of which 19 are not found in the original annotation
- Yeast validation: of 28 predicted metabolites comparable to MAVEN's targeted annotation of 101 metabolites, 24 agree (≈0.857); enforcing M−H[−] has little effect
- Confidence rating sanity check: metabolites with both M+H[1+] and M(C13)+H[1+] forms present should be flagged high confidence
- Stability: sweeping the feature-selection p-value cutoff across the Figure S7 range yields predicted metabolite counts bracketing ~60 with largely consistent sets
- Output schema check: result.html is valid HTML and the JSON pathway/network strings parse

## Limitations

- A single m/z feature can match multiple metabolites even with high instrumental accuracy; a feature may match several metabolites in the same module/pathway, inflating the overlap number
- Features from high-resolution profiling experiments far exceed current annotations in metabolite databases, causing a de facto filtering during analysis
- A number of metabolites in metabolomics databases are not present in any metabolic models, and models need improvement on lipid annotation, physiological context and tissue specificity
- Quantification and feature selection steps upstream of mummichog can still be problematic; the current version does not incorporate a statistical matrix from feature selection or mass flow balance

## Evidence

- [results] Confidence rating filters qualified metabolites: "if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
- [results] Optional adduct enforcement: "There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction."
- [results] Input requirements: "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [readme] Neutral formula requirement from README: "We use neutral formula to calculate adducts in mass spec, which includes salts."
- [discussion] Scope limitation: "Mummichog is not designed to replace tandem mass spectrometry in metabolite identification."
- [results] Yeast agreement validation: [evidence span withheld — anchor status: not_found]

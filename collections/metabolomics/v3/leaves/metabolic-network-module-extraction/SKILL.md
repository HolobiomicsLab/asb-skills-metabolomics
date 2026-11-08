---
name: metabolic-network-module-extraction
description: Use when you have an untargeted metabolomics feature table (m/z, retention
  time, p-value) with a significant vs reference feature split, and need pathway/network-level
  activity or predicted metabolite networks without doing MS/MS identification.
when_to_use_negative:
- The goal is definitive metabolite identification — mummichog is not designed to
  replace tandem mass spectrometry in metabolite identification.
- Input is already a set of identified/annotated metabolites rather than raw m/z features;
  the skill's value comes from bypassing identification of spectral features.
- Mass flow balance of reactions within the sample matrix matters (e.g. serum or urine,
  where producing reactions occur in other tissues) — mummichog does not incorporate
  mass balance.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3314
tools:
- name: mummichog
  role: Core algorithm package performing module extraction, activity scoring, and
    activity-network construction from feature tables
  repo: https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- name: metDataModel
  role: Data models for metabolomics used to structure feature/compound annotation
    consumed by mummichog
- name: Azimuth DB
  role: Chemical database supplying metabolic models for the reference network searched
    during module extraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:aca90cf61153d978678ad6d000af630299495837c3b6a81425b6ab10e614cf78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - metDataModel
  - mummichog
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolic-network-module-extraction

## Summary

Extracts structural metabolic modules and an activity network directly from untargeted LC/MS m/z feature tables using mummichog, which matches features to tentative metabolites, searches a genome-scale metabolic model (KEGG/Recon1/Edinburgh human network) for connected subnetworks of 1–4 reactions, and predicts functional activity without a priori metabolite identification.

## When to use

Use when you have an untargeted metabolomics feature table (m/z, retention time, p-value) with a significant vs reference feature split, and need pathway/network-level activity or predicted metabolite networks without doing MS/MS identification.

## When NOT to use

- The goal is definitive metabolite identification — mummichog is not designed to replace tandem mass spectrometry in metabolite identification.
- Input is already a set of identified/annotated metabolites rather than raw m/z features; the skill's value comes from bypassing identification of spectral features.
- Mass flow balance of reactions within the sample matrix matters (e.g. serum or urine, where producing reactions occur in other tissues) — mummichog does not incorporate mass balance.

## Inputs

- m/z feature table with m/z, retention time, and p-value from a statistical test (row_number used as feature ID if none supplied)
- Significant feature list selected by univariate statistics (e.g. 524 gender-difference features from MetaboLights study 439020)
- Reference list of all features detected in the MS experiment
- Metabolic model in JSON format containing list_of_pathways, list_of_reactions, list_of_compounds (with neutral-mass chemical formulas, not salt format)
- Optional annotation table on the features, structured via metDataModel (from authentic standards and MS/MS)

## Outputs

- Predicted activity network of metabolites with connecting reactions (enzymes as EC numbers; e.g. 45 metabolites for the human urine gender analysis, 60 for moDC)
- Pathway enrichment results (Fisher's exact test with EASE score, permutation-based Gamma-modeled null p-values)
- JSON strings from pathway analysis and network module analysis for programmatic use
- Result tables and figures, result.html (metabolites colored by log2 fold change)

## How to apply

Prepare two lists of m/z features: the significant list (e.g. selected by univariate statistics, as with the 524 gender-difference features where testosterone glucuronide was low in females) and the reference list of all features detected. mummichog computes all possibly matched metabolites from these features, searches the reference metabolic network for all modules formable by these tentative metabolites, extracts subnetworks within one to four reactions, and identifies structural modules via Newman's spectral split algorithm. Each candidate module graph is cleaned of protruding edges that do not connect input metabolites, then scored for activity with p-values computed against a Gamma-modeled null from random permutations of m/z features (Fisher's exact test with EASE score for pathway enrichment). Predicted metabolites from significant activities are collected, their isotopic derivatives and adducts looked up, and a confidence rating filter applied (e.g. co-presence of M+H[1+] and M(C13)+H[1+] forms confers high confidence; optionally enforce M+H[+] or M-H[-]). The result is an activity network whose metabolites carry log2 fold-change coloring and connecting enzymes shown as EC numbers.

## Related tools

- **mummichog** (Core algorithm package performing module extraction, activity scoring, and activity-network construction from feature tables) — https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- **metDataModel** (Data models for metabolomics used to structure feature/compound annotation consumed by mummichog)
- **Azimuth DB** (Chemical database supplying metabolic models for the reference network searched during module extraction)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- The reproduced activity network size matches the expected count (e.g. 45 metabolites for the human urine gender-difference analysis; 60 metabolites for the moDC YF17D activation data).
- Agreement between predicted metabolites and the original authors' annotations meets the reference threshold (31/32 overlapping metabolites, 97%, on the human urine dataset).
- Module p-values are computed against a Gamma-modeled null distribution obtained by random permutation of m/z features; check p-value calibration is present.
- Candidate module graphs contain no protruding edges that fail to connect input metabolites.
- Qualified metabolites pass the confidence rating: high confidence requires presence of both M+H[1+] and M(C13)+H[1+] forms.

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number.
- De facto filtering occurs because features from high-resolution profiling experiments far exceed current annotations in metabolite databases, and some metabolites in metabolomics databases are absent from the metabolic models.
- The current version does not incorporate a statistical matrix from the feature selection step prior to analysis, nor mass flow balance of metabolic reactions.
- Metabolic models need improvement on lipid annotation, physiological context and tissue specificity; a single m/z feature can match multiple metabolites even with high instrumental mass accuracy.

## Evidence

- [results] mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites.: "mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites."
- [results] Structural modules are identified within these subnetworks according to Newman's spectral split algorithm: "Structural modules are identified within these subnetworks according to Newman's spectral split algorithm [43]."
- [results] Each graph is cleaned of protruding edges if such edges do not connect input metabolites therefore not contributing to the activity score.: "Each graph is cleaned of protruding edges if such edges do not connect input metabolites therefore not contributing to the activity score."
- [results] A confidence rating system is applied to filter for qualified metabolites.: "A confidence rating system is applied to filter for qualified metabolites."
- [readme] It leverages the organization of metabolic networks to predict functional activity directly from feature tables, bypassing metabolite identification.: "It leverages the organization of metabolic networks to predict functional activity directly from feature tables, bypassing metabolite identification."
- [readme] The format of a metabolic model contains list_of_pathways, list_of_reactions and list_of_compounds.: "The format of a metabolic model contains list_of_pathways, list_of_reactions and list_of_compounds."

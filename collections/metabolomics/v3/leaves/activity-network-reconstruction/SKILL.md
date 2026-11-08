---
name: activity-network-reconstruction
description: Use when you have an untargeted metabolomics feature table (m/z, RT,
  p-value) and want pathway/module-level biological activity without prior metabolite
  identification.
when_to_use_negative:
- The goal is definitive metabolite identification — mummichog is not designed to
  replace tandem mass spectrometry in metabolite identification
- The input is already a list of identified metabolites rather than an m/z feature
  table; standard pathway over-representation tools then apply
- Mass flow balance of reactions is required — mummichog does not incorporate mass
  balance, which is near-impossible in serum or urine samples where producing reactions
  occur in other tissues
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: mummichog
  role: Core algorithm package performing m/z-metabolite matching, module detection
    via Newman's spectral split, and activity network prediction
  repo: https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- name: metDataModel
  role: Data models for metabolomics used to structure feature annotation and metabolic
    models consumed by mummichog
- name: Azimuth DB
  role: Chemical database for biology, including metabolic models, supplying compound
    formulas and adduct/isotope enumeration
- name: JMS
  role: Library for identifier mapping between genome-scale metabolic models and metabolite
    databases, and for mass/empirical-compound search used to build annotations
  repo: https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:acadb6b8ab2b61ae1d8aebadc31ba3e319466d17d3a3839479af3c23413670a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - JMS
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# activity-network-reconstruction

## Summary

Reconstructs a predicted metabolic activity network directly from an untargeted LC/MS feature table (m/z, retention time, p-value) using mummichog, bypassing metabolite identification by leveraging genome-scale metabolic models and network module analysis.

## When to use

Use when you have an untargeted metabolomics feature table (m/z, RT, p-value) and want pathway/module-level biological activity without prior metabolite identification.

## When NOT to use

- The goal is definitive metabolite identification — mummichog is not designed to replace tandem mass spectrometry in metabolite identification
- The input is already a list of identified metabolites rather than an m/z feature table; standard pathway over-representation tools then apply
- Mass flow balance of reactions is required — mummichog does not incorporate mass balance, which is near-impossible in serum or urine samples where producing reactions occur in other tissues

## Inputs

- User-supplied feature table with m/z, rtime, and p-value from a statistical test (e.g. TSV; row_number used as ID if no feature ID given)
- Optional annotation table on the features (from authentic standards and MS/MS, structured with metDataModel)
- Optional metabolic model (JSON) with list_of_pathways, list_of_reactions, list_of_compounds including neutral chemical formulas; default models (KEGG, UCSD Recon1, Edinburgh human metabolic network, BioCyc) supplied by mummichog

## Outputs

- Result tables and figures plus result.html (as in version 2)
- JSON strings from pathway analysis and network module analysis for programmatic use
- Predicted activity network of metabolites (e.g. 60 metabolites for moDC/YF17D, 19 not in original annotation), colored by log2 fold change with EC-number enzymes

## How to apply

Load the feature table (m/z, retention time, p-value from a statistical test such as a t-test) and a metabolic model containing list_of_pathways, list_of_reactions, and list_of_compounds. Match all possibly-matched metabolites to each m/z feature by enumerating isotopic derivatives and adducts from neutral compound formulas (e.g. M+H[1+] co-present with M(C13)+H[1+] confers high confidence; optionally enforce M+H[+] in positive mode or M−H[−] in negative mode). Extract subnetworks within one to four reactions of matched metabolites, clean protruding edges that do not connect input metabolites, and identify structural modules with Newman's spectral split algorithm while extensively sampling subnetwork structures. Score module activity, estimate pathway enrichment by Fisher's exact test with the EASE (smaller-of overlap) correction, and model permutation-based null p-value distributions as Gamma. Collapse significant modules into the predicted activity network and write result tables, figures (metabolites colored by log2 fold change, enzymes as EC numbers), result.html, and JSON strings for programmatic use.

## Related tools

- **mummichog** (Core algorithm package performing m/z-metabolite matching, module detection via Newman's spectral split, and activity network prediction) — https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- **metDataModel** (Data models for metabolomics used to structure feature annotation and metabolic models consumed by mummichog)
- **Azimuth DB** (Chemical database for biology, including metabolic models, supplying compound formulas and adduct/isotope enumeration)
- **JMS** (Library for identifier mapping between genome-scale metabolic models and metabolite databases, and for mass/empirical-compound search used to build annotations) — https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- Reproduce the moDC/YF17D validation: mummichog predicts a network of 60 metabolites, of which 19 are not found in the original annotation
- Significant pathways reported in result.html are consistent with the published moDC analysis
- Confidence ratings behave as specified: co-presence of M+H[1+] and M(C13)+H[1+] forms confers high confidence on a metabolite prediction
- Protruding edges in candidate module graphs are removed when they do not connect input metabolites and thus do not contribute to the activity score
- Pathway p-values follow the permutation-based null modeled as Gamma with the EASE smaller-of correction applied

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number
- Features from high-resolution profiling experiments far exceed current annotations in metabolite databases, causing de facto filtering; some metabolite-database compounds are absent from the metabolic models
- Quantification and feature selection steps upstream of mummichog can still be problematic, and validation is particularly challenging for multivariate in vivo analyses
- Models need improvement on lipid annotation, physiological context, and tissue specificity; the current version does not incorporate a statistical matrix from the feature selection step or mass flow balance
- Salt vs neutral formula handling matters: neutral formula (not salt format) must be used to calculate adducts in mass spectrometry

## Evidence

- [readme] Bypassing identification via network leverage: "It leverages the organization of metabolic networks to predict functional activity directly from feature tables, bypassing metabolite identification."
- [results] Module identification algorithm: "Structural modules are identified within these subnetworks according to Newman's spectral split algorithm [43]."
- [results] Confidence rating rule: "if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
- [results] Subnetwork extraction and cleaning: "Each graph is cleaned of protruding edges if such edges do not connect input metabolites therefore not contributing to the activity score."
- [supplementary] moDC validation output: "Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation."
- [readme] Input feature requirements: "User supplied features with m/z, rtime, p-value from a statistical test. If no unique feature ID is supplied, row_number will be used as ID."
- [readme] Model formula requirement: "We use neutral formula to calculate adducts in mass spec, which includes salts."
- [discussion] Overlap inflation caveat: "Sometimes, a single m/z feature may match to several metabolites in the same module/pathway, inflating the overlap number."

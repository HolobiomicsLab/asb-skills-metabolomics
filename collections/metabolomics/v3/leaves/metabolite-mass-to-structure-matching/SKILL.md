---
name: metabolite-mass-to-structure-matching
description: Use when you have a table of significant m/z features plus a reference
  list of all detected features from LC/MS metabolomics, and you need tentative metabolite
  predictions (not definitive IDs) as input to pathway/network activity analysis.
when_to_use_negative:
- When definitive metabolite identification is required — 'Mummichog is not designed
  to replace tandem mass spectrometry in metabolite identification.'
- When the input is already a list of annotated metabolites rather than raw m/z features;
  the matching step is unnecessary
- When quantification/feature selection upstream is unreliable — the article notes
  these upstream steps can still be problematic and propagate errors
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mummichog
  role: Core algorithm that computes possibly matched metabolites from m/z features
    and supports the M+H[+] enforcement option and confidence rating system
  repo: https://github.com/shuzhao-li/mummichog
- name: metDataModel
  role: Data models for metabolomics used by mummichog to structure compounds and
    annotations for matching
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:11302be638e462af99f79828051c468ef69f1536a012d34ce4a032b41d26607d
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
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolite-mass-to-structure-matching

## Summary

Tentatively match m/z features from an untargeted metabolomics feature table to metabolites in a genome-scale metabolic model (e.g. the BioCyc human model), using mummichog's adduct/ion-form logic such as enforcing the M+H[+] form in positive mode, to feed pathway and network-module activity prediction without a priori metabolite identification.

## When to use

Use when you have a table of significant m/z features plus a reference list of all detected features from LC/MS metabolomics, and you need tentative metabolite predictions (not definitive IDs) as input to pathway/network activity analysis.

## When NOT to use

- When definitive metabolite identification is required — 'Mummichog is not designed to replace tandem mass spectrometry in metabolite identification.'
- When the input is already a list of annotated metabolites rather than raw m/z features; the matching step is unnecessary
- When quantification/feature selection upstream is unreliable — the article notes these upstream steps can still be problematic and propagate errors

## Inputs

- Filtered LC/MS feature table with significant m/z features (e.g. the Roux et al 2012 human urine dataset)
- Reference list of all m/z features detected in the MS experiment
- Genome-scale metabolic model with chemical formulas (e.g. BioCyc human metabolic model)

## Outputs

- List of possibly matched (tentative) metabolites from the m/z features
- JSON outputs of pathway analysis and network module analysis
- Predicted-vs-annotated agreement summary (e.g. urine_mh_agreement.csv, 41/41 = 100%)

## How to apply

Load the filtered feature table — for the validation case, the human urine data set from Roux et al 2012 with its significant m/z features and reference list of all detected features — and invoke the mummichog CLI (python3 -m mummichog.main) with a chosen metabolic model, e.g. the BioCyc human metabolic model. mummichog computes all possibly matched metabolites from the m/z features; because a single m/z can match multiple metabolites, apply the confidence rating system — presence of both the M+H[1+] and M(C13)+H[1+] forms confers high confidence — and optionally enable the option to enforce the presence of the M+H[+] form (positive mode) or M−H[−] (negative mode). The matched metabolites then search the reference metabolic network for modules and feed Fisher's exact test (EASE score) pathway enrichment and network module analysis against a Gamma-modeled permutation null. Compare predicted metabolites against an independent annotation to quantify agreement; with M+H[+] enforcement on the urine analysis, 41 of 44 predicted metabolites matched the original annotation exactly (3 were not in it).

## Related tools

- **mummichog** (Core algorithm that computes possibly matched metabolites from m/z features and supports the M+H[+] enforcement option and confidence rating system) — https://github.com/shuzhao-li/mummichog
- **metDataModel** (Data models for metabolomics used by mummichog to structure compounds and annotations for matching)

## Examples

```
python3 -m mummichog.main -f mummichog/tests/testdata0710.txt -o t3
```

## Evaluation signals

- With M+H[+] enforcement on the Roux et al 2012 urine analysis, predicted-vs-annotated agreement must equal 41/41 metabolites (100%), with 3 of 44 predictions outside the original annotation
- Metabolite predictions where both M+H[1+] and M(C13)+H[1+] forms are present must carry the high-confidence rating
- JSON strings of pathway analysis and network module analysis must be emitted as programmatic outputs alongside result tables and figures
- Fisher's exact test with EASE score and Gamma-modeled permutation adjusted p-values must be applied per pathway

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number
- Features from high-resolution profiling experiments far exceed current annotations in metabolite databases, causing de facto filtering
- Some compounds in a GSMM lack chemical formulas; neutral (not salt) formulas are required for adduct calculation, and missing formulas need lookup via compound identifiers

## Evidence

- [results] M+H[+] enforcement option: "There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction."
- [results] confidence rating system: "if both the single-charged form M+H[1+] and the form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
- [results] computation of matched metabolites: "mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites."
- [results] input requirements: "mummichog requires two lists of m/z features, the significant list (e.g. selected by univariate statistics) and the reference list (all features detected in the MS experiment)."
- [readme] tool purpose: "It leverages the organization of metabolic networks to predict functional activity directly from feature tables, bypassing metabolite identification."
- [discussion] scope limit on identification: "Mummichog is not designed to replace tandem mass spectrometry in metabolite identification."

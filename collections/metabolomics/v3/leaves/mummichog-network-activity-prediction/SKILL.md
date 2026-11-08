---
name: mummichog-network-activity-prediction
description: Use when you have two m/z feature lists — a significant list selected
  by univariate statistics and a reference list of all detected features — and need
  to infer pathway/module activity without metabolite identification.
when_to_use_negative:
- When metabolite identification is required — mummichog is not designed to replace
  tandem mass spectrometry in metabolite identification
- When a statistical matrix from feature selection or mass flow balance of metabolic
  reactions is needed — the current mummichog version does not incorporate these
- When the input is a list of already-identified metabolites rather than raw m/z features
  — mummichog requires m/z feature lists, not annotation tables alone
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mummichog
  role: Core algorithm package that performs pathway/network analysis, computing possibly
    matched metabolites, module sampling, permutation-based p-values, and building
    the activity network
  repo: http://atcg.googlecode.com
- name: cloud-mummichog
  role: Server and worker (RESTful) implementations of the mummichog workflow
- name: Azimuth DB
  role: Chemical database for biology, including metabolic models, providing compound
    formulas and identifiers needed by mummichog
- name: metDataModel
  role: Data models for metabolomics used to structure annotation tables and metabolic
    models consumed by mummichog
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:cdffa86682a481da85835cb7a602561666459effc3d1536be10a27d3f3764cd1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - cloud-mummichog
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mummichog-network-activity-prediction

## Summary

mummichog predicts functional network activity directly from an untargeted LC/MS feature table by matching m/z features to possibly matched metabolites, sampling modules in a genome-scale metabolic model, and estimating null module-activity distributions via permutations — bypassing a priori metabolite identification.

## When to use

Use when you have two m/z feature lists — a significant list selected by univariate statistics and a reference list of all detected features — and need to infer pathway/module activity without metabolite identification.

## When NOT to use

- When metabolite identification is required — mummichog is not designed to replace tandem mass spectrometry in metabolite identification
- When a statistical matrix from feature selection or mass flow balance of metabolic reactions is needed — the current mummichog version does not incorporate these
- When the input is a list of already-identified metabolites rather than raw m/z features — mummichog requires m/z feature lists, not annotation tables alone

## Inputs

- Significant list of m/z features (e.g. selected by univariate statistics)
- Reference list of all features detected in the MS experiment
- Optional user-supplied annotation table on the features (metDataModel format)
- Optional metabolic model in JSON format (list_of_pathways, list_of_reactions, list_of_compounds with neutral chemical formulas); mummichog default models include KEGG/Recon1/Edinburgh human metabolic network

## Outputs

- Result tables and figures (result.html as in version 2)
- JSON strings from pathway analysis (Fisher's exact test with EASE score, Gamma-modeled adjusted p-values)
- JSON strings from network module analysis (activity network of predicted metabolites with connecting EC numbers)
- Predicted metabolite activity network colored by log2 fold change

## How to apply

Provide mummichog with two lists of m/z features: a significant list (e.g. selected by univariate statistics) and a reference list of all features detected in the MS experiment. mummichog computes all possibly matched metabolites from the m/z features, searches the reference metabolic network for all modules that can be formed by these tentative metabolites, and estimates null module-activity distributions by random permutations of m/z features (modeled as a Gamma distribution, with the EASE score for robustness in Fisher's exact test pathway enrichment). Structural modules are identified within subnetworks of one to four reactions using Newman's spectral split algorithm, with candidate graphs cleaned of protruding edges that do not connect input metabolites. Finally, mummichog collects predicted metabolites from significant activities, looks up their isotopic derivatives and adducts, applies a confidence rating (high confidence when both M+H[1+] and M(C13)+H[1+] forms are present), and carries over connections from the reference network to build the activity network.

## Related tools

- **mummichog** (Core algorithm package that performs pathway/network analysis, computing possibly matched metabolites, module sampling, permutation-based p-values, and building the activity network) — http://atcg.googlecode.com
- **cloud-mummichog** (Server and worker (RESTful) implementations of the mummichog workflow)
- **Azimuth DB** (Chemical database for biology, including metabolic models, providing compound formulas and identifiers needed by mummichog)
- **metDataModel** (Data models for metabolomics used to structure annotation tables and metabolic models consumed by mummichog)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- For the human urinary gender-difference analysis, the reproduced mummichog activity network should contain 45 metabolites and 97% agreement (31/32) with the original authors' annotation
- For the moDC yellow fever virus activation data (7,995 spectral features, 601 significant), the predicted activity network should reproduce the published module structure
- Confirm that the model JSON supplies neutral (not salt) chemical formulas of compounds, as mummichog uses neutral formula mass to calculate adducts including salts
- Verify the null distribution of module activity is Gamma-modeled and permutation-based (random permutations of m/z features)
- Check the confidence rating: metabolite predictions carrying both M+H[1+] and M(C13)+H[1+] forms should be rated high confidence

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number
- Quantification and feature selection, the steps upstream of mummichog, can still be problematic
- Features from a high-resolution profiling experiment by far exceed the current annotations in metabolite databases, leading to de facto filtering
- Validation is particularly challenging for multivariate analysis in in vivo studies
- Current mummichog version does not incorporate a statistical matrix from the feature selection step nor mass flow balance of metabolic reactions

## Evidence

- [results] mummichog requires two lists of m/z features, the significant list (e.g. selected by univariate statistics) and the reference list (all features detected in the MS experiment): "mummichog requires two lists of m/z features, the significant list  (e.g. selected by univariate statistics) and the reference list  (all features detected in the MS experiment)."
- [results] mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites: "mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites."
- [results] A confidence rating system is applied to filter for qualified metabolites: "Mummichog collects these metabolites, and look up all their isotopic derivatives and adducts in . A confidence rating system is applied to filter for qualified metabolites."
- [results] The basic test for pathway enrichment here is Fisher's exact test (FET): "The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis."
- [discussion] This leads to a de facto filtering when data are run on mummichog: "This leads to a de facto filtering when data are run on mummichog (similiar situation in database searches)."
- [readme] Mummichog is a Python program for analyzing data from high throughput, untargeted metabolomics: "Mummichog is a Python program for analyzing data from high throughput, untargeted metabolomics."
- [readme] python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .: "(base) MLG-JGM467:mummichog lish$ python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d ."
- [readme] If compound formulas are provided in a model, be sure to have correct neutral mass and not a salt format: "If compound formulas are provided in a model, be sure to have correct neutral mass and not a salt format."

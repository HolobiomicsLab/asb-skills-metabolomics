---
name: newman-spectral-split-subnetwork-identification
description: Use when running mummichog module analysis on a significant m/z feature
  list and you need to partition the matched-metabolite subnetworks into structural
  modules for activity scoring against a permuted null.
when_to_use_negative:
- When the goal is definitive metabolite identification — mummichog is not designed
  to replace tandem mass spectrometry in metabolite identification
- When mass balance across reactions is required — mass balance is almost impossible
  within serum or urine samples because producing reactions occur in other tissues
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: mummichog
  role: Core algorithm package whose module analysis extracts one-to-four-reaction
    subnetworks, applies Newman's spectral split, scores activity, and computes permutation-based
    p-values
  repo: https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- name: metDataModel
  role: Data models for metabolomics used by mummichog to structure the metabolic
    model and feature data used in module identification
- name: Azimuth DB
  role: Chemical database for biology, including metabolic models, providing the reference
    network compounds from which modules are formed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:fa16a635b0f3bcca5504e19e5e9b4a8262f94790504c73cf05e8570f902bb964
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
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

# newman-spectral-split-subnetwork-identification

## Summary

Within mummichog's module analysis, tentatively matched metabolites are grouped into subnetworks of one to four reactions, and structural modules are identified using Newman's spectral split algorithm to score pathway/network activity from untargeted LC/MS feature tables without prior metabolite identification.

## When to use

Use when running mummichog module analysis on a significant m/z feature list and you need to partition the matched-metabolite subnetworks into structural modules for activity scoring against a permuted null.

## When NOT to use

- When the goal is definitive metabolite identification — mummichog is not designed to replace tandem mass spectrometry in metabolite identification
- When mass balance across reactions is required — mass balance is almost impossible within serum or urine samples because producing reactions occur in other tissues

## Inputs

- Significant m/z feature list (e.g. 601 features selected by univariate statistics from the moDC experiment)
- Reference list of all detected m/z features (e.g. 7,995 spectral features post QC)
- Genome-scale metabolic model with list_of_pathways, list_of_reactions and list_of_compounds (KEGG/UCSD Recon1/Edinburgh human metabolic network, or BioCyc alternative)

## Outputs

- Structural modules within one-to-four-reaction subnetworks, identified by Newman's spectral split
- Module activity scores with p-values against a Gamma-modeled null from random permutations of m/z features
- Predicted metabolites feeding the activity network (e.g. 60 metabolites, 19 not in the original annotation) and JSON strings from network module analysis

## How to apply

First compute all possibly matched metabolites from the significant m/z list (e.g. the 601 significant features of the moDC YF17D experiment against the 7,995-feature reference list) and extract subnetworks within one to four reactions from the genome-scale human metabolic network (KEGG, UCSD Recon1, Edinburgh human metabolic network). Apply Newman's spectral split algorithm within each subnetwork to identify structural modules. Score the activity of each module and compute p-values against a null distribution obtained by random permutations of m/z features, which is modeled as Gamma. Before scoring, clean candidate module graphs of protruding edges that do not connect input metabolites, since such edges do not contribute to the activity score. The resulting significant modules feed downstream construction of the activity network and pathway enrichment (Fisher's exact test with EASE score).

## Related tools

- **mummichog** (Core algorithm package whose module analysis extracts one-to-four-reaction subnetworks, applies Newman's spectral split, scores activity, and computes permutation-based p-values) — https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- **metDataModel** (Data models for metabolomics used by mummichog to structure the metabolic model and feature data used in module identification)
- **Azimuth DB** (Chemical database for biology, including metabolic models, providing the reference network compounds from which modules are formed)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- On the moDC YF17D vs mock data, the analysis reproduces a network of 60 metabolites, 19 of which are not found in the original annotation (Dataset S1 comparison)
- The arginine/citrulline and glutathione/GSSG subnetworks show significant activity, consistent with nitric oxide synthase pathway activation
- Module p-values conform to the Gamma-modeled null distribution built from random permutations of m/z features
- Cleaned module graphs retain only edges connecting input metabolites; protruding edges not connecting input metabolites are removed before activity scoring

## Limitations

- A single m/z feature may match several metabolites in the same module/pathway, inflating the overlap number
- Quantification and feature selection upstream of mummichog can still be problematic and are not incorporated as a statistical matrix in the analysis
- A number of metabolites in metabolomics databases are not present in any of the metabolic models, causing de facto filtering of features
- Metabolic models need improvement on lipid annotation, physiological context and tissue specificity

## Evidence

- [results] Structural modules are identified within these subnetworks according to Newman's spectral split algorithm: "Structural modules are identified within these subnetworks according to Newman's spectral split algorithm [43]."
- [results] Module analysis workflow in mummichog: "mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites."
- [results] Graph cleaning before activity scoring: "Each graph is cleaned of protruding edges if such edges do not connect input metabolites therefore not contributing to the activity score."
- [results] Null distribution via permutation and Gamma model: "The basic test for pathway enrichment here is Fisher's exact test (FET), which is widely used in transcriptomic analysis."
- [supplementary] Validated prediction size on moDC data: "Mummichog predicts a network of 60 metabolites, in which 19 are not found in the original annotation."
- [discussion] Module finding design rationale: "The module finding procedure in the program was designed to extensively sample subnetwork structures."

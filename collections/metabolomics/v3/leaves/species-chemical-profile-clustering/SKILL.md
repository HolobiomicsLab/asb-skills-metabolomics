---
name: species-chemical-profile-clustering
description: Use when you have ConCISE consensus annotations for ion features from
  multiple species/conditions and want to test whether consensus chemical-class profiles
  (relative production per class) group samples by phenotype or toxicity.
when_to_use_negative:
- Input is a raw feature table or molecular networking output that has not yet been
  annotated — run ConCISE consensus annotation propagation first
- The analysis goal is absolute metabolite quantification across species rather than
  relative chemical-class profile comparison
- The dataset has too few samples/species (e.g., fewer than 3) for hierarchical clustering
  to be meaningful
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: ConCISE
  role: Produces the consensus ClassyFire annotations per ion feature that are aggregated
    into class-abundance profiles
  repo: https://github.com/Zquinlan/conCISE
- name: Python
  role: Pandas for aggregation of consensus classifications per species; scipy.cluster.hierarchy
    (linkage, dendrogram) and matplotlib for clustering and dendrogram rendering
- name: ClassyFire
  role: ChemOnt hierarchical ontology (superclass, class, subclass) defining the chemical
    classes used as clustering features
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:a85d4a452d4db3d2e00b6a3faa436c6c46e060722d57bc87b18ca3bbed4dc612
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zquinlan/conCISE
  tools:
  - ClassyFire
  - ConCISE
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# species-chemical-profile-clustering

## Summary

Clusters biological samples/species by their relative production of ConCISE consensus chemical classification classes (ClassyFire superclass/class/subclass), revealing chemotaxonomic groupings such as the two domoic-acid-producing toxic Pseudo-nitzschia species sharing one clade in Case Study 2.

## When to use

Use when you have ConCISE consensus annotations for ion features from multiple species/conditions and want to test whether consensus chemical-class profiles (relative production per class) group samples by phenotype or toxicity.

## When NOT to use

- Input is a raw feature table or molecular networking output that has not yet been annotated — run ConCISE consensus annotation propagation first
- The analysis goal is absolute metabolite quantification across species rather than relative chemical-class profile comparison
- The dataset has too few samples/species (e.g., fewer than 3) for hierarchical clustering to be meaningful

## Inputs

- ConCISE consensus annotation file for a GNPS FBMN task (per-ion-feature consensus ClassyFire superclass/class/subclass labels with percent consensus and species/sample of origin)
- Species/isolate metadata mapping ion features to the five Pseudo-nitzschia isolates (MSV000081731)

## Outputs

- species_class_abundance.csv — species-by-chemical-class relative abundance matrix
- species_cluster_dendrogram.png — hierarchical clustering dendrogram of species

## How to apply

Load the ConCISE consensus annotation file (e.g., for GNPS task MSV000081731) with pandas, retaining per-ion-feature ClassyFire superclass/class/subclass consensus labels, their percent consensus values, and the species of origin. Aggregate per species and compute the relative production of each chemical class as the fraction of annotated ion features (and their total intensity) assigned to that class, so fractions sum to 1 across classes per species. Assemble the species-by-class relative-abundance matrix and save it (e.g., species_class_abundance.csv). Cluster samples with scipy.cluster.hierarchy.linkage on these profiles (e.g., average linkage) and render a dendrogram with scipy.cluster.hierarchy.dendrogram and matplotlib. Validate that the dendrogram reproduces the expected grouping (the two domoic-acid-producing toxic Pseudo-nitzschia together) and that the annotation coverage and consensus levels match expectations (56.3% of ion features annotated, mean consensus 87.8%).

## Related tools

- **ConCISE** (Produces the consensus ClassyFire annotations per ion feature that are aggregated into class-abundance profiles) — https://github.com/Zquinlan/conCISE
- **Python** (Pandas for aggregation of consensus classifications per species; scipy.cluster.hierarchy (linkage, dendrogram) and matplotlib for clustering and dendrogram rendering)
- **ClassyFire** (ChemOnt hierarchical ontology (superclass, class, subclass) defining the chemical classes used as clustering features)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Abundance fractions per species sum to 1 across chemical classes
- The dendrogram places the two domoic-acid-producing toxic Pseudo-nitzschia species together in one clade, as reported in the paper
- Fraction of ion features receiving a putative consensus annotation approximates the reported 56.3%
- Mean consensus of annotated features approximates the reported 87.8%

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches; if library matches lack superclass/class/subclass columns, use the NPClassifier workaround or manually merge Fiehn Labs classyFire Batch results into the DBResult file before clustering
- Only a minority of ion features (~56% in Case Study 2) receive consensus annotations, so profiles represent annotated features only
- Class-level consensus thresholds (default 50% superclass, 70% class, 70% subclass) filter which annotations enter the profile; different thresholds change the clustering input

## Evidence

- [results] Clustering the five Pseudo-nitzschia species by the relative production of each ConCISE consensus classification grouped the two domoic-acid-producing toxic species together: [evidence span withheld — anchor status: not_found]
- [abstract] ConCISE consensus across ChemOnt levels: "ConCISE maintains a true positive rate greater than 95% across all levels of the ChemOnt hierarchical ontology used by the ClassyFire annotation software (superclass, class, subclass)"
- [readme] CLI consensus thresholds: "Superclass percent consensus (optional; default = 50)
- Class percent consensus (optional; default = 70)
- Subclass percent consensus (optional; default = 70)"
- [methods] Consensus annotation method: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS"

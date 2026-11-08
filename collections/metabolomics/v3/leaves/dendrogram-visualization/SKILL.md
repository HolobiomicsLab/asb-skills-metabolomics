---
name: dendrogram-visualization
description: Use when you have a samples-by-chemical-class relative-abundance matrix
  (e.g. from ConCISE consensus ClassyFire annotations of ion features) and need to
  visualize which samples group together, e.g. to test whether toxic and non-toxic
  strains separate.
when_to_use_negative:
- Input is raw MS/MS data or an unannotated feature table — run ConCISE consensus
  annotation first; there are no class labels to cluster on yet
- Your question concerns annotation quality (consensus percentages, TPR) rather than
  comparing chemical-composition profiles between organisms or conditions
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Used with pandas to build the species-by-class relative-abundance matrix and
    with scipy to compute linkage and render the dendrogram
  repo: https://github.com/Zquinlan/conCISE
- name: conCISE
  role: Produces the consensus ClassyFire (or NPClassifier) annotations whose per-class
    relative production profiles are the clustering input
  repo: https://github.com/Zquinlan/conCISE
- name: myBinder
  role: Free virtual machine (Jupyter notebook) for running the ConCISE workflow that
    generates the consensus annotation file upstream of clustering
  repo: https://mybinder.org/v2/gh/Zquinlan/conCISE/HEAD?labpath=src%2FconciseBinder.ipynb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:d00ec05b16f93d55a0c6c35b3b9917cf6a6eb75f85d1bffb3e29864613a8c822
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zquinlan/conCISE
  tools:
  - Python
  - conCISE
  - myBinder
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# dendrogram-visualization

## Summary

Hierarchical clustering of samples (here, five Pseudo-nitzschia isolates) by their ConCISE consensus chemical-class profiles, followed by drawing the resulting species dendrogram with scipy.cluster.hierarchy in Python to visually test whether metabolomic phenotypes (e.g. domoic-acid-producing toxic species) cluster together.

## When to use

Use when you have a samples-by-chemical-class relative-abundance matrix (e.g. from ConCISE consensus ClassyFire annotations of ion features) and need to visualize which samples group together, e.g. to test whether toxic and non-toxic strains separate.

## When NOT to use

- Input is raw MS/MS data or an unannotated feature table — run ConCISE consensus annotation first; there are no class labels to cluster on yet
- Your question concerns annotation quality (consensus percentages, TPR) rather than comparing chemical-composition profiles between organisms or conditions

## Inputs

- ConCISE consensus annotation file for the dataset (e.g. MSV000081731), with per-ion-feature ClassyFire superclass/class/subclass labels, consensus percentages, and species of origin
- species_class_abundance.csv — five-species by chemical-class relative-abundance matrix built with pandas

## Outputs

- species_cluster_dendrogram.png — hierarchical clustering dendrogram of the five Pseudo-nitzschia species
- Linkage matrix from scipy.cluster.hierarchy.linkage (average linkage) encoding clade structure and merge distances

## How to apply

First assemble the five-species by chemical-class matrix with pandas, computing each class's relative production per species as the fraction of annotated ion features (and total intensity) assigned to that class — each row must sum to 1 across classes. Then cluster the samples with scipy.cluster.hierarchy.linkage on these relative class-abundance profiles (average linkage as in the paper's Case Study 2). Finally, draw the tree with scipy.cluster.hierarchy.dendrogram and matplotlib and save it as an image (species_cluster_dendrogram.png). Rationale: ConCISE annotates 56.3% of ion features with a mean consensus of 87.8% across ClassyFire superclass/class/subclass levels, giving a stable chemical-ontology vector per organism; the dendrogram then tests whether domoic-acid-producing toxic species share a clade, as reported in the paper.

## Related tools

- **Python** (Used with pandas to build the species-by-class relative-abundance matrix and with scipy to compute linkage and render the dendrogram) — https://github.com/Zquinlan/conCISE
- **conCISE** (Produces the consensus ClassyFire (or NPClassifier) annotations whose per-class relative production profiles are the clustering input) — https://github.com/Zquinlan/conCISE
- **myBinder** (Free virtual machine (Jupyter notebook) for running the ConCISE workflow that generates the consensus annotation file upstream of clustering) — https://mybinder.org/v2/gh/Zquinlan/conCISE/HEAD?labpath=src%2FconciseBinder.ipynb

## Examples

```
from scipy.cluster.hierarchy import linkage, dendrogram; Z = linkage(species_class_abundance, method='average'); dendrogram(Z)
```

## Evaluation signals

- Dendrogram places the two domoic-acid-producing toxic Pseudo-nitzschia species together in one clade, matching the paper's Case Study 2 result
- Relative class-abundance fractions sum to 1 across classes for each species before clustering
- Row/column dimensions of the abundance matrix equal the number of species (5) and the number of distinct consensus chemical classes
- Linkage is computed on relative (normalized) class profiles, not raw ion-feature counts

## Limitations

- Only 56.3% of ion features received a putative consensus annotation in the paper's dataset, so the profile covers a subset of the metabolome; unannotated features are invisible to the clustering
- GNPS has stopped supplying ClassyFire ontology information for spectral library matches — if ClassyFire data is missing, either switch to NPClassifier (NPC=True) or manually pull ClassyFire classes via the Fiehn Labs classyfire Batch using InChiKeys from the GNPS DBResult file
- Consensus thresholds (default 50% superclass, 70% class, 70% subclass) filter which labels survive; changing them changes the abundance matrix and hence the dendrogram topology

## Evidence

- [results] Clustering the five Pseudo-nitzschia species by the relative production of each ConCISE consensus classification grouped the two domoic-acid-producing toxic species together: [evidence span withheld — anchor status: not_found]
- [readme] ConCISE finds consensus annotations using ClassyFire ontologies supplied by GNPS: "ConCISE works by finding consensus annotations of putative annotations using the ClassyFire ontologies which are supplied by GNPS"
- [readme] README workaround for missing ClassyFire data from GNPS: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches."

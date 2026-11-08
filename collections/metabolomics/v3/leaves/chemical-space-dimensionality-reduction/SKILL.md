---
name: chemical-space-dimensionality-reduction
description: Use when you need to visually compare the structural diversity or chemical-space
  coverage of multiple compound libraries (e.g., OMSLs vs. natural-product catalogues)
  by embedding a fixed molecular fingerprint representation into 2-D with t-SNE.
when_to_use_negative:
- Input compounds already have a low-dimensional embedding or PCA/UMAP coordinates
  available — t-SNE is stochastic and not designed to replace an existing projection.
- The goal is quantitative similarity search between individual spectra rather than
  global chemical-space visualization — use spectral similarity scoring (dot product
  / reverse dot product) instead.
- Compound identifiers lack normalized InChIKeys or PubChem fingerprint coverage (e.g.,
  purely in-silico spectra without chemical identifiers), so the descriptor vector
  cannot be built.
edam_operation: http://edamontology.org/operation_3938
edam_topics:
- http://edamontology.org/topic_3339
- http://edamontology.org/topic_0154
tools:
- name: FragHub
  role: Produces the standardize lib files containing normalized InChIKeys and harmonized
    metadata used as the compound source for the t-SNE projection
  repo: https://github.com/eMetaboHUB/FragHub
- name: PubChem
  role: Source of the molecular fingerprint descriptors (offline PubChem datas bundled
    with FragHub) used as the t-SNE input
- name: NPClassifier
  role: Provides PathwayNP / superclassNP coloration for each compound in the t-SNE
    scatter plot
- name: Python
  role: Implementation language for the t-SNE run (sklearn.manifold.TSNE) and matplotlib
    figure rendering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:464e2714809e9ea3607841c2fac27580bb8e7f17f1aedd9c2fb85e5be04af75e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  repo_url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - NPClassifier
  - PubChem
  - Python
  edam_operation: http://edamontology.org/operation_3938
  edam_topics:
  - http://edamontology.org/topic_3339
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chemical-space-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces PubChem molecular fingerprints to a 2-D t-SNE projection so that the chemical-space coverage of different open mass spectral libraries (GNPS, MoNA, MassBank, MS-Dial-VS17) can be visually compared, with compounds colored by NPClassifier superclass.

## When to use

Use when you need to visually compare the structural diversity or chemical-space coverage of multiple compound libraries (e.g., OMSLs vs. natural-product catalogues) by embedding a fixed molecular fingerprint representation into 2-D with t-SNE.

## When NOT to use

- Input compounds already have a low-dimensional embedding or PCA/UMAP coordinates available — t-SNE is stochastic and not designed to replace an existing projection.
- The goal is quantitative similarity search between individual spectra rather than global chemical-space visualization — use spectral similarity scoring (dot product / reverse dot product) instead.
- Compound identifiers lack normalized InChIKeys or PubChem fingerprint coverage (e.g., purely in-silico spectra without chemical identifiers), so the descriptor vector cannot be built.

## Inputs

- InChIKey identifiers for compounds from each OMSL (GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial-VS17) produced by FragHub standardization
- PubChem fingerprints (offline PubChem data bundled with FragHub) keyed by InChIKey
- NPClassifier PathwayNP and superclassNP annotations per compound

## Outputs

- 2-D t-SNE projection of the pooled PubChem fingerprint space, labeled by originating library
- Matplotlib scatter plot 'tsne_chemical_space.png' colored by NPClassifier superclass, with per-library legend and compound counts

## How to apply

Extract the normalized InChIKey for every compound in each library (as produced by FragHub's RDKit standardization), retrieve the corresponding PubChem fingerprint from the offline PubChem data bundled with FragHub, and use the fingerprint block as the molecular descriptor vector. Run sklearn.manifold.TSNE over the pooled, library-labeled compound set with perplexity = 30 and exaggeration = 1, matching the parameters reported in the article. Color each point by its NPClassifier PathwayNP / superclassNP annotation (retrieved via the NPClassifier API) and add a legend that distinguishes the constituent libraries and their compound counts (e.g., 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17, 32,193 FragHub unique chemicals). The rationale is that comparing the t-SNE footprints side-by-side reveals which structural regions each OMSL covers and which regions are underrepresented, motivating orthogonal library-expansion strategies. Validate that the projection covers the reported per-library compound counts and that the rendered figure is saved as tsne_chemical_space.png.

## Related tools

- **FragHub** (Produces the standardize lib files containing normalized InChIKeys and harmonized metadata used as the compound source for the t-SNE projection) — https://github.com/eMetaboHUB/FragHub
- **PubChem** (Source of the molecular fingerprint descriptors (offline PubChem datas bundled with FragHub) used as the t-SNE input)
- **NPClassifier** (Provides PathwayNP / superclassNP coloration for each compound in the t-SNE scatter plot)
- **Python** (Implementation language for the t-SNE run (sklearn.manifold.TSNE) and matplotlib figure rendering)

## Examples

```
from sklearn.manifold import TSNE; import numpy as np; X = np.load('pubchem_fingerprints.npy'); labels = np.load('library_labels.npy'); emb = TSNE(n_components=2, perplexity=30, exaggeration=1).fit_transform(X); np.save('tsne_embedding.npy', emb)
```

## Evaluation signals

- The script runs end-to-end and produces tsne_chemical_space.png.
- The pooled compound set matches the reported per-library counts: 13,507 GNPS, 21,839 MoNA, 8,358 MassBank, 22,282 MS-Dial-VS17, 32,193 FragHub unique chemicals.
- t-SNE parameters match perplexity = 30 and exaggeration = 1 as stated in the methods.
- Each plotted point is colored by superclassNP (or PathwayNP fallback), and the legend distinguishes the constituent libraries with their compound counts.
- The projection shows denser GNPS/MS-Dial coverage in terpenoid/alkaloid regions and broader MoNA coverage for carbohydrates, consistent with the reported finding.

## Limitations

- t-SNE is stochastic and sensitive to hyperparameters; the article fixes perplexity = 30 and exaggeration = 1, so deviating from these parameters makes the visualization non-comparable.
- PubChem fingerprint retrieval depends on the offline PubChem datas bundled with FragHub; compounds absent from PubChem will lack descriptors and be dropped.
- The projection visualizes chemical space but does not harmonize collision energies or spectral redundancy, so it cannot be used to assess spectral-level quality across OMSLs.

## Evidence

- [methods] t-SNE parameters and fingerprint source: "dimensionality reduction was calculated from PubChem fingerprints"
- [results] Per-library compound counts feeding the projection: "combined total of 794,985 spectra for 35,673 unique chemical identifiers. The FragHub data integration workflow refines this further to 602,744 spectra for 32,193 unique chemicals"
- [results] Chemical-space coverage finding motivating the visualization: "GNPS-tagged databases in the .MGF format, comprising 13,507 compounds and 63,935 spectra"
- [results] MoNA chemical-space breadth: "MoNA (MassBank of North America) significantly enriches our data set with 21,839 unique compounds across 190,359 spectra"
- [discussion] Chemical-class disparity underscoring the need for the visualization: "This disparity underscores the necessity for orthogonal strategies to fill this gap"

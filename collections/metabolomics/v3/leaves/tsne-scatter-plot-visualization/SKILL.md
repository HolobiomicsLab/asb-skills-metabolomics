---
name: tsne-scatter-plot-visualization
description: Use when you need to visually compare chemical-space coverage across
  multiple mass spectral libraries using normalized chemical identifiers, PubChem
  fingerprints, and t-SNE dimensionality reduction.
when_to_use_negative:
- The goal is spectral-level comparison (peak intensities, entropy scores, similarity
  scoring) rather than chemical-space (fingerprint-based) coverage — use spectral
  visualization or scoring tools instead.
- The compound set is very small (< ~50 compounds), where t-SNE embeddings are unstable
  and simple PCA or a table would suffice.
- You lack offline PubChem fingerprint data or NPClassifier annotations for the compound
  identifiers; without these, the plot cannot be color-coded or projected correctly.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3172
tools:
- name: FragHub
  role: Produces the standardized library files (e.g., standardize lib) from which
    per-compound normalized InChIKeys are extracted; also supplies offline PubChem
    fingerprint data.
  repo: github.com/eMetaboHUB/FragHub
- name: PubChem
  role: Source of the molecular fingerprints used as t-SNE input descriptors, retrieved
    per InChIKey from offline data.
- name: NPclassifier
  role: Provides PathwayNP and superclassNP annotations used to color-code each compound
    in the t-SNE scatter.
- name: Python
  role: Implementation language for parsing the standardize lib files, running sklearn.manifold.TSNE,
    and rendering the matplotlib scatter.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:cb2f89afc1ca870b84d88dff3615f3811136b982be36ed1e5912fa610515b9d6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - FragHub
  - NPclassifier
  - PubChem
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# tsne-scatter-plot-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Projects PubChem molecular fingerprints of OMSL compounds into a 2-D t-SNE scatter plot to compare the chemical space coverage of GNPS, MoNA, MassBank, MS-Dial and the merged FragHub library, color-coded by NPClassifier pathway/superclass.

## When to use

Use when you need to visually compare chemical-space coverage across multiple mass spectral libraries using normalized chemical identifiers, PubChem fingerprints, and t-SNE dimensionality reduction.

## When NOT to use

- The goal is spectral-level comparison (peak intensities, entropy scores, similarity scoring) rather than chemical-space (fingerprint-based) coverage — use spectral visualization or scoring tools instead.
- The compound set is very small (< ~50 compounds), where t-SNE embeddings are unstable and simple PCA or a table would suffice.
- You lack offline PubChem fingerprint data or NPClassifier annotations for the compound identifiers; without these, the plot cannot be color-coded or projected correctly.

## Inputs

- standardize lib files produced by FragHub for each OMSL (GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial-VS17) with normalized InChIKey identifiers
- Offline PubChem fingerprint data indexed by InChIKey (bundled with FragHub)
- NPClassifier API annotations (PathwayNP and superclassNP) per compound

## Outputs

- tsne_chemical_space.png — a 2-D t-SNE scatter plot of the pooled compound set, color-coded by NPClassifier superclass/pathway and labeled by originating library
- Per-library compound counts used for the legend (e.g., 13,507 GNPS; 21,839 MoNA; 8,358 MassBank; 22,282 MS-Dial-VS17; 32,193 FragHub unique chemicals)

## How to apply

Extract the normalized InChIKey for every compound from the standardized library files produced by FragHub (GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial-VS17). Retrieve the PubChem fingerprint for each InChIKey from the offline PubChem data bundled with FragHub and use the fingerprint block as the molecular descriptor. Color-code each compound with the NPClassifier API, keeping PathwayNP and superclassNP per compound. Run sklearn.manifold.TSNE on the pooled fingerprints with perplexity = 30 and exaggeration = 1, labeling each point by originating library (FragHub vs GNPS vs MoNA vs MassBank vs MS-Dial-VS17). Draw the 2-D scatter with matplotlib, coloring by superclassNP (or PathwayNP fallback), with a legend distinguishing the constituent libraries and their compound counts, and save as tsne_chemical_space.png.

## Related tools

- **FragHub** (Produces the standardized library files (e.g., standardize lib) from which per-compound normalized InChIKeys are extracted; also supplies offline PubChem fingerprint data.) — github.com/eMetaboHUB/FragHub
- **PubChem** (Source of the molecular fingerprints used as t-SNE input descriptors, retrieved per InChIKey from offline data.)
- **NPclassifier** (Provides PathwayNP and superclassNP annotations used to color-code each compound in the t-SNE scatter.)
- **Python** (Implementation language for parsing the standardize lib files, running sklearn.manifold.TSNE, and rendering the matplotlib scatter.)

## Examples

```
from sklearn.manifold import TSNE; import matplotlib.pyplot as plt; emb = TSNE(n_components=2, perplexity=30, early_exaggeration=1).fit_transform(pubchem_fps); plt.scatter(emb[:,0], emb[:,1], c=superclassNP); plt.savefig('tsne_chemical_space.png')
```

## Evaluation signals

- tsne_chemical_space.png is produced and opens as a valid 2-D scatter image.
- The projection covers the compound counts reported per library: 13,507 (GNPS), 21,839 (MoNA), 8,358 (MassBank), 22,282 (MS-Dial-VS17), 32,193 (FragHub unique chemicals).
- The plotted t-SNE parameters match the methods: perplexity = 30 and exaggeration = 1.
- Each point is color-coded by NPClassifier superclassNP (with PathwayNP as fallback) and the legend distinguishes the constituent libraries and their compound counts.
- The t-SNE projection visibly denser regions correspond to terpenoid and alkaloid areas for GNPS/MS-Dial and broader carbohydrate coverage for MoNA, matching the reported observation.

## Limitations

- Chemical-space coverage of the OMSLs is low relative to the diversity of natural products, which are estimated to be several million molecules.
- Chemical class disparity between OMSLs and natural product catalogues means t-SNE chemical-space visualization alone cannot fill coverage gaps; orthogonal strategies are necessary.
- The projection depends on offline PubChem fingerprints and NPClassifier annotations; compounds lacking these identifiers cannot be placed in the plot.
- t-SNE is non-deterministic unless seeds are fixed; the paper's parameters (perplexity = 30, exaggeration = 1) must be matched to reproduce the figure.

## Evidence

---
name: npclassifier-chemical-class-annotation
description: Use when you have standardized InChIKeys for compounds from one or more
  OMSLs (e.g. FragHub 'standardize lib' files) and need chemical-class labels — pathway,
  superclass, class — for class-distribution tables, library comparisons, or coloring
  t-SNE/fingerprint-based chemical-space plots.
when_to_use_negative:
- When compounds lack valid SMILES/InChI (and therefore no resolvable InChIKey) —
  FragHub removes such inconsistent spectra, so NPClassifier cannot be queried
- When the goal is substructure or fingerprint similarity (e.g. t-SNE projection itself)
  rather than class assignment — use PubChem fingerprints for the projection and NPClassifier
  only for coloration
- When collision-energy harmonization or spectral-scoring consistency is the analysis
  target; NPClassifier annotates chemistry classes, not spectral quality or redundancy
edam_operation: http://edamontology.org/operation_3777
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: NPClassifier API
  role: Queries each unique compound InChIKey to return PathwayNP, superclassNP and
    classNP annotations used for library class distributions and t-SNE coloration
- name: FragHub
  role: Produces the standardized compound set (normalized SMILES/InChI/InChIKey)
    and per-library 'standardize lib' files from which InChIKeys are drawn for annotation
  repo: https://github.com/eMetaboHUB/FragHub
- name: RDKit
  role: Standardizes chemical identifiers and calculates molecular masses during FragHub
    preprocessing, ensuring InChIKeys are normalized before NPClassifier query
- name: PubChem (offline data)
  role: Provides molecular fingerprints used for the t-SNE projection that is colored
    by NPClassifier superclass/pathway labels
- name: Python
  role: Scripting language for orchestrating InChIKey extraction, NPClassifier API
    calls, class tabulation and CSV aggregation
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:e8fe8f369c0e919d47e8765d353e3ac40d303abbcba011102b37765193bf26ec
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
  - NPClassifier API
  - NPclassifier
  - PubChem (offline data)
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3777
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# npclassifier-chemical-class-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate every unique compound in a mass spectral library with natural-product chemical classes (PathwayNP, superclassNP, classNP) by querying the NPClassifier service on standardized InChIKeys, so class distributions and t-SNE chemical-space coloration can be computed per OMSL (GNPS, MoNA, MassBank, MS-Dial-VS17).

## When to use

Use when you have standardized InChIKeys for compounds from one or more OMSLs (e.g. FragHub 'standardize lib' files) and need chemical-class labels — pathway, superclass, class — for class-distribution tables, library comparisons, or coloring t-SNE/fingerprint-based chemical-space plots.

## When NOT to use

- When compounds lack valid SMILES/InChI (and therefore no resolvable InChIKey) — FragHub removes such inconsistent spectra, so NPClassifier cannot be queried
- When the goal is substructure or fingerprint similarity (e.g. t-SNE projection itself) rather than class assignment — use PubChem fingerprints for the projection and NPClassifier only for coloration
- When collision-energy harmonization or spectral-scoring consistency is the analysis target; NPClassifier annotates chemistry classes, not spectral quality or redundancy

## Inputs

- Normalized InChIKey identifiers for unique compounds extracted from standardized OMSL library files (FragHub 'standardize lib' outputs from GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial-VS17)
- Compound-to-source-library mapping (InChIKey -> originating OMSL) if per-library class distributions are required
- Optional: compound-to-plot-label mapping (e.g. originating library or compound count) for downstream visualization

## Outputs

- Per-compound NPClassifier annotations (PathwayNP, superclassNP, and optionally classNP) keyed by InChIKey
- Chemical-class distribution table per OMSL: compound and spectrum counts per NPClassifier pathway and superclass
- Class labels (superclassNP with PathwayNP fallback) attached to t-SNE scatter points for the chemical-space figure
- Aggregated CSV with one row per library/chemical-class combination plus per-library median spectra-per-compound summary

## How to apply

First, obtain one normalized InChIKey per unique compound: FragHub recalculates and normalizes SMILES, InChI and InChIKey via RDKit during library integration, and the t-SNE workflow explicitly extracts the normalized InChIKey from each parsed library (GNPS .MGF, MoNA .MSP/.SDF/.JSON, MassBank, MS-Dial-VS17). Then query the NPClassifier API for every unique InChIKey, retaining the PathwayNP and superclassNP fields returned per compound — these are the fields used both in the per-library chemical-class distribution analysis and as the coloration categories for the t-SNE chemical-space figure. When a superclassNP is missing, fall back to PathwayNP for figure coloration. Finally, tabulate compounds (and optionally spectra) per pathway and superclass per library; expect uneven coverage across OMSLs (fatty acids predominant in GNPS and MassBank, alkaloids in GNPS and MS-Dial-VS17, carbohydrates/amino acids/peptides in MoNA), which signals NPClassifier annotations were applied consistently. Validate by checking that every compound intended for class comparison carries PathwayNP/superclassNP before downstream plots are generated.

## Related tools

- **NPClassifier API** (Queries each unique compound InChIKey to return PathwayNP, superclassNP and classNP annotations used for library class distributions and t-SNE coloration)
- **FragHub** (Produces the standardized compound set (normalized SMILES/InChI/InChIKey) and per-library 'standardize lib' files from which InChIKeys are drawn for annotation) — https://github.com/eMetaboHUB/FragHub
- **RDKit** (Standardizes chemical identifiers and calculates molecular masses during FragHub preprocessing, ensuring InChIKeys are normalized before NPClassifier query)
- **PubChem (offline data)** (Provides molecular fingerprints used for the t-SNE projection that is colored by NPClassifier superclass/pathway labels)
- **Python** (Scripting language for orchestrating InChIKey extraction, NPClassifier API calls, class tabulation and CSV aggregation)

## Examples

```
for ink in unique_inchikeys: annotations[ink] = npclassifier_api.annotate(ink)  # returns PathwayNP, superclassNP
```

## Evaluation signals

- Every unique compound InChIKey submitted for class comparison has non-null PathwayNP and superclassNP annotations before plotting
- Chemical-class distribution table reproduces the published uneven composition: fatty acids predominate in GNPS and MassBank, alkaloids in GNPS and MS-Dial-VS17, carbohydrates/amino acids/peptides in MoNA
- Compound counts per library match reported values (GNPS 13,507; MoNA 21,839; MassBank 8,358; MS-Dial-VS17 22,282; FragHub unique chemicals 32,193) after filtering for classifiable InChIKeys
- The t-SNE figure's color legend is driven by superclassNP (with PathwayNP fallback) and correctly distinguishes the constituent libraries and their compound counts
- Output CSV contains one row per library/chemical-class combination with the companion median-spectra-per-compound summary (e.g. GNPS median 2, MassBank median 12)

## Limitations

- NPClassifier coverage depends on the InChIKey resolving in the PubChem-backed knowledge used by FragHub; unresolvable or inconsistent identifiers (no SMILES/InChI, precursor m/z, adduct) are dropped by FragHub preprocessing and cannot be annotated
- The chemical class disparity between OMSLs and the natural-product catalogue (estimated several million molecules) means annotations only reflect what the integrated libraries contain, not the full chemical universe
- FragHub only enriches 'NAMES and descriptor' fields from RDkit and PubChem datas (offline), so annotation quality is bounded by those offline resources; no software version numbers are stated for the tools used
- Chemical classes are assigned at PathwayNP/superclassNP granularity; lower-level classNP may be absent or inconsistent across libraries, limiting fine-grained comparisons

## Evidence

- [methods] Per-compound NPClassifier query feeding class distributions: [evidence span withheld — anchor status: not_found]
- [results] Class distributions per library: [evidence span withheld — anchor status: not_found]
- [methods] t-SNE coloration by NPClassifier labels: [evidence span withheld — anchor status: not_found]

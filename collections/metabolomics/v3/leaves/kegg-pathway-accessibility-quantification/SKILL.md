---
name: kegg-pathway-accessibility-quantification
description: Use when an untargeted metabolomics experiment (e.g. MetaboLights MTBLS136
  split by assay type) needs its assay-specific chemical bias quantified as a count
  or proportion of KEGG human pathways reachable for ORA.
when_to_use_negative:
- The goal is to test ORA robustness to parameters such as background set, differential
  metabolite threshold (q <= 0.05 BH FDR), or pathway database choice (KEGG vs Reactome
  vs BioCyc) — those are separate simulations in the reproducible_simulations.ipynb
  notebook.
- The dataset already has a well-defined, assay-specific background set and the question
  is whether ORA results change, rather than how many pathways the assay can reach.
- The compounds are metabolites with misidentifications to simulate (swapping isobaric
  ±20 ppm or same-formula compounds) — that is a distinct sensitivity experiment,
  not an accessibility census.
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MetaboAnalyst 4.0
  role: Converts assay compound names to KEGG identifiers before pathway mapping
- name: Python 3.8
  role: Implements the assay chemical-bias simulation cells in the Jupyter notebook
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Jupyter Notebook
  role: Runs reproducible_simulations.ipynb to execute the accessibility analysis
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Browser-based alternative for running the same notebook without local installation
  repo: https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing
- name: pip3
  role: Installs requirements.txt dependencies in metabolomics-ORA/src before running
    the notebook
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:1151b2b3e9f3225f841943270250df6fcd7af864cb111949103b4ab2969ecffa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/metabolomics-ORA
  tools:
  - Google Colaboratory
  - Jupyter Notebook
  - MetaboAnalyst
  - MetaboAnalyst 4.0
  - Python
  - Python 3.8
  - pip3
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# kegg-pathway-accessibility-quantification

## Summary

Quantifies how much a metabolomics assay's detectable compound set limits KEGG pathway coverage: convert each assay's identified compounds to KEGG IDs, map them onto the 256 KEGG human pathways, and count how many pathways are accessible (≥2 compounds) per assay.

## When to use

Use when an untargeted metabolomics experiment (e.g. MetaboLights MTBLS136 split by assay type) needs its assay-specific chemical bias quantified as a count or proportion of KEGG human pathways reachable for ORA.

## When NOT to use

- The goal is to test ORA robustness to parameters such as background set, differential metabolite threshold (q <= 0.05 BH FDR), or pathway database choice (KEGG vs Reactome vs BioCyc) — those are separate simulations in the reproducible_simulations.ipynb notebook.
- The dataset already has a well-defined, assay-specific background set and the question is whether ORA results change, rather than how many pathways the assay can reach.
- The compounds are metabolites with misidentifications to simulate (swapping isobaric ±20 ppm or same-formula compounds) — that is a distinct sensitivity experiment, not an accessibility census.

## Inputs

- MetaboLights dataset MTBLS136 compound lists partitioned by assay type (RP/UPLC-MS/MS and HILIC, positive and negative ESI)
- KEGG human pathway sets (256 pathways)
- Compound name → KEGG identifier mappings from MetaboAnalyst 4.0 ID conversion

## Outputs

- accessible_pathways_mtbls136.csv (per-assay count of accessible KEGG human pathways out of 256)
- Cross-assay intersection/overlap tables showing single-assay-exclusive pathways
- Optional iPath3.0-style network coverage map (Figure 6B analog)

## How to apply

Partition the dataset's identified compounds by assay type (e.g. RP/UPLC-MS/MS vs HILIC, under positive or negative ESI), convert compound names to KEGG identifiers with the MetaboAnalyst 4.0 ID conversion tool, then map each assay's KEGG compounds onto KEGG human pathway sets, retaining only pathways containing at least 2 compounds detected by that assay. Count accessible pathways per assay out of the 256 KEGG human pathways and compute intersections across assays to reveal which pathways are reachable only via a single assay. Write per-assay counts to accessible_pathways_mtbls136.csv and optionally render an iPath3.0-style network coverage map. In the Wieder et al. study the maximally inclusive assay, RP/UPLC-MS/MS positive ESI, reached only 63 of 256 pathways (24.6%), the validation anchor for this quantification.

## Related tools

- **MetaboAnalyst 4.0** (Converts assay compound names to KEGG identifiers before pathway mapping)
- **Python 3.8** (Implements the assay chemical-bias simulation cells in the Jupyter notebook) — https://github.com/cwieder/metabolomics-ORA
- **Jupyter Notebook** (Runs reproducible_simulations.ipynb to execute the accessibility analysis) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Browser-based alternative for running the same notebook without local installation) — https://colab.research.google.com/drive/1Dit_EesYxN4z-vV9jBUoDOuWRpZxZoIk?usp=sharing
- **pip3** (Installs requirements.txt dependencies in metabolomics-ORA/src before running the notebook)

## Examples

```
cd metabolomics-ORA/src && pip3 install -r requirements.txt && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- The maximally inclusive assay (RP/UPLC-MS/MS positive ESI) must report exactly 63 of 256 KEGG human pathways accessible, i.e. 24.6%, matching Figure 6B.
- No counted pathway should contain fewer than 2 compounds detected by the corresponding assay.
- Cross-assay intersection counts should show a large share of pathways accessible only via one specific assay.
- Per-assay accessible-pathway totals must sum consistently with the union/intersection relationships across the four ESI/assay combinations.
- Output CSV must be named accessible_pathways_mtbls136.csv and contain per-assay rows over the 256-pathway denominator.

## Limitations

- Only the 256 KEGG human pathways are covered; Reactome and BioCyc pathway sets are not part of this accessibility census.
- Accessibility is capped by the assay's detected compound set — the simulation shows no assay exceeds ~25% of KEGG human pathways, so pathway coverage depends heavily on assay chemistry.
- The analysis relies on compound-name-to-KEGG conversion via MetaboAnalyst, which may drop compounds lacking valid KEGG identifiers, reducing apparent accessibility.
- Results depend on MTBLS136's specific compound identification set; other datasets may yield different per-assay coverage.

## Evidence

- [other] How much does the analytical assay used in a metabolomics experiment bias which metabolic pathways are accessible for over-representation analysis: [evidence span withheld — anchor status: not_found]
- [other] single most inclusive assay reached only 63 of 256 KEGG human pathways (24.6%): [evidence span withheld — anchor status: not_found]
- [discussion] For the conversion of compound names to KEGG identifiers, the MetaboAnalyst 4.0 ID conversion tool was used: "For the conversion of compound names to KEGG identifiers, the MetaboAnalyst 4.0 [47] ID conversion tool was used"
- [other] pathways that contain at least 2 compounds detected by that assay: [evidence span withheld — anchor status: not_found]
- [other] a large share of pathways was accessible only via one specific assay: [evidence span withheld — anchor status: not_found]
- [other] write accessible_pathways_mtbls136.csv: [evidence span withheld — anchor status: not_found]
- [readme] Install the package dependencies with pip3 install -r requirements.txt inside the metabolomics-ORA/src directory: "cd metabolomics-ORA/src
pip3 install -r requirements.txt"
- [readme] Launch the reproducible_simulations.ipynb Jupyter notebook and run the cells: "Launch the reproducible_simulations.ipynb Jupyter notebook and run the code cells"

---
name: pathway-set-intersection-analysis
description: Use when a metabolomics dataset contains multiple analytical assays (e.
when_to_use_negative:
- The dataset comes from a single analytical assay, so no cross-assay chemical-bias
  comparison is possible.
- The goal is to compare pathway databases (KEGG vs Reactome vs BioCyc) rather than
  assay-derived pathway accessibility within one database.
- The objective is quantifying metabolite misidentification effects rather than assay-specific
  pathway coverage.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Runs the assay chemical-bias simulations inside the reproducible_simulations.ipynb
    Jupyter notebook
  repo: https://github.com/cwieder/metabolomics-ORA
- name: MetaboAnalyst 4.0
  role: Converts compound names to KEGG identifiers for each assay's compound list
- name: Jupyter Notebook
  role: Hosts the reproducible_simulations.ipynb cells that partition MTBLS136 by
    assay and compute pathway intersections
  repo: https://github.com/cwieder/metabolomics-ORA
- name: Google Colaboratory
  role: Alternative browser-based execution of the same notebook without local installation
- name: pip3
  role: Installs the metabolomics-ORA/src package dependencies via requirements.txt
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1009105
    title: 'Pathway analysis in metabolomics: Recommendations for the use of over-representation
      analysis'
  source_content_hash: sha256:9e3f92690831891c9d1073c997c98e26a8851388fa39fad8002608f9e83a34bc
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
  - pip3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# pathway-set-intersection-analysis

## Summary

Quantifies the chemical bias of an analytical assay on over-representation analysis by partitioning a metabolomics dataset (e.g. MetaboLights MTBLS136, 667 controls and 332 estrogen users) by assay type, converting each assay's compounds to KEGG IDs via MetaboAnalyst 4.0, and intersecting the pathway sets each assay can reach among the 256 KEGG human pathways.

## When to use

Use when a metabolomics dataset contains multiple analytical assays (e.g. RP/UPLC-MS/MS vs HILIC under positive/negative ESI) and you need to quantify how assay chemistry restricts which KEGG pathways are accessible to ORA, or to explain partial overlap between pathway sets derived from different assays.

## When NOT to use

- The dataset comes from a single analytical assay, so no cross-assay chemical-bias comparison is possible.
- The goal is to compare pathway databases (KEGG vs Reactome vs BioCyc) rather than assay-derived pathway accessibility within one database.
- The objective is quantifying metabolite misidentification effects rather than assay-specific pathway coverage.

## Inputs

- MetaboLights MTBLS136 metabolomics dataset (667 control and 332 estrogen-user samples) with per-assay compound identifications
- Compound-to-KEGG ID mapping from the MetaboAnalyst 4.0 ID conversion tool
- KEGG human pathway definitions (256 pathways)

## Outputs

- Per-assay accessible pathway counts out of 256 KEGG human pathways (e.g. 63/256 = 24.6% for RP/UPLC-MS/MS positive ESI)
- Cross-assay pathway-set intersections showing pathways reachable only via one assay
- accessible_pathways_mtbls136.csv
- Optional iPath3.0-style network coverage map

## How to apply

Partition the identified compounds by assay type (e.g. RP/UPLC-MS/MS and HILIC in positive/negative ESI), then convert compound names to KEGG identifiers using the MetaboAnalyst 4.0 ID conversion tool. Map each assay's KEGG-identified compounds onto KEGG human pathway sets, retaining pathways that contain at least 2 compounds detected by that assay. Count the accessible pathways per assay out of the 256 KEGG human pathways and compute intersections across assays to identify pathways accessible only via a single assay. Optionally render an iPath3.0-style network coverage map, and write the per-assay accessible-pathway table (accessible_pathways_mtbls136.csv). The rationale is that a non-assay-specific background set inflates false positives, so knowing which pathways each assay can actually observe is a prerequisite for choosing the correct background.

## Related tools

- **Python** (Runs the assay chemical-bias simulations inside the reproducible_simulations.ipynb Jupyter notebook) — https://github.com/cwieder/metabolomics-ORA
- **MetaboAnalyst 4.0** (Converts compound names to KEGG identifiers for each assay's compound list)
- **Jupyter Notebook** (Hosts the reproducible_simulations.ipynb cells that partition MTBLS136 by assay and compute pathway intersections) — https://github.com/cwieder/metabolomics-ORA
- **Google Colaboratory** (Alternative browser-based execution of the same notebook without local installation)
- **pip3** (Installs the metabolomics-ORA/src package dependencies via requirements.txt)

## Examples

```
cd metabolomics-ORA/src && pip3 install -r requirements.txt && jupyter-notebook reproducible_simulations.ipynb
```

## Evaluation signals

- The maximally inclusive assay (RP/UPLC-MS/MS positive ESI) reaches exactly 63 of 256 KEGG human pathways (24.6%), matching Figure 6B.
- Each assay's accessible pathway set is built only from pathways containing at least 2 detected compounds for that assay.
- Per-assay counts sum consistently across assays when intersected, and single-assay-only pathways are identifiable.
- Output CSV accessible_pathways_mtbls136.csv is produced with one row per assay or pathway as specified.
- All compound-to-KEGG conversions trace back to the MetaboAnalyst 4.0 ID conversion tool output.

## Limitations

- The study is limited by the lack of availability of a ground-truth dataset where the identities of enriched pathways are known.
- Defining the ideal assay-specific background set remains an area for further study.
- Simulations were performed using Python v3.8 on MacOS v11.2.3 with 16GB RAM; other environments are untested.

## Evidence

- [readme] Assay chemical bias analysis: "This repository contains the code to run the simulations presented in the study. The Python code to generate the results is contained within the Jupyter notebook src/reproducible_simulations.ipynb"
- [discussion] Assay partitioning and compound conversion: "For the conversion of compound names to KEGG identifiers, the MetaboAnalyst 4.0 [47] ID conversion tool was used"
- [abstract] Assay chemical bias finding: "The use of a non-assay-specific background set, for example, resulted in large numbers of false-positive pathways."
- [discussion] Assay chemical bias limitations: "Defining the ideal assay-specific background set for a particular dataset remains an area for further study."

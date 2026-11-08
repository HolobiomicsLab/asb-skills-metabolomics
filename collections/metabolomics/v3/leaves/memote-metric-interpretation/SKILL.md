---
name: memote-metric-interpretation
description: Use when a genome-scale metabolic model (e.g. an SBML GEM from BioModels)
  must be quality-checked against community standards, with metrics like GPR coverage,
  metabolite InChI/annotation completeness, and stoichiometric consistency.
when_to_use_negative:
- When the goal is functional prediction (growth rates, drug targets, bottleneck reactions)
  rather than model quality scoring — use FBA/FVA/findCPcli workflows instead
- When the input is not a genome-scale metabolic model (e.g. it is an interaction
  network, PPI dataset, or gene expression table)
- When a non-SBML model format cannot be converted, since MEMOTE reads SBML via COBRApy
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
tools:
- name: MEMOTE
  role: Standardised genome-scale metabolic model testing software used to run the
    quality-control test battery and generate the scored report
- name: COBRApy
  role: Python toolbox through which MEMOTE reads the SBML model; version 0.16.0 reported
    for instantiating iHsaEC21 without error
  repo: https://github.com/opencobra/cobrapy
- name: Python
  role: Language environment in which COBRApy-based model loading and MEMOTE execution
    are performed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:b5ef0485c9c0c8a970572d917a23f3d0686af5f5cfeebb9fe754bb6a1096a2d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
  tools:
  - COBRApy
  - MEMOTE
  - Python
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# memote-metric-interpretation

## Summary

Runs the MEMOTE standardised test battery on a genome-scale metabolic model (SBML, read via COBRApy) and interprets the per-metric breakdown to judge whether the model meets community GEM standards, as done for the iHsaEC21_SARS-CoV-2 model from BioModels MODEL2007210001.

## When to use

Use when a genome-scale metabolic model (e.g. an SBML GEM from BioModels) must be quality-checked against community standards, with metrics like GPR coverage, metabolite InChI/annotation completeness, and stoichiometric consistency.

## When NOT to use

- When the goal is functional prediction (growth rates, drug targets, bottleneck reactions) rather than model quality scoring — use FBA/FVA/findCPcli workflows instead
- When the input is not a genome-scale metabolic model (e.g. it is an interaction network, PPI dataset, or gene expression table)
- When a non-SBML model format cannot be converted, since MEMOTE reads SBML via COBRApy

## Inputs

- Genome-scale metabolic model in SBML format (e.g. iHsaEC21 from BioModels accession MODEL2007210001)
- Optional model metadata (name, version, creators) embedded in the SBML file

## Outputs

- memote_report.html with overall MEMOTE score and per-metric breakdowns
- Per-metric values for GPR coverage (e.g. 92.5%), metabolite InChI-key coverage (100%), database annotation consistency, stoichiometric consistency, and model metadata

## How to apply

Download the GEM in SBML format (for iHsaEC21, from BioModels accession MODEL2007210001) and load it into MEMOTE, which reads the model through COBRApy. Execute the MEMOTE test battery covering annotation consistency (metabolite InChI keys and cross-references to BiGG, MetaNetX, KEGG, ChEBI, ModelSEED, HMDB, MetaCyc), gene-protein-reaction association coverage, reaction and metabolite annotation, stoichiometric consistency, and model metadata. Extract both the overall MEMOTE score and per-metric values; for iHsaEC21 the reference targets are an overall score of ~51%, ~92.5% of reactions with GPRs, and 100% of metabolites with InChI keys and database annotations. Save the complete results, including the overall score and per-metric breakdowns, to memote_report.html and validate against values reported in Supplemental Data 1. If metrics fall short, refinements can be made using cross-references from Recon3D, HumanCyc, MetaNetX 4.2, BiGG, ChEBI, MetaCyc, and PubChem databases.

## Related tools

- **MEMOTE** (Standardised genome-scale metabolic model testing software used to run the quality-control test battery and generate the scored report)
- **COBRApy** (Python toolbox through which MEMOTE reads the SBML model; version 0.16.0 reported for instantiating iHsaEC21 without error) — https://github.com/opencobra/cobrapy
- **Python** (Language environment in which COBRApy-based model loading and MEMOTE execution are performed)

## Examples

```
memote report snapshot iHsaEC21.xml --filename memote_report.html
```

## Evaluation signals

- Overall MEMOTE score for iHsaEC21 is approximately 51%, matching the article's Supplemental Data 1
- Reaction GPR coverage is approximately 92.5%
- Metabolite annotation coverage is 100%: all metabolites carry descriptive names, InChI keys, charge/formula, and conforming database annotations
- memote_report.html is generated and contains both the overall score and per-metric breakdowns
- The SBML model loads without error through COBRApy (version 0.16.0 reported)

## Limitations

- No MEMOTE version number was reported for the quality scoring in the article
- A passing subset of metrics (e.g. 100% metabolite annotation) does not imply a high overall score — iHsaEC21 scored 51% overall despite perfect metabolite annotation, so per-metric interpretation is required
- Quality metrics assess annotation and consistency, not biological validity of the model's predictions

## Evidence

- [discussion] MEMOTE quality score reported for iHsaEC21: "iHsaEC21 now has a MEMOTE quality score of 51% (Supplemental Data 1)"
- [methods] MEMOTE used for standardised quality control and model comparison: "we performed quality control checks on MEMOTE, a standardised genome-scale metabolic model testing software (Leiven et al, 2020)"
- [intro] Model checking and refinement via database cross-references: "Model checks were done with MEMOTE, and refinements were performed with MetaNetX 4.2, BiGG, ChEBI, MetaCyc, and PubChem databases"
- [methods] Model loads without error on COBRApy: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"

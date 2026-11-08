---
name: sbml-model-validation
description: Use when an SBML genome-scale metabolic model (e.g. from BioModels) must
  be assessed against community standards for annotation completeness, GPR coverage,
  and stoichiometric consistency before publication or reuse.
when_to_use_negative:
- The model is not in SBML format or is not a constraint-based metabolic model (MEMOTE's
  test battery targets GEM content)
- The goal is functional simulation (e.g. FBA/FVA growth or chokepoint analysis) rather
  than structural/annotation quality assessment
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: MEMOTE
  role: Standardised genome-scale metabolic model testing software used to run the
    quality-control test battery and generate the quality report
- name: COBRApy
  role: Python toolbox used by MEMOTE to read the SBML model; also used (v0.16.0)
    to instantiate and manipulate iHsaEC21
  repo: https://github.com/opencobra/cobrapy
- name: Python
  role: Language environment in which COBRApy and MEMOTE are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:e5c52a534ca7e51950f3ece785c764d52f77e1dd5dd50be3f473928bde0273b7
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
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sbml-model-validation

## Summary

Standardised quality-control of a genome-scale metabolic model in SBML format using MEMOTE (which reads the model via COBRApy), covering annotation consistency, GPR coverage, stoichiometric consistency, and model metadata, benchmarked against community standards.

## When to use

Use when an SBML genome-scale metabolic model (e.g. from BioModels) must be assessed against community standards for annotation completeness, GPR coverage, and stoichiometric consistency before publication or reuse.

## When NOT to use

- The model is not in SBML format or is not a constraint-based metabolic model (MEMOTE's test battery targets GEM content)
- The goal is functional simulation (e.g. FBA/FVA growth or chokepoint analysis) rather than structural/annotation quality assessment

## Inputs

- Genome-scale metabolic model in SBML format (e.g. iHsaEC21 from BioModels accession MODEL2007210001)
- Community-standard reference metric values from prior reports (e.g. Supplemental Data 1)

## Outputs

- memote_report.html containing the overall MEMOTE score and per-metric breakdowns (annotation coverage, GPR coverage, stoichiometric consistency, metadata)
- Per-metric percentages (e.g. 92.5% reactions with GPRs, 100% metabolites with InChI keys)

## How to apply

Download the SBML model (here iHsaEC21 from BioModels accession MODEL2007210001) and load it into MEMOTE, which parses it via COBRApy. Run the MEMOTE test battery, which checks metabolite InChI keys, charge and formula presence, and cross-reference annotations to BiGG, MetaNetX, KEGG, ChEBI, ModelSEED, HMDB, and MetaCyc; gene-protein-reaction association coverage; stoichiometric consistency; and model metadata. Refine annotation gaps using database cross-references (e.g. MetaNetX 4.2, BiGG, ChEBI, MetaCyc, PubChem, HumanCyc, Recon3D) before re-scoring. Extract the overall score and per-metric breakdowns from the generated report and compare them against published values. For this paper's iHsaEC21 model, the accepted reference values are an overall MEMOTE score of 51%, 92.5% of reactions with GPRs, and 100% of metabolites with descriptive names, InChI keys, charge/formula, and conforming database annotations (Supplemental Data 1). Save the full test results to memote_report.html.

## Related tools

- **MEMOTE** (Standardised genome-scale metabolic model testing software used to run the quality-control test battery and generate the quality report)
- **COBRApy** (Python toolbox used by MEMOTE to read the SBML model; also used (v0.16.0) to instantiate and manipulate iHsaEC21) — https://github.com/opencobra/cobrapy
- **Python** (Language environment in which COBRApy and MEMOTE are executed)

## Examples

```
memote report snapshot --model iHsaEC21.xml --filename memote_report.html
```

## Evaluation signals

- Overall MEMOTE score is approximately 51% for iHsaEC21, matching Supplemental Data 1
- Approximately 92.5% of reactions have gene-protein-reaction associations
- 100% of metabolites carry descriptive names, InChI keys, charge and formula, and conforming database annotations
- Model instantiates without error in COBRApy (version 0.16.0 reported for iHsaEC21)
- memote_report.html is generated with complete per-metric breakdowns

## Limitations

- The article reports no version number for the MEMOTE release used, so exact reproducibility of the 51% score across MEMOTE versions is not guaranteed
- A 51% overall score indicates annotation/consistency gaps remain even though metabolite-level annotation coverage is 100%; the score depends on the full test battery, not just metabolite annotations

## Evidence

- [intro] MEMOTE used for standardised QC: "used the MEMOTE quality control software to assess and compare our model with previously developed GEMS"
- [discussion] 51% MEMOTE score: "iHsaEC21 now has a MEMOTE quality score of 51% (Supplemental Data 1)"
- [intro] Refinement databases: "Model checks were done with MEMOTE, and refinements were performed with MetaNetX 4.2, BiGG, ChEBI, MetaCyc, and PubChem databases"
- [methods] COBRApy instantiation check: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
- [methods] MEMOTE described as standardised testing software: "we performed quality control checks on MEMOTE, a standardised genome-scale metabolic model testing software (Leiven et al, 2020)"

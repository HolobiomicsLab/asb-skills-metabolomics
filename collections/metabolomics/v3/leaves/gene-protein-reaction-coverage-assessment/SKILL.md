---
name: gene-protein-reaction-coverage-assessment
description: Use when a genome-scale metabolic model (e.g. an SBML file such as BioModels
  MODEL2007210001) must be quality-controlled before or after publication, and GPR
  coverage, metabolite annotation, or an overall MEMOTE score are needed as evidence
  of community-standard compliance.
when_to_use_negative:
- The model is not a genome-scale metabolic model in SBML format (e.g. a kinetic model,
  PPI network, or expression matrix) — MEMOTE's test battery will not apply.
- The goal is drug-target identification or bottleneck analysis rather than annotation
  quality assessment — findCPcli's chokepoint/dead-end-metabolite workflow, not MEMOTE,
  is the appropriate tool.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MEMOTE
  role: Runs the standardised test battery (GPR coverage, annotation consistency,
    stoichiometric consistency, metadata) and generates the HTML quality report
- name: COBRApy
  role: Reads the SBML model into MEMOTE and manipulates/simulates it (version 0.16.0,
    with GLPK)
  repo: https://github.com/opencobra/cobrapy
- name: Python
  role: Execution environment for COBRApy/MEMOTE quality-control workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:0c8386a49ba4cf3e9caaddb293bce9ac0c282bb2ccf487ac726ce752a18484a9
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Gene-Protein-Reaction coverage assessment

## Summary

Quantifies the fraction of reactions in a genome-scale metabolic model that carry Gene-Protein-Reaction (GPR) associations, using MEMOTE's standardised test battery so that annotation completeness can be compared against community standards for published GEMs.

## When to use

Use when a genome-scale metabolic model (e.g. an SBML file such as BioModels MODEL2007210001) must be quality-controlled before or after publication, and GPR coverage, metabolite annotation, or an overall MEMOTE score are needed as evidence of community-standard compliance.

## When NOT to use

- The model is not a genome-scale metabolic model in SBML format (e.g. a kinetic model, PPI network, or expression matrix) — MEMOTE's test battery will not apply.
- The goal is drug-target identification or bottleneck analysis rather than annotation quality assessment — findCPcli's chokepoint/dead-end-metabolite workflow, not MEMOTE, is the appropriate tool.

## Inputs

- Genome-scale metabolic model in SBML format (e.g. BioModels MODEL2007210001, the iHsaEC21 model)
- Model metadata and annotation cross-references to BiGG, MetaNetX, KEGG, ChEBI, ModelSEED, HMDB, MetaCyc

## Outputs

- memote_report.html with the overall MEMOTE score and per-metric breakdowns
- GPR coverage percentage (target: 92.5% of reactions with GPRs)
- Metabolite annotation coverage metrics (target: 100% with names, InChI keys, charge/formula, and database annotations)

## How to apply

Download the SBML model (iHsaEC21 from BioModels accession MODEL2007210001) and load it into MEMOTE, which reads it via COBRApy. Run the MEMOTE test battery covering annotation consistency (InChI keys and cross-references to BiGG, MetaNetX, KEGG, ChEBI, ModelSEED, HMDB, MetaCyc), GPR coverage, reaction/metabolite annotation, stoichiometric consistency, and model metadata. Extract the overall score and per-metric values from the generated report, saving the full breakdown to memote_report.html. For the source paper's iHsaEC21 model, the target values are an overall MEMOTE score of ~51%, GPR coverage of ~92.5% of reactions, and 100% of metabolites with InChI keys and conforming database annotations; the article notes the model instantiates without error on COBRA (version 0.16.0) with GLPK. Where GPR or annotation gaps are found, refinements can be guided by database cross-references from Recon3D, HumanCyc, MetaNetX 4.2, BiGG, ChEBI, MetaCyc, and PubChem.

## Related tools

- **MEMOTE** (Runs the standardised test battery (GPR coverage, annotation consistency, stoichiometric consistency, metadata) and generates the HTML quality report)
- **COBRApy** (Reads the SBML model into MEMOTE and manipulates/simulates it (version 0.16.0, with GLPK)) — https://github.com/opencobra/cobrapy
- **Python** (Execution environment for COBRApy/MEMOTE quality-control workflow)

## Examples

```
memote report snapshot iHsaEC21.sbml --outfile memote_report.html
```

## Evaluation signals

- Overall MEMOTE score of the iHsaEC21 model is approximately 51%, matching Supplemental Data 1
- Approximately 92.5% of reactions carry Gene-Protein-Reaction associations
- 100% of metabolites have descriptive names, InChI keys, charge/formula, and conforming database annotations
- The SBML model loads without error into MEMOTE via COBRApy (COBRA 0.16.0)
- memote_report.html contains both the overall score and the per-metric breakdowns

## Limitations

- No version number of MEMOTE is reported in the article, so exact metric implementations may differ across MEMOTE releases.
- A 100% metabolite annotation score and 92.5% GPR coverage still yield an overall score of only 51%, so individual metrics should be inspected rather than relying on the aggregate score.
- Metric values are specific to iHsaEC21 and its refinement databases (Recon3D, HumanCyc, MetaNetX 4.2, BiGG, ChEBI, MetaCyc, PubChem); other models will differ.

## Evidence

- [discussion] iHsaEC21 now has a MEMOTE quality score of 51%: "iHsaEC21 now has a MEMOTE quality score of 51% (Supplemental Data 1)"
- [methods] MEMOTE used for standardised GEM testing: "we performed quality control checks on MEMOTE, a standardised genome-scale metabolic model testing software (Leiven et al, 2020)"
- [intro] Refinements with annotation databases: "Model checks were done with MEMOTE, and refinements were performed with MetaNetX 4.2, BiGG, ChEBI, MetaCyc, and PubChem databases"
- [methods] Model instantiates without error on COBRA: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"

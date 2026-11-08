---
name: metabolite-annotation-verification
description: Use when a genome-scale metabolic model (e.g. an SBML file from BioModels)
  must be assessed against community standards for metabolite/reaction annotation
  coverage and GPR completeness before publication or reuse.
when_to_use_negative:
- The task is metabolic function analysis (e.g. chokepoint, bottleneck, or FBA computation
  with findCPcli) rather than annotation/quality assessment
- The model is not in a MEMOTE-readable format (e.g. no SBML file loadable via COBRApy)
- Only the annotated metabolite list (not the full model structure with GPRs and stoichiometry)
  is available, since MEMOTE scores require the complete model
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3172
tools:
- name: MEMOTE
  role: Executes the standardised quality-control test battery that scores metabolite
    annotation consistency (InChI keys, BiGG/MetaNetX/KEGG/ChEBI/ModelSEED/HMDB/MetaCyc
    cross-references) and GPR coverage
- name: COBRApy 0.16.0
  role: Reads the SBML model so MEMOTE can test it; also used to instantiate and manipulate
    the model without error
  repo: https://github.com/opencobra/cobrapy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:697373ef4fce2b3080172d9487821365ba373b771de1d39186ef39bf6a42ae0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
  tools:
  - COBRApy
  - COBRApy 0.16.0
  - MEMOTE
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolite-annotation-verification

## Summary

Quality-check the annotation completeness and consistency of a genome-scale metabolic model (SBML) using the MEMOTE test battery via COBRApy, verifying that metabolites carry names, InChI keys, charge/formula, and cross-references to standard databases such as BiGG, MetaNetX, KEGG, ChEBI, ModelSEED, HMDB, and MetaCyc.

## When to use

Use when a genome-scale metabolic model (e.g. an SBML file from BioModels) must be assessed against community standards for metabolite/reaction annotation coverage and GPR completeness before publication or reuse.

## When NOT to use

- The task is metabolic function analysis (e.g. chokepoint, bottleneck, or FBA computation with findCPcli) rather than annotation/quality assessment
- The model is not in a MEMOTE-readable format (e.g. no SBML file loadable via COBRApy)
- Only the annotated metabolite list (not the full model structure with GPRs and stoichiometry) is available, since MEMOTE scores require the complete model

## Inputs

- Genome-scale metabolic model in SBML format (e.g. iHsaEC21, BioModels MODEL2007210001)
- Reference annotation databases for refinement: MetaNetX 4.2, BiGG, ChEBI, MetaCyc, PubChem

## Outputs

- MEMOTE HTML report (memote_report.html) with overall score and per-metric breakdowns
- Per-metric coverage values for metabolite InChI keys, database annotations, charge/formula, and reaction GPR coverage

## How to apply

Download the model in SBML format (e.g. iHsaEC21 from BioModels accession MODEL2007210001) and load it into MEMOTE, which reads it via COBRApy. Run the MEMOTE test battery covering annotation consistency (metabolite InChI keys and cross-references to BiGG, MetaNetX, KEGG, ChEBI, ModelSEED, HMDB, MetaCyc), gene-protein-reaction association coverage, stoichiometric consistency, and model metadata. Extract the overall MEMOTE score and per-metric values from the generated HTML report, and refine the model against MetaNetX 4.2, BiGG, ChEBI, MetaCyc, and PubChem databases where annotations are missing or inconsistent. Accept the run when the overall score and per-metric coverages match reference values (for iHsaEC21: overall 51%, 92.5% of reactions with GPRs, 100% of metabolites with InChI keys and database annotations).

## Related tools

- **MEMOTE** (Executes the standardised quality-control test battery that scores metabolite annotation consistency (InChI keys, BiGG/MetaNetX/KEGG/ChEBI/ModelSEED/HMDB/MetaCyc cross-references) and GPR coverage)
- **COBRApy 0.16.0** (Reads the SBML model so MEMOTE can test it; also used to instantiate and manipulate the model without error) — https://github.com/opencobra/cobrapy

## Examples

```
memote report snapshot iHsaEC21.xml --output memote_report.html
```

## Evaluation signals

- Overall MEMOTE score approximates the reported reference value (51% for iHsaEC21)
- 92.5% of reactions have gene-protein-reaction associations
- 100% of metabolites carry InChI keys, charge/formula, and conforming database annotations
- Model instantiation in COBRA 0.16.0 completes without error
- Values match the article's Supplemental Data 1 per-metric breakdowns

## Limitations

- The article does not report which MEMOTE version was used for the quality scoring, so exact score reproduction across MEMOTE releases may drift
- A low overall score (e.g. 51%) can coexist with perfect metabolite annotation coverage because the composite score weights reaction/GPR and stoichiometric metrics too
- Gaps in GPR coverage (7.5% of reactions) are not resolved by annotation refinement alone

## Evidence

- [intro] Model checks were done with MEMOTE, and refinements were performed with MetaNetX 4.2, BiGG, ChEBI, MetaCyc, and PubChem databases: "Model checks were done with MEMOTE, and refinements were performed with MetaNetX 4.2, BiGG, ChEBI, MetaCyc, and PubChem databases"
- [discussion] MEMOTE quality score 51%: "iHsaEC21 now has a MEMOTE quality score of 51% (Supplemental Data 1)"
- [methods] MEMOTE as standardised testing software: "we performed quality control checks on MEMOTE, a standardised genome-scale metabolic model testing software (Leiven et al, 2020)"

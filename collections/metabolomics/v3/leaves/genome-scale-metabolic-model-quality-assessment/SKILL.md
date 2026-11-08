---
name: genome-scale-metabolic-model-quality-assessment
description: Use when a newly constructed or revised genome-scale metabolic model
  (SBML, e.g. from BioModels) must be checked against community standards such as
  MEMOTE before publication or reuse in downstream analyses.
when_to_use_negative:
- The goal is functional simulation (e.g. FBA growth rates, chokepoint prediction
  with findCPcli) rather than structural/annotation quality scoring of the model.
- The model is not in SBML or cannot be instantiated by COBRApy without errors, so
  the MEMOTE battery cannot be run meaningfully.
- Evaluating context-specificity or gene-expression integration — MEMOTE assesses
  annotation and consistency, not biological context fit.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: MEMOTE
  role: Runs the standardised genome-scale metabolic model test battery (annotation
    consistency, GPR coverage, stoichiometric consistency, metadata) and produces
    the scored report
- name: COBRApy
  role: Reads the SBML model into memory so MEMOTE can test it; version 0.16.0 is
    reported to instantiate iHsaEC21 without error
  repo: https://github.com/opencobra/cobrapy
- name: Python
  role: Execution environment for the COBRApy/MEMOTE workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:676370539602f53858c7b26d38af4098bd33cd8015afa83d7799d9489874359d
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
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# genome-scale-metabolic-model-quality-assessment

## Summary

Standardised quality control of a genome-scale metabolic model (GEM) using MEMOTE, which reads the SBML model via COBRApy and scores annotation consistency, gene-protein-reaction coverage, stoichiometric consistency, and metadata. Applied here to the iHsaEC21_SARS-CoV-2 model to verify it meets community standards for a newly published GEM.

## When to use

Use when a newly constructed or revised genome-scale metabolic model (SBML, e.g. from BioModels) must be checked against community standards such as MEMOTE before publication or reuse in downstream analyses.

## When NOT to use

- The goal is functional simulation (e.g. FBA growth rates, chokepoint prediction with findCPcli) rather than structural/annotation quality scoring of the model.
- The model is not in SBML or cannot be instantiated by COBRApy without errors, so the MEMOTE battery cannot be run meaningfully.
- Evaluating context-specificity or gene-expression integration — MEMOTE assesses annotation and consistency, not biological context fit.

## Inputs

- Genome-scale metabolic model in SBML format (e.g. BioModels MODEL2007210001, iHsaEC21_SARS-CoV-2)
- Database cross-reference sources for annotation refinement (MetaNetX 4.2, BiGG, ChEBI, MetaCyc, PubChem)

## Outputs

- memote_report.html containing the overall MEMOTE score and per-metric breakdowns
- Per-metric values: % reactions with GPRs, % metabolites with InChI keys, % metabolites with conforming database annotations, stoichiometric consistency and metadata scores

## How to apply

Download the GEM in SBML format from its repository (here BioModels accession MODEL2007210001 for iHsaEC21) and load it into MEMOTE, which reads the model through COBRApy. Execute the MEMOTE test battery covering metabolite annotation consistency (names, InChI keys, charge/formula, and cross-references to BiGG, MetaNetX, KEGG, ChEBI, ModelSEED, HMDB, MetaCyc), gene-protein-reaction association coverage, reaction and metabolite annotation, stoichiometric consistency, and model metadata. Where metrics fall short, refine the model using database cross-references (e.g. MetaNetX 4.2, BiGG, ChEBI, MetaCyc, PubChem) and re-run the tests iteratively. Extract the overall MEMOTE score and per-metric breakdowns from the generated report; for iHsaEC21 the target values were an overall score of 51%, 92.5% of reactions with GPRs, and 100% of metabolites with InChI keys and database annotations. Save the complete results to memote_report.html for provenance and comparison against published Supplemental Data.

## Related tools

- **MEMOTE** (Runs the standardised genome-scale metabolic model test battery (annotation consistency, GPR coverage, stoichiometric consistency, metadata) and produces the scored report)
- **COBRApy** (Reads the SBML model into memory so MEMOTE can test it; version 0.16.0 is reported to instantiate iHsaEC21 without error) — https://github.com/opencobra/cobrapy
- **Python** (Execution environment for the COBRApy/MEMOTE workflow)

## Examples

```
memote report snapshot iHsaEC21.sbml --filename memote_report.html
```

## Evaluation signals

- Overall MEMOTE score for iHsaEC21 is approximately 51%, matching Supplemental Data 1.
- Reaction GPR coverage is approximately 92.5%.
- Metabolite annotation coverage is 100% — every metabolite carries a descriptive name, InChI key, charge/formula, and conforming database annotations (BiGG, MetaNetX, KEGG, ChEBI, ModelSEED, HMDB, MetaCyc).
- The SBML model loads without error in COBRApy (version 0.16.0 reported).
- A complete memote_report.html is produced with both overall and per-metric breakdowns retained.

## Limitations

- The article does not report the MEMOTE version used, so exact reproduction of per-metric scores may vary across MEMOTE releases.
- Annotation completeness (100% metabolites annotated) does not guarantee functional correctness — stoichiometric consistency and other metric categories drive the lower overall 51% score.
- Refinements rely on external databases (Recon3D, HumanCyc, MetaNetX 4.2, BiGG, ChEBI, MetaCyc, PubChem); database version drift can change cross-reference matches over time.

## Evidence

- [intro] MEMOTE quality control scope: "used the MEMOTE quality control software to assess and compare our model with previously developed GEMS"
- [discussion] Overall score: "iHsaEC21 now has a MEMOTE quality score of 51% (Supplemental Data 1)"
- [intro] Refinement databases: "Model checks were done with MEMOTE, and refinements were performed with MetaNetX 4.2, BiGG, ChEBI, MetaCyc, and PubChem databases"
- [methods] COBRApy instantiation: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
- [methods] Standardised testing tool: "we performed quality control checks on MEMOTE, a standardised genome-scale metabolic model testing software (Leiven et al, 2020)"

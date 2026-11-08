---
name: sbml-model-export
description: Use when a Reconstructor reconstruction (from FASTA, BLASTp hits, or
  an existing SBML) is complete or gap-filled and needs to be written out as a portable
  SBML GENRE for benchmarking or downstream COBRApy analysis.
when_to_use_negative:
- The input is raw sequence data needing annotation or BLASTp alignment first — SBML
  export assumes a draft or completed COBRApy model
- The target pipeline requires formats other than SBML or non-ModelSEED identifier
  namespaces, since Reconstructor's naming follows the ModelSEED biochemistry database
- You only need gap-filling statistics without a portable exported model file
edam_operation: http://edamontology.org/operation_3431
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: Reconstructor
  role: Generates and gap-fills the GENRE and writes the final model to SBML with
    ModelSEED-consistent naming
  repo: http://github.com/emmamglass/reconstructor
- name: COBRApy
  role: In-memory model representation and SBML read/write used to load, configure
    media, and export the model
- name: MEMOTE
  role: Scores the exported SBML file to confirm export quality (expected overall
    scores 83%-85% for Reconstructor models)
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:549a43d448b91c5be2ec40bdc995fbddf011ee4a69997fc7a861d660f05452a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COBRApy
  - MEMOTE
  - Python
  - Reconstructor
  - pip
  edam_operation: http://edamontology.org/operation_3431
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sbml-model-export

## Summary

Export Reconstructor-built genome-scale metabolic network reconstructions (GENREs) as SBML files with reaction and metabolite naming consistent with the ModelSEED biochemistry database, so models are portable into COBRApy, MEMOTE, and other COBRA tooling.

## When to use

Use when a Reconstructor reconstruction (from FASTA, BLASTp hits, or an existing SBML) is complete or gap-filled and needs to be written out as a portable SBML GENRE for benchmarking or downstream COBRApy analysis.

## When NOT to use

- The input is raw sequence data needing annotation or BLASTp alignment first — SBML export assumes a draft or completed COBRApy model
- The target pipeline requires formats other than SBML or non-ModelSEED identifier namespaces, since Reconstructor's naming follows the ModelSEED biochemistry database
- You only need gap-filling statistics without a portable exported model file

## Inputs

- Annotated amino acid .fasta file (Type 1 input)
- DIAMOND BLASTp output hits file (Type 2 input)
- Existing SBML GENRE (.sbml) for further gap-filling (Type 3 input)
- universal.pickle universal reaction database (ModelSEED-based, COBRApy Model)
- Media definition ('rich' or comma-separated ModelSEED-namespace metabolite list ending in _e)

## Outputs

- SBML GENRE file (e.g., gapfilled_model.sbml) with ModelSEED-consistent reaction and metabolite naming
- Gap-filling statistics CSV (gapfill_stats.csv: reactions added, biomass objective flux)

## How to apply

Run Reconstructor on the chosen input (Type 1 annotated amino acid .fasta, Type 2 DIAMOND BLASTp output, or Type 3 existing .sbml for additional gap-filling), letting the tool produce a COBRApy-compatible model whose reaction and metabolite identifiers follow ModelSEED conventions (e.g., cpd00001_e style metabolite IDs). Optionally set the --out and --name arguments to control the output GENRE filename and model ID rather than the defaults. After the parsimony-based (pFBA) gap-filling step restores biomass production under the specified media condition (default 'rich', or a comma-separated list of ModelSEED-namespace extracellular metabolites ending in _e), export the completed model to SBML (e.g., gapfilled_model.sbml). Validate the export by loading it back with COBRApy and scoring with MEMOTE; Reconstructor-generated models scored 83%-85% overall in benchmarking across 10 bacterial species.

## Related tools

- **Reconstructor** (Generates and gap-fills the GENRE and writes the final model to SBML with ModelSEED-consistent naming) — http://github.com/emmamglass/reconstructor
- **COBRApy** (In-memory model representation and SBML read/write used to load, configure media, and export the model)
- **MEMOTE** (Scores the exported SBML file to confirm export quality (expected overall scores 83%-85% for Reconstructor models))

## Examples

```
python -m reconstructor --input_file Osplanchnicus.aa.fasta --file_type 1 --gram negative --media rich --out gapfilled_model.sbml
```

## Evaluation signals

- The exported .sbml loads in COBRApy without errors and carries a positive biomass flux under the specified media conditions
- Reaction and metabolite identifiers in the SBML match ModelSEED namespace conventions (e.g., cpd IDs ending in _e for extracellular metabolites)
- MEMOTE overall score of the exported SBML falls in the 83%-85% range reported for Reconstructor models
- Reaction count in gapfill_stats.csv is consistent with the number of reactions present in the exported model
- Output filename and model ID match the --out and --name arguments if they were set (otherwise defaults)

## Limitations

- Identifier namespace is fixed to ModelSEED conventions, which may require mapping for pipelines expecting BiGG or KEGG identifiers
- The original emmamglass/reconstructor repository is no longer updated; maintenance has moved to csbl/reconstructor
- Type 3 usage on an existing SBML only performs additional gap-filling; it does not rebuild the model from sequence data

## Evidence

- [abstract] Reconstructor can generate SBML GENREs from three input types: annotated protein .fasta sequences (Type 1 input), a BLASTp output (Type 2), or an existing SBML GENRE that can be further gap-filled: "Reconstructor can generate SBML GENREs from three input types: annotated protein .fasta sequences (Type 1 input), a BLASTp output (Type 2), or an existing SBML GENRE"
- [abstract] ModelSEED-consistent naming in exports: "reaction and metabolite naming conventions that are consistent with the ModelSEED biochemistry database"
- [readme] CLI invocation with output control: [evidence span withheld — anchor status: external_doc]
- [readme] Output arguments: [evidence span withheld — anchor status: external_doc]
- [readme] Gap-filled model validation: [evidence span withheld — anchor status: external_doc]

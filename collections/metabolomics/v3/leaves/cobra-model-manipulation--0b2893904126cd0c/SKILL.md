---
name: cobra-model-manipulation
description: Use when a draft SBML GENRE must be loaded, media-conditioned, and gap-filled
  via pFBA against a universal reaction database (universal.pickle) using COBRApy-compatible
  tooling.
when_to_use_negative:
- The input is an annotated .fasta or BLASTp output requiring de novo reconstruction
  (Type 1/2 input) rather than an existing SBML GENRE needing additional gap-filling
  (Type 3).
- The goal is to minimize the number of reactions added during gap-filling rather
  than total flux through the optimal reaction set — Reconstructor's parsimony criterion
  is flux-based, not reaction-count-based.
- The model is not in SBML format or naming conventions inconsistent with the ModelSEED
  biochemistry namespace, which Reconstructor and universal.pickle assume.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: Reconstructor
  role: Provides the `reconstruct` gap-filling routine that runs pFBA over the universal
    database to gap-fill the draft COBRApy model under user-defined media conditions.
  repo: https://github.com/emmamglass/reconstructor
- name: COBRApy
  role: Python framework used to load and manipulate the draft SBML GENRE and universal.pickle
    as Model objects, configure media, and export the gap-filled model.
- name: Python
  role: Runtime environment (>= Python 3.9) in which COBRApy model manipulation and
    the Reconstructor package execute; also used to modify universal.pickle via pickle.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:f78ad3a4a002b6541b18b11e8abc1e79d5e539fd48a947c421b91fb38fb5fbbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emmamglass/reconstructor
  tools:
  - COBRApy
  - Python
  - Reconstructor
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cobra-model-manipulation

## Summary

Manipulate genome-scale metabolic network reconstructions (GENREs) as COBRApy Model objects — loading SBML files, configuring media conditions, and running parsimonious flux balance analysis (pFBA) to gap-fill draft models. This is the programmatic backbone of Reconstructor's two-step gap-filling technique.

## When to use

Use when a draft SBML GENRE must be loaded, media-conditioned, and gap-filled via pFBA against a universal reaction database (universal.pickle) using COBRApy-compatible tooling.

## When NOT to use

- The input is an annotated .fasta or BLASTp output requiring de novo reconstruction (Type 1/2 input) rather than an existing SBML GENRE needing additional gap-filling (Type 3).
- The goal is to minimize the number of reactions added during gap-filling rather than total flux through the optimal reaction set — Reconstructor's parsimony criterion is flux-based, not reaction-count-based.
- The model is not in SBML format or naming conventions inconsistent with the ModelSEED biochemistry namespace, which Reconstructor and universal.pickle assume.

## Inputs

- Draft GENRE in SBML format (.sbml)
- Universal reaction database as a pickled COBRApy Model (universal.pickle)
- Media specification (e.g. 'rich' or comma-separated ModelSEED-namespace extracellular metabolite IDs ending in _e, such as 'cpd00001_e')

## Outputs

- Gap-filled GENRE exported as SBML (gapfilled_model.sbml)
- Gap-filling statistics (count of reactions added and biomass objective flux) in gapfill_stats.csv

## How to apply

Load the draft SBML GENRE and the universal reaction database (universal.pickle, a modified version of the ModelSEED biochemistry database) as COBRApy Model objects with `cobra.io.read_sbml_model` / pickle. Configure the draft model by opening uptake reactions for the specified extracellular metabolites (user-defined media, e.g. 'rich' or comma-separated ModelSEED-namespace compounds ending in _e). Run Reconstructor's `reconstruct` gap-filling routine, which performs parsimonious FBA to select the optimal reaction set from the universal database by minimizing total flux through gene-associated and non-gene-associated reactions while restoring biomass production — rather than minimizing the number of reactions added. Add the selected reactions to the draft network and export as gapfilled_model.sbml; tune gap-filling strictness with `--min_frac` (default 0.01, minimum objective fraction) and `--max_frac` (default 0.5, maximum objective fraction). The universal database can itself be modified via COBRApy and re-saved with pickle if curation (e.g., formula corrections) is needed.

## Related tools

- **Reconstructor** (Provides the `reconstruct` gap-filling routine that runs pFBA over the universal database to gap-fill the draft COBRApy model under user-defined media conditions.) — https://github.com/emmamglass/reconstructor
- **COBRApy** (Python framework used to load and manipulate the draft SBML GENRE and universal.pickle as Model objects, configure media, and export the gap-filled model.)
- **Python** (Runtime environment (>= Python 3.9) in which COBRApy model manipulation and the Reconstructor package execute; also used to modify universal.pickle via pickle.)

## Examples

```
python -m reconstructor --input draft_model.sbml --type 3 --media rich --min_frac 0.01 --max_frac 0.5
```

## Evaluation signals

- Confirm the gap-filled model carries positive biomass flux under the specified media conditions.
- The number of reactions added reported in gapfill_stats.csv is consistent with the model contents after gap-filling.
- The exported gapfilled_model.sbml loads cleanly as a COBRApy Model object.
- Objective flux after gap-filling falls within the configured bounds set by --min_frac (0.01) and --max_frac (0.5).
- Reaction and metabolite identifiers in the output remain consistent with ModelSEED biochemistry database naming conventions.

## Limitations

- Gap-filling minimizes flux through an optimal reaction set rather than minimizing the number of reactions added, so added-reaction counts may be larger than minimal-addition alternatives.
- Maintenance of the emmamglass/reconstructor repository has moved to csbl/reconstructor; the original repository is no longer updated.
- The universal reaction database is a modified version of ModelSEED biochemistry; models must follow ModelSEED-namespace metabolite IDs (e.g., 'cpd00001_e') for media specification to work.
- Results depend on Gram classification (`--gram positive/negative`) supplied for reconstruction, an organism-level prior that may be wrong or unknown.

## Evidence

- [full_text] The gap-filling mechanism minimizes flux through an optimal reaction set comprising all gene-associated reactions plus non-gene-associated reactions that minimize flux, rather than minimizing the number of reactions added: [evidence span withheld — anchor status: not_found]
- [methods] universal database source: "The universal reaction database used in `Reconstructor` is a modified version of the [ModelSEED biochemistry database](https://github.com/ModelSEED/ModelSEEDDatabase)"
- [methods] locate universal.pickle: "locate the universal.pickle file that was downloaded during the `Reconstructor` installation phase on your local computer"
- [intro] Reconstructor is COBRApy compatible: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations."
- [readme] Type 3 gap-filling CLI invocation: "#### Type 3: Additional gap-filling (if necessary)

shell
python -m reconstructor --input Osplanchnicus.sbml --type 3 --media rich
"
- [readme] gap-filling fraction bounds: [evidence span withheld — anchor status: not_found]

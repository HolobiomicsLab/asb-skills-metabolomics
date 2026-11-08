---
name: parsimonious-flux-balance-analysis-gap-filling
description: Use when a draft SBML GENRE fails to produce biomass under defined media
  conditions and reactions must be added from a universal reaction database (universal.pickle)
  to restore growth.
when_to_use_negative:
- The input is a .fasta or BLASTp output file requiring de novo reconstruction — use
  Reconstructor type 1 or type 2 inputs instead of type 3 gap-filling
- The draft model already achieves the required biomass objective under the media
  conditions and no reactions need to be added
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: Reconstructor
  role: Provides the reconstruct gap-filling routine that performs pFBA-based reaction
    selection and adds reactions to the draft model
  repo: https://github.com/emmamglass/reconstructor
- name: COBRApy
  role: Loads the draft SBML model and universal.pickle database as Model objects
    and executes flux balance analysis
- name: Python
  role: Runtime environment (>= Python 3.9) for running Reconstructor and COBRApy
    scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:4b589f1810fb003be9a7c4667904cf60ddfddccce6acf4b4cff1a66398591c13
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# parsimonious-flux-balance-analysis-gap-filling

## Summary

Gap-filling of draft genome-scale metabolic network reconstructions (GENREs) using parsimonious flux balance analysis (pFBA), as implemented in the COBRApy-compatible Reconstructor package. It minimizes total flux through candidate reactions from the universal ModelSEED-derived database rather than minimizing the number of reactions added.

## When to use

Use when a draft SBML GENRE fails to produce biomass under defined media conditions and reactions must be added from a universal reaction database (universal.pickle) to restore growth.

## When NOT to use

- The input is a .fasta or BLASTp output file requiring de novo reconstruction — use Reconstructor type 1 or type 2 inputs instead of type 3 gap-filling
- The draft model already achieves the required biomass objective under the media conditions and no reactions need to be added

## Inputs

- Draft genome-scale metabolic network reconstruction in SBML format (.sbml)
- Universal reaction database (universal.pickle), a modified version of the ModelSEED biochemistry database
- User-defined media condition: comma-separated list of ModelSEED-namespace extracellular metabolites ending with _e, or the 'rich' default

## Outputs

- Final gap-filled GENRE exported in SBML format (gapfilled_model.sbml)
- Gap-filling statistics (count of reactions added and biomass objective flux) in gapfill_stats.csv

## How to apply

Load the draft SBML GENRE and the universal reaction database (universal.pickle, a modified version of the ModelSEED biochemistry database) as COBRApy Model objects. Configure the draft model with the user-defined media conditions by opening uptake reactions for the specified extracellular metabolites (ModelSEED namespace IDs ending in _e, e.g. 'cpd00001_e'). Run Reconstructor's reconstruct gap-filling routine, which performs parsimonious flux balance analysis to select the optimal reaction set — minimizing total flux through all gene-associated reactions plus non-gene-associated reactions — while restoring biomass production, respecting the min_frac (default 0.01) and max_frac (default 0.5) objective fraction bounds. Add the selected reactions to the draft network to produce the final gap-filled GENRE and export it as gapfilled_model.sbml. Record the count of reactions added and biomass objective flux to gapfill_stats.csv, and confirm positive biomass flux under the specified media as validation.

## Related tools

- **Reconstructor** (Provides the reconstruct gap-filling routine that performs pFBA-based reaction selection and adds reactions to the draft model) — https://github.com/emmamglass/reconstructor
- **COBRApy** (Loads the draft SBML model and universal.pickle database as Model objects and executes flux balance analysis)
- **Python** (Runtime environment (>= Python 3.9) for running Reconstructor and COBRApy scripts)

## Examples

```
python -m reconstructor --input Osplanchnicus.sbml --type 3 --media rich
```

## Evaluation signals

- Gap-filled model carries positive biomass flux under the specified media conditions
- Number of added reactions reported in gapfill_stats.csv is consistent with the gap-filled model's contents
- Added reactions and metabolites use ModelSEED-consistent naming conventions
- Biomass objective falls within the allowed gap-filling fraction bounds (min_frac default 0.01, max_frac default 0.5)
- Reconstructor's installation test suite passes (python -m reconstructor --test yes), confirming a functional reconstruction environment

## Limitations

- The universal database requires prior curation — poorly defined metabolite formulas must be corrected and mass-imbalanced reactions removed (curateuniversal.py) before gap-filling
- Gap-filling minimizes total flux through the optimal reaction set, not the number of reactions added, so the added reaction count may be larger than minimal-set gap-filling approaches
- The emmamglass/reconstructor repository is no longer updated; maintenance has moved to csbl/reconstructor
- Documentation was noted as still under development

## Evidence

- [abstract] pFBA gap-filling principle: "produces high-quality draft reconstructions with reaction and metabolite naming conventions that are consistent with the ModelSEED biochemistry database and includes a gap-filling technique based on"
- [abstract] type 3 gap-filling input: "Reconstructor can generate SBML GENREs from three input types: annotated protein .fasta sequences (Type 1 input), a BLASTp output (Type 2), or an existing SBML GENRE that can be further gap-filled"
- [methods] universal database curation: "We corrected poorly
defined metabolite formulas and removed mass imbalanced reactions using the
curateuniversal.py script"
- [methods] universal database provenance: "The universal reaction database used in `Reconstructor` is a modified version of
the ModelSEED biochemistry database"
- [readme] type 3 CLI invocation: "#### Type 3: Additional gap-filling (if necessary)

shell
python -m reconstructor --input Osplanchnicus.sbml --type 3 --media rich
"
- [readme] gap-filling fraction bounds: [evidence span withheld — anchor status: external_doc]

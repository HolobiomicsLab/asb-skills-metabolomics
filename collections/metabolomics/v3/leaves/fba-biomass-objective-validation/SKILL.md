---
name: fba-biomass-objective-validation
description: Use when a draft or Reconstructor-generated SBML model has just been
  built and must be confirmed functional (nonzero biomass flux) prior to gene essentiality
  or other perturbation analyses.
when_to_use_negative:
- The model is not yet reconstructed or the SBML file fails to load into COBRApy as
  a valid Model object.
- The goal is model quality benchmarking (e.g., MEMOTE scores) rather than confirming
  flux feasibility before perturbation analysis.
- The model already has published growth/essentiality predictions and only needs comparison
  rather than revalidation.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_2259
tools:
- name: COBRApy
  role: Loads the SBML GENRE as a Model object and performs the flux balance analysis
    used to verify nonzero biomass flux.
- name: Reconstructor
  role: Produces the SBML GENRE (with parsimony-based gap-filling and ModelSEED naming)
    that is the input to the FBA validation step.
  repo: https://github.com/emmamglass/reconstructor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad367
    title: 'Reconstructor: a COBRApy compatible tool for automated genome-scale metabolic
      network reconstruction with parsimonious flux-based gap-filling'
  source_content_hash: sha256:56f66b89be1671f2119556b5506da580199246e1b4579ef34fe95cf99ddec5b6
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
yes_rate: 0.5
n_claims: 10
---

# fba-biomass-objective-validation

## Summary

Validates a newly reconstructed genome-scale metabolic network (GENRE) by running COBRApy flux balance analysis to confirm the model carries nonzero flux to the biomass objective before downstream deletion screening.

## When to use

Use when a draft or Reconstructor-generated SBML model has just been built and must be confirmed functional (nonzero biomass flux) prior to gene essentiality or other perturbation analyses.

## When NOT to use

- The model is not yet reconstructed or the SBML file fails to load into COBRApy as a valid Model object.
- The goal is model quality benchmarking (e.g., MEMOTE scores) rather than confirming flux feasibility before perturbation analysis.
- The model already has published growth/essentiality predictions and only needs comparison rather than revalidation.

## Inputs

- SBML GENRE file exported by Reconstructor (e.g., NCGM2.S1.xml)
- COBRApy Model object loaded from the SBML file
- Biomass reaction set as the model objective

## Outputs

- FBA objective value (biomass flux) confirming the model supports growth
- Go/no-go signal for proceeding to single-gene deletion essentiality analysis

## How to apply

Load the Reconstructor-exported SBML file (e.g., NCGM2.S1.xml) into COBRApy as a Model object, then perform flux balance analysis on the model whose objective is the biomass reaction. Confirm the optimized objective value is nonzero, which shows the parsimony-based gap-filled model can produce biomass under the specified (e.g., rich media) conditions. Only after this gate passes should you proceed to cobra.flux_analysis.single_gene_deletion, since essentiality calls are meaningless on a model that cannot grow. Rationale: Reconstructor models use ModelSEED-consistent reaction and metabolite naming, so the biomass reaction and exchange reactions resolve directly in COBRApy without format conversion.

## Related tools

- **COBRApy** (Loads the SBML GENRE as a Model object and performs the flux balance analysis used to verify nonzero biomass flux.)
- **Reconstructor** (Produces the SBML GENRE (with parsimony-based gap-filling and ModelSEED naming) that is the input to the FBA validation step.) — https://github.com/emmamglass/reconstructor

## Examples

```
import cobra; model = cobra.io.read_sbml_model('NCGM2.S1.xml'); solution = model.optimize(); assert solution.objective_value > 0
```

## Evaluation signals

- The SBML file (e.g., NCGM2.S1.xml) exists and loads into COBRApy as a valid Model object without errors.
- The FBA solution objective value (biomass flux) is strictly nonzero.
- The validation passes before any cobra.flux_analysis.single_gene_deletion screening is run.
- Downstream essential-genes table (e.g., NCGM2.S1_essential_genes.csv) covers the genes of the model with an essential/non-essential label for each.

## Limitations

- A nonzero biomass flux only confirms the gap-filled model can grow under the tested media condition (e.g., rich media); it does not guarantee biologically accurate predictions.
- Essentiality calls downstream are only as good as the draft reconstruction and parsimony-based gap-filling; missing reactions can produce false essentiality or false non-essentiality.
- Reconstructor requires Python >= 3.9, and the DIAMOND binary/database download step can be slow (~1hr) depending on the system.

## Evidence

- [other] Reconstructor's direct COBRApy compatibility enabled applying COBRApy tools to run a gene essentiality analysis.: [evidence span withheld — anchor status: not_found]
- [other] Load the resulting SBML model into COBRApy as a Model object.: [evidence span withheld — anchor status: not_found]
- [other] Perform COBRApy flux balance analysis on the model to confirm it carries flux to the biomass objective before deletion screening.: [evidence span withheld — anchor status: not_found]
- [other] verify that NCGM2.S1.xml exists, is a valid SBML model loadable by COBRApy, supports nonzero biomass flux: [evidence span withheld — anchor status: not_found]
- [readme] Reconstructor is a COBRApy compatible, automated GENRE building tool from annotated aminoa acid .fasta files based on KEGG annotations.: "`Reconstructor` is a COBRApy compatible, automated GENRE building tool
from annotated aminoa acid .fasta files based on KEGG annotations."

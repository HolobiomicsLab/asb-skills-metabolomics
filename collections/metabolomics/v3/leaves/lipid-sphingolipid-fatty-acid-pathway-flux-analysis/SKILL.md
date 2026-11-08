---
name: lipid-sphingolipid-fatty-acid-pathway-flux-analysis
description: Use when comparing flux distributions between host-optimal and virus-optimal
  objectives in an integrated host–virus genome-scale metabolic model, especially
  to quantify pathway-level (lipid, sphingolipid, fatty acid) flux perturbations caused
  by maximal viral growth.
when_to_use_negative:
- The analysis goal is chokepoint/essential-reaction drug-target identification rather
  than comparing flux distributions between two objective-function optima (use findCPcli
  bottleneck analysis instead)
- The model is not an integrated host–virus model with a distinct viral biomass objective,
  so no virus-optimum condition can be defined
- Only a single objective (host-only or virus-only) is of interest and no host-vs-virus
  flux comparison is needed
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: COBRApy
  role: Loads the SBML model and runs FBA and FVA to compute flux bounds under host-
    and virus-optimal conditions
  repo: https://github.com/opencobra/cobrapy
- name: GLPK
  role: Linear programming solver used by COBRApy for the FBA and FVA optimizations
- name: findCPcli
  role: Companion tool of the article for chokepoint and FVA-based flux-bound computation
    on the same genome-scale models
  repo: https://github.com/findCP/findCPcli
- name: Python
  role: Scripting environment for computing percentage flux increases and pathway
    groupings and writing flux_comparison.xls
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.26508/lsa.202000954
    title: Integrated human/SARS-CoV-2 metabolic models present novel treatment strategies
      against COVID-19
  source_content_hash: sha256:c26267329d35d02ffc174f0d1598c37b53a1fd5b98f07d1fd28cd1f11f8a6834
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobrapy
  tools:
  - COBRApy
  - GLPK
  - Python
  - findCPcli
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# lipid-sphingolipid-fatty-acid-pathway-flux-analysis

## Summary

Quantifies how host-cell metabolic fluxes in lipid, sphingolipid, and fatty-acid biosynthesis pathways shift when the SARS-CoV-2 virus is run at its optimum versus the host-optimum condition in the integrated iHsaEC21+SARS-CoV-2 model, using FVA in COBRApy with the GLPK solver.

## When to use

Use when comparing flux distributions between host-optimal and virus-optimal objectives in an integrated host–virus genome-scale metabolic model, especially to quantify pathway-level (lipid, sphingolipid, fatty acid) flux perturbations caused by maximal viral growth.

## When NOT to use

- The analysis goal is chokepoint/essential-reaction drug-target identification rather than comparing flux distributions between two objective-function optima (use findCPcli bottleneck analysis instead)
- The model is not an integrated host–virus model with a distinct viral biomass objective, so no virus-optimum condition can be defined
- Only a single objective (host-only or virus-only) is of interest and no host-vs-virus flux comparison is needed

## Inputs

- SBML file of the integrated human airway epithelial cell / SARS-CoV-2 model (iHsaEC21+SARS-CoV-2; 3,752 reactions, 3,914 metabolites, 4,660 genes, 48 pathways; BioModels MODEL2007210001)
- Host biomass objective function and viral biomass maintenance objective function defined in the model

## Outputs

- Per-reaction lower and upper flux bounds from FVA under host-optimum and virus-optimum conditions
- Per-reaction percentage increase in flux (virus-optimum vs host-optimum)
- Pathway-level summary averages of flux increases for lipid metabolism, sphingolipid metabolism, and fatty acid biosynthesis
- flux_comparison.xls spreadsheet containing all reaction fluxes and percentage increases

## How to apply

Load the SBML-encoded integrated model iHsaEC21+SARS-CoV-2 (3,752 reactions, 3,914 metabolites, 4,660 genes, 48 pathways) with COBRApy 0.16.0 and the GLPK solver. First run FBA maximizing the host biomass objective without the viral production reaction to obtain the host-optimal growth rate μmax; then run FBA with the viral biomass maintenance function as objective to obtain the virus-optimal solution. Run FVA on the host network with and without the viral production reaction, constraining vgrowth = μmax for the host case, to obtain lower and upper flux bounds per reaction under both optima. For each reaction, compute the percentage increase in flux between the host-optimum and virus-optimum solutions, then group perturbed reactions by pathway (lipid metabolism, sphingolipid metabolism, fatty acid biosynthesis) and average per pathway. Save per-reaction host vs virus fluxes and percentage increases as flux_comparison.xls with pathway-level summary averages. Validation: pathway averages should reproduce the reported values — 277% (32 lipid reactions), 228% (14 sphingolipid reactions), and ~190% (4 fatty-acid biosynthesis reactions) — and the table must cover all reactions returned by both FVA runs.

## Related tools

- **COBRApy** (Loads the SBML model and runs FBA and FVA to compute flux bounds under host- and virus-optimal conditions) — https://github.com/opencobra/cobrapy
- **GLPK** (Linear programming solver used by COBRApy for the FBA and FVA optimizations)
- **findCPcli** (Companion tool of the article for chokepoint and FVA-based flux-bound computation on the same genome-scale models) — https://github.com/findCP/findCPcli
- **Python** (Scripting environment for computing percentage flux increases and pathway groupings and writing flux_comparison.xls)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('iHsaEC21_SARS-CoV-2.xml'); host_sol = cobra.flux_analysis.flux_variability_analysis(model, fraction_of_optimum=1.0)
```

## Evaluation signals

- Pathway-level averages reproduce the reported values: 277% for 32 lipid reactions, 228% for 14 sphingolipid reactions, and ~190% for 4 fatty-acid biosynthesis reactions
- The flux comparison table covers all reactions returned by both FVA runs (with and without the viral production reaction)
- Host FVA under the virus-optimum solution yields zero biomass maintenance flux, confirming host-cell non-viability under virus-optimum conditions
- Host-case FVA is constrained at vgrowth = μmax, so flux bounds in the host-optimum run satisfy the host biomass equality constraint
- flux_comparison.xls contains per-reaction host and virus fluxes plus computed percentage increases for every pathway-grouped reaction

## Limitations

- Under the virus-optimum condition the host-cell biomass maintenance flux is zero, so percentage increases reflect an infeasible-for-the-host state rather than a co-viable steady state
- Results are specific to the integrated epithelial cell/SARS-CoV-2 model and its 48 pathways; pathway averages depend on the pathway annotation of the model
- FVA flux bounds can be large intervals; percentage increases based on bounds may not correspond to a single unique flux solution
- No experimental validation of the in silico flux predictions (e.g., lipid pathway perturbations) was presented in the article

## Evidence

- [results] Host non-viable at virus optimum: [evidence span withheld — anchor status: not_found]
- [methods] COBRApy 0.16.0 instantiation: "iHsaEC21, can be instantiated without error on the COBRA software (version 0.16.0) (Ebrahim et al, 2013)"
- [intro] FBA used for both objectives: "The Flux Balance Analysis (FBA) method (Orth et al, 2010) was used to compute both the biomass maintenance of the cell in the absence of virus and the maximum growth rate of the virus"
- [intro] FVA computes flux bounds: "FVA is used to compute flux bounds of the reactions, and in turn, to determine whether a given reaction is reversible or not."
- [readme] findCPcli FVA flux-bound updates: "find essential reactions and update the flux bounds of the reactions according to the results of Flux Variability Analysis"

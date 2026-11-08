---
name: genome-scale-metabolic-model-curation
description: Use when a workflow requires a simulation-ready GEM (e.g., for FVA-based
  shadow-price extraction) and only a raw published model file such as iJO1366.mat
  or Yeast-GEM.mat is available.
when_to_use_negative:
- A curated irreversible model (ModelIrrevOpt90.mat or yeast_IrrevOpt90.mat) is already
  available — rerunning curation is unnecessary
- The downstream task needs original reversible model semantics (e.g., SBML-level
  model browsing) rather than the irreversible form required by SARTRE's optimization
  step
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0605
tools:
- name: COBRAToolbox
  role: MATLAB toolbox used to curate the GEMs (irreversibility transformation and
    model preparation)
  repo: https://github.com/opencobra/cobratoolbox
- name: MATLAB
  role: Runtime environment in which modelPreparing_*.m curation scripts are executed
- name: SARTRE
  role: Downstream framework that consumes the curated irreversible models to compute
    shadow prices for MPI prediction
  repo: https://github.com/fayazsoleymani/SARTRE
- name: yeast-GEM
  role: Source consensus S. cerevisiae model (Yeast8) supplied as Yeast-GEM.mat input
    to the yeast curation script
  repo: https://github.com/SysBioChalmers/yeast-GEM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad098
    title: Prediction of metabolite–protein interactions based on integration of machine
      learning and constraint-based modeling
  source_content_hash: sha256:f23895b3d1d3b6f5bebaa551868da4edd54520f4c1dda0086e6f4fe46ed6d86f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - Bio.Entrez
  - COBRAToolbox
  - GUROBI
  - MATLAB
  - Python
  - RDKit
  - SARTRE
  - scikit-learn
  - yeast-GEM
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# genome-scale-metabolic-model-curation

## Summary

Prepare genome-scale metabolic models (GEMs) for downstream constraint-based analyses by curating published reconstructions (iJO1366, Yeast-GEM/Yeast8) with COBRAToolbox in MATLAB, producing irreversible optimized models that support shadow-price calculation in SARTRE.

## When to use

Use when a workflow requires a simulation-ready GEM (e.g., for FVA-based shadow-price extraction) and only a raw published model file such as iJO1366.mat or Yeast-GEM.mat is available.

## When NOT to use

- A curated irreversible model (ModelIrrevOpt90.mat or yeast_IrrevOpt90.mat) is already available — rerunning curation is unnecessary
- The downstream task needs original reversible model semantics (e.g., SBML-level model browsing) rather than the irreversible form required by SARTRE's optimization step

## Inputs

- iJO1366.mat — comprehensive genome-scale reconstruction of E. coli metabolism
- Yeast-GEM.mat — consensus S. cerevisiae metabolic model Yeast8 (github.com/SysBioChalmers/yeast-GEM)
- MATLAB environment with COBRAToolbox installed

## Outputs

- ModelIrrevOpt90.mat — irreversible optimized E. coli model
- yeast_IrrevOpt90.mat — irreversible optimized S. cerevisiae model
- Curated GEMs ready for shadow-price computation via optimization.py (GUROBI)

## How to apply

Run the model preparation scripts in MATLAB with COBRAToolbox installed: `modelPreparing_iJO1366.m` takes `iJO1366.mat` as input and creates `ModelIrrevOpt90.mat`, while `modelPreparing_yeastGEM.m` takes `Yeast-GEM.mat` (the consensus S. cerevisiae model Yeast8) and creates `yeast_IrrevOpt90.mat`. The curation converts the reversible stoichiometric matrix into an irreversible optimized form (the 'IrrevOpt90' outputs), which is the format required by the subsequent GUROBI-based optimization step (`optimization.py`) that computes per-reaction shadow prices. Ensure the MATLAB/COBRA environment can read the model files, and verify that the output .mat files exist and load correctly before handing them to the Python optimization stage.

## Related tools

- **COBRAToolbox** (MATLAB toolbox used to curate the GEMs (irreversibility transformation and model preparation)) — https://github.com/opencobra/cobratoolbox
- **MATLAB** (Runtime environment in which modelPreparing_*.m curation scripts are executed)
- **SARTRE** (Downstream framework that consumes the curated irreversible models to compute shadow prices for MPI prediction) — https://github.com/fayazsoleymani/SARTRE
- **yeast-GEM** (Source consensus S. cerevisiae model (Yeast8) supplied as Yeast-GEM.mat input to the yeast curation script) — https://github.com/SysBioChalmers/yeast-GEM

## Examples

```
matlab -batch "run('modelPreparing_yeastGEM.m')"  % produces yeast_IrrevOpt90.mat from Yeast-GEM.mat
```

## Evaluation signals

- The script produces its named output model file: modelPreparing_iJO1366.m → ModelIrrevOpt90.mat; modelPreparing_yeastGEM.m → yeast_IrrevOpt90.mat
- The curated model loads in MATLAB/COBRAToolbox without errors and preserves the organism's reaction/metabolite/gene content from the source .mat
- The irreversible optimized model is in the format expected by `optimization.py` (GUROBI optimizer and its Python interface), which writes per-reaction results to the `opt` directory
- Published model provenance is correct: E. coli workflow uses iJO1366; yeast workflow uses Yeast-GEM/Yeast8 from SysBioChalmers/yeast-GEM

## Limitations

- Curation depends on the specific source model versions (e.g., yeast-GEM releases are versioned; users should record the exact Yeast8 release used, as recommended by the yeast-GEM README)
- The README warns 'MAKE sure to unzip archives' — downstream optimization results shipped as .zip must be unpacked before reuse
- GUROBI requires an active license for the subsequent optimization step; the curation step itself is MATLAB/COBRAToolbox-dependent, so the full pipeline spans two toolchains

## Evidence

- [readme] modelPreparing_iJO1366.m uses `iJO1366.mat` as the input and creates `ModelIrrevOpt90.mat` and  `modelPreparing_yeastGEM.m` uses `Yeast-GEM.mat` and creates `yeast_IrrevOpt90.mat`.: "modelPreparing_iJO1366.m uses `iJO1366.mat` as the input and creates `ModelIrrevOpt90.mat` and  `modelPreparing_yeastGEM.m` uses `Yeast-GEM.mat` and creates `yeast_IrrevOpt90.mat`."
- [methods] Curation of these mentioned GEMs have been performed by the  COBRAToolbox in MATLAB: "Curation of these mentioned GEMs have been performed by the  COBRAToolbox in MATLAB"
- [readme] [Yeast-GEM](https://github.com/SysBioChalmers/yeast-GEM): A consensus *S. cerevisiae* metabolic model Yeast8 and its ecosystem for comprehensively probing cellular metabolism: "[Yeast-GEM](https://github.com/SysBioChalmers/yeast-GEM): A consensus *S. cerevisiae* metabolic model Yeast8 and its ecosystem for comprehensively probing cellular metabolism"
- [readme] The optimization process has been implemented by [GUROBI](https://www.gurobi.com/) optimizer and its python interface. Make sure to be installed and have an active license for it beforehand.: "The optimization process has been implemented by [GUROBI](https://www.gurobi.com/) optimizer and its python interface. Make sure to be installed and have an active license for it beforehand."

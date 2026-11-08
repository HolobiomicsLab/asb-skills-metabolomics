---
name: python-scientific-module-implementation
description: Reference for implementing a Python scientific module when a paper's
  algorithm must be reconstructed in code — e.g. reproducing MetAssimulo 2.0's peak
  detection, pH-shift computation, or spectral simulation routines inside an existing
  Python package.
when_to_use_negative:
- The spectral data is real experimental NMR rather than MetAssimulo-simulated spectra
  — the modules are built around the paper's simulated spectral data structures and
  its HMDB 5.0 parameter tables, not raw instrument output.
- The goal is interactive parameter exploration of the simulator rather than reproducible
  module implementation — use the Dash web app (http://127.0.0.1:8060/) driven by
  Input/parameters.txt instead of writing standalone modules.
- The metabolite of interest has no pKa and no Tredwell et al. normal-distribution
  fallback is desired — the module cannot then compute a defensible Henderson-Hasselbalch
  shift.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetAssimulo 2.0
  role: 'The Python codebase being extended: new modules must be importable from its
    package root after PYTHONPATH export and must integrate with apps/index.py and
    Input/parameters.txt.'
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: Python
  role: Implementation language for the reusable shift and cluster-localization functions
    and their unit tests.
- name: Anaconda
  role: Environment manager used to create and activate the py37_metassimulo_2_env
    conda environment from environment.yml before implementing or testing modules.
- name: Git
  role: Used to clone the MetAssimulo_2 repository so the modules can be added to
    the existing package structure.
  repo: https://github.com/yanyan5420/MetAssimulo_2.git
- name: gdown
  role: Downloads Input.zip (input data and parameters.txt) needed to exercise and
    validate the implemented modules end-to-end.
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf045
    title: 'MetAssimulo 2.0: a web app for simulating realistic 1D and 2D metabolomic
      1H NMR spectra'
  source_content_hash: sha256:f14c76dcb09dd7adea8c5fe5e887c7f7b065927d470994d384e742c7a448a07e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yanyan5420/MetAssimulo_2.git
  tools:
  - Anaconda
  - Git
  - MetAssimulo 2
  - MetAssimulo 2.0
  - Python
  - gdown
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7083333333333334
n_claims: 24
---

# python-scientific-module-implementation

## Summary

Wrap the paper's computational steps (pH-dependent Henderson-Hasselbalch peak shifting and threshold-based peak-cluster detection on 1D and projected 2D NMR profiles) as reusable Python modules in the MetAssimulo 2.0 codebase, so 1D 1H, J-Res and COSY spectra share a single implementation pathway.

## When to use

Reference for implementing a Python scientific module when a paper's algorithm must be reconstructed in code — e.g. reproducing MetAssimulo 2.0's peak detection, pH-shift computation, or spectral simulation routines inside an existing Python package.

## When NOT to use

- The spectral data is real experimental NMR rather than MetAssimulo-simulated spectra — the modules are built around the paper's simulated spectral data structures and its HMDB 5.0 parameter tables, not raw instrument output.
- The goal is interactive parameter exploration of the simulator rather than reproducible module implementation — use the Dash web app (http://127.0.0.1:8060/) driven by Input/parameters.txt instead of writing standalone modules.
- The metabolite of interest has no pKa and no Tredwell et al. normal-distribution fallback is desired — the module cannot then compute a defensible Henderson-Hasselbalch shift.

## Inputs

- Metabolite peak parameter tables (delta_L, delta_HL, pH0, pH1, pKa) sourced from HMDB 5.0
- Target pH value supplied via the Input/parameters.txt configuration file consumed by apps/index.py
- Simulated 1D 1H NMR spectral data (intensity vs ppm profiles)
- Simulated 2D J-Res spectra requiring skyline (F2) projection
- Simulated 2D COSY spectra requiring diagonal projection

## Outputs

- Reusable Python module implementing threshold-based peak-cluster detection shared by 1D, J-Res and COSY pathways
- Reusable function computing pH-dependent peak-shift values per peak (equation 3 output) and per projected 2D cluster
- Unit-test results validating a known HMDB 5.0 metabolite's pH shift against equation (3)

## How to apply

First set up the MetAssimulo 2 environment: clone https://github.com/yanyan5420/MetAssimulo_2.git, create the conda environment from environment.yml (py37_metassimulo_2_env), and export PYTHONPATH so the package is importable; the web entry point is apps/index.py driven by Input/parameters.txt. Then implement each algorithmic step as a separate, reusable function rather than inline code: one module computes the pH-dependent chemical-shift change per peak via the Henderson-Hasselbalch transformation of equation (3), using per-metabolite delta_L, delta_HL, pH0 (typically 7.4), pH1 and pKa parameters drawn from HMDB 5.0 (falling back to a Tredwell et al. normal distribution when pKa is missing); a second module handles peak detection — for 1D 1H spectra, scan for local maxima above a user-defined intensity threshold and group adjacent above-threshold points into clusters with filtering; for 2D J-Res, apply a skyline projection along F2 and for COSY a diagonal projection to reduce the 2D matrix to a 1D profile that feeds the same cluster-localization routine. Decide on module boundaries so 1D and projected-2D data share one common cluster-localization entry point, and emit computed shift values and detected clusters as inspectable outputs (e.g. peak_shift.py) that can be unit-tested against known HMDB 5.0 metabolite parameters.

## Related tools

- **MetAssimulo 2.0** (The Python codebase being extended: new modules must be importable from its package root after PYTHONPATH export and must integrate with apps/index.py and Input/parameters.txt.) — https://github.com/yanyan5420/MetAssimulo_2.git
- **Python** (Implementation language for the reusable shift and cluster-localization functions and their unit tests.)
- **Anaconda** (Environment manager used to create and activate the py37_metassimulo_2_env conda environment from environment.yml before implementing or testing modules.)
- **Git** (Used to clone the MetAssimulo_2 repository so the modules can be added to the existing package structure.) — https://github.com/yanyan5420/MetAssimulo_2.git
- **gdown** (Downloads Input.zip (input data and parameters.txt) needed to exercise and validate the implemented modules end-to-end.)

## Examples

```
export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH; conda activate py37_metassimulo_2_env; python3 apps/index.py -p Input/parameters.txt
```

## Evaluation signals

- The implemented module runs end-to-end on a simulated spectrum from Input.zip and its detected peak clusters match manual inspection at the user-defined intensity threshold.
- Unit test on a metabolite whose pKa and delta values exist in HMDB 5.0 reproduces the equation-3 pH-dependent shift, consistent with section 2.3.2 of the paper.
- 1D, J-Res and COSY spectra all reach the same cluster-localization function (single entry point) rather than duplicating detection code per spectral type.
- Peak-shift and cluster outputs are inspectable (e.g. in peak_shift.py's output) and cover all peaks/clusters of the target metabolite set.
- Module is importable after `export PYTHONPATH=$HOME/MetAssimulo_2/:$PYTHONPATH` and runs without triggering changes to apps/index.py's parameter-driven behaviour.

## Limitations

- pKa is not always available in HMDB 5.0, requiring a Tredwell et al. normal-distribution fallback that introduces sampling variability in the computed shift.
- For 2D spectra, detection operates on a reduced 1D profile (skyline F2 projection for J-Res, diagonal projection for COSY), so true 2D cross-peak structure is not directly modelled by the cluster-localization routine.
- The modules depend on MetAssimulo 2.0's specific data structures and Python 3.7 conda environment (py37_metassimulo_2_env), so they are not drop-in portable to other codebases.

## Evidence

- [methods] For 1D 1H spectra, peaks are located based on a user-defined intensity threshold; for 2D J-Res and COSY spectra, a skyline projection (F2) or diagonal projection first produces a 1D profile that then feeds the same cluster-localization algorithm.: [evidence span withheld — anchor status: not_found]
- [methods] The peak shift is quantified by equation (3), which defines Δδ, the amount the peak is shifted in ppm, computed from δL and δHL (the peak positions in the acid and basic limits) and pH0, the pH of the standard spectrum, usually 7.4.: [evidence span withheld — anchor status: not_found]
- [methods] git clone https://github.com/yanyan5420/MetAssimulo_2.git: "git clone https://github.com/yanyan5420/MetAssimulo_2.git"
- [readme] conda env create -f environment.yml: "conda env create -f environment.yml
conda activate py37_metassimulo_2_env"

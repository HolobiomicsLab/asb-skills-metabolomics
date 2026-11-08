---
name: lc-msms-simulation-environment-orchestration
description: Use when a prepared chemical list (e.g. from ChemicalMixtureCreator)
  must be subjected to a simulated fragmentation strategy (Top-N DDA, DIA, SmartROI,
  WeightedDEW) and the resulting scans exported as an mzML file.
when_to_use_negative:
- When the goal is to generate the chemical mixture itself — use ChemicalMixtureCreator
  (upstream chemicals task) instead
- When the acquisition should run on real hardware — that requires the Thermo Fisher
  IAPI bridging code path, not the simulated Environment
- When input is already an mzML of acquired scans to be evaluated — use ViMMS's evaluation
  methods rather than re-running a simulation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: ViMMS 2.0
  role: Framework providing Environment, IndependentMassSpectrometer and Controller
    classes used to execute the simulated tandem-MS acquisition and write mzML
  repo: https://github.com/glasgowcompbio/vimms
- name: Thermo Fisher IAPI
  role: Bridging layer that lets the same controllers later run unchanged on Thermo
    Tribrid Fusion instruments instead of the simulator
  repo: https://github.com/thermofisherlsms/iapi
- name: Python
  role: Language in which controllers and the Environment run loop are instantiated
    and executed (requires Python 3.0 or higher)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:910f0414aa2efda63835dcd429cea1cb7d204f793a7bd44633c4de75149df600
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - Thermo Fisher IAPI
  - ViMMS 2.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# lc-msms-simulation-environment-orchestration

## Summary

Orchestrates a simulated LC-MS/MS acquisition in ViMMS 2.0 by binding an IndependentMassSpectrometer and a fragmentation controller (e.g. Top-N DDA) inside an Environment object, running the tandem-MS simulation, and exporting the scan list as mzML.

## When to use

Use when a prepared chemical list (e.g. from ChemicalMixtureCreator) must be subjected to a simulated fragmentation strategy (Top-N DDA, DIA, SmartROI, WeightedDEW) and the resulting scans exported as an mzML file.

## When NOT to use

- When the goal is to generate the chemical mixture itself — use ChemicalMixtureCreator (upstream chemicals task) instead
- When the acquisition should run on real hardware — that requires the Thermo Fisher IAPI bridging code path, not the simulated Environment
- When input is already an mzML of acquired scans to be evaluated — use ViMMS's evaluation methods rather than re-running a simulation

## Inputs

- Simulated chemical list produced via ChemicalMixtureCreator (upstream chemicals task)
- Configured controller instance (e.g. TopNController with N=3, isolation window 1, mz_tol 10, rt_tol 15, min_ms1_intensity 1.75E5)
- Mass spectrometer configuration (e.g. IndependentMassSpectrometer in positive-ion mode)
- Retention-time range for the simulated run (e.g. 0–1440 minutes)

## Outputs

- mzML file of the simulated run (e.g. simulated_topn_run.mzML) containing MS1 and MS2 scans
- Accumulated scan list held by the Environment for downstream evaluation

## How to apply

Load the simulated chemical list and instantiate an IndependentMassSpectrometer over it (e.g. in positive-ion mode). Instantiate the fragmentation controller — for Top-N DDA use TopNController with N=3, isolation window 1, mz_tol 10, rt_tol 15 and min_ms1_intensity 1.75E5 — reusing the parameter set fixed in the controller-definition step so the acquisition strategy is unchanged. Create an Environment binding the mass spectrometer and controller, set the retention-time range (e.g. (0, 1440) minutes) and enable the progress bar, then call env.run() so the controller steers fragmentation of the most intense MS1 ions while the spectrometer generates MS1/MS2 scans. Finally export the accumulated scan list with the Environment's write_mzML capability (e.g. to simulated_topn_run.mzML). Rationale: ViMMS 2.0's modularity means controllers, chemicals and the mass spectrometer are composed as a single simulated context, and the same controller code can later run unchanged on real Thermo Tribrid Fusion hardware via IAPI bridging.

## Related tools

- **ViMMS 2.0** (Framework providing Environment, IndependentMassSpectrometer and Controller classes used to execute the simulated tandem-MS acquisition and write mzML) — https://github.com/glasgowcompbio/vimms
- **Thermo Fisher IAPI** (Bridging layer that lets the same controllers later run unchanged on Thermo Tribrid Fusion instruments instead of the simulator) — https://github.com/thermofisherlsms/iapi
- **Python** (Language in which controllers and the Environment run loop are instantiated and executed (requires Python 3.0 or higher))

## Examples

```
env = Environment(mass_spec, controller, rt_range=(0, 1440), progress_bar=True); env.run(); env.write_mzML('simulated_topn_run.mzML', 'simulated_topn_run')
```

## Evaluation signals

- The exported mzML file (e.g. simulated_topn_run.mzML) exists and loads as a standard mzML mass-spectrometry document
- Scans in the mzML cover the full configured simulated retention-time range (e.g. 0–1440 minutes)
- The MS2 events in the scan list match the configured Top-N schedule (N=3 fragmentations per MS1 cycle with isolation window 1)
- Fragmented precursor intensities respect the min_ms1_intensity threshold of 1.75E5 (note: lowering to 1E4 gives a more accurate but slower simulation)
- The Environment run completed without the controller and spectrometer diverging from the shared chemical list, confirming the single-context composition

## Limitations

- The min_intensity threshold of 1.75E5 is set very high only to make notebooks complete quickly; a more accurate simulation requires lowering it to e.g. 1E4, which slows runs and increases memory usage, while a threshold of 0 includes all ROIs including noise
- Utility validation of ViMMS 2.0 (including running controllers unchanged on real instruments) is deferred to Davies et al. (2021), not this paper
- No evaluation results, benchmarks, or quantitative performance data are reported in the paper itself

## Evidence

- [methods] create an environment to run both the mass spec and controller: "create an environment to run both the mass spec and controller"
- [methods] The `Environment` class provides `write_mzML` to export the generated scans: "The `Environment` class provides `write_mzML` to export the generated scans"
- [results] the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [results] you should lower this threshold value to e.g. 1E4. However this results in more objects to be processed, which will make the simulation runs much slower: "you should lower this threshold value to e.g. 1E4. However this results in more objects to be processed, which will make the simulation runs much slower"
- [readme] a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics: "a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics"

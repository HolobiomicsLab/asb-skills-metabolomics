---
name: retention-time-range-configuration-for-virtual-runs
description: Use when executing a simulated LC-MS/MS acquisition in ViMMS 2.0 and
  you must bind the mass spectrometer and controller in an Environment and define
  the simulated RT window before env.run().
when_to_use_negative:
- The goal is to define or change the fragmentation strategy parameters themselves
  (N, isolation window, tolerances) — those belong to controller configuration, not
  Environment RT-range setup
- Acquisition is to be run on an actual Thermo Tribrid Fusion instrument via Thermo
  Fisher IAPI bridging rather than in simulation
- Only exporting or evaluating scans from an already-completed simulated run — use
  write_mzML or the evaluation methods directly instead of re-running the Environment
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ViMMS 2.0
  role: Provides the Environment class that binds mass spectrometer and controller
    as a single execution context and exports scans via write_mzML
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which the Environment, IndependentMassSpectrometer and controller
    objects are instantiated and env.run() is executed
  repo: https://github.com/glasgowcompbio/vimms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:0f373b9058e05d0e3ecbffddb481027aadf92bd93e42e4da2bcccdcb59f27d35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Retention-time range configuration for virtual runs

## Summary

Configures the ViMMS 2.0 Environment object that binds a simulated mass spectrometer and a fragmentation controller, setting the simulated retention-time window (e.g. 0–1440 minutes) before executing env.run() for an in-silico LC-MS/MS acquisition.

## When to use

Use when executing a simulated LC-MS/MS acquisition in ViMMS 2.0 and you must bind the mass spectrometer and controller in an Environment and define the simulated RT window before env.run().

## When NOT to use

- The goal is to define or change the fragmentation strategy parameters themselves (N, isolation window, tolerances) — those belong to controller configuration, not Environment RT-range setup
- Acquisition is to be run on an actual Thermo Tribrid Fusion instrument via Thermo Fisher IAPI bridging rather than in simulation
- Only exporting or evaluating scans from an already-completed simulated run — use write_mzML or the evaluation methods directly instead of re-running the Environment

## Inputs

- Simulated chemical list produced via ChemicalMixtureCreator (upstream chemicals task)
- Configured fragmentation controller instance (e.g. TopNController, N=3, isolation window 1, mz_tol 10, rt_tol 15, min_ms1_intensity 1.75E5)
- IndependentMassSpectrometer instance in positive-ion mode

## Outputs

- Environment object configured with retention-time range (0, 1440) minutes and progress bar enabled
- Accumulated scan list of simulated MS1/MS2 scans after env.run()
- Exported mzML file (simulated_topn_run.mzML) via the Environment's write_mzML capability

## How to apply

Instantiate an IndependentMassSpectrometer in positive-ion mode over the simulated chemical list produced upstream by ChemicalMixtureCreator, and attach the configured controller (e.g. TopNController with N=3, isolation window 1, mz_tol 10, rt_tol 15, min_ms1_intensity 1.75E5). Create an Environment object that brings together the input chemicals, mass spectrometer and controller as a single context, setting the retention-time range to (0, 1440) minutes and enabling the progress bar as shown in the Running Controllers example. Call env.run() to let the controller steer fragmentation of the most intense MS1 ions under the Top-N schedule while the simulated spectrometer generates MS1/MS2 scans. The rationale for a high min_intensity threshold (1.75E5) is simulation speed; the article notes lowering it to e.g. 1E4 gives a more accurate but much slower simulation, and setting it to 0 includes all ROIs including noise.

## Related tools

- **ViMMS 2.0** (Provides the Environment class that binds mass spectrometer and controller as a single execution context and exports scans via write_mzML) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which the Environment, IndependentMassSpectrometer and controller objects are instantiated and env.run() is executed) — https://github.com/glasgowcompbio/vimms

## Examples

```
env = Environment(mass_spec, controller, min_rt=0, max_rt=1440, progress_bar=True); env.run()
```

## Evaluation signals

- The exported mzML file (simulated_topn_run.mzML) exists and is loadable as a standard mzML mass-spectrometry document
- Scans in the mzML cover the full configured simulated retention-time range (0–1440 minutes) with no truncation
- Number and timing of Top-N MS2 events match the configured N=3 fragmentation schedule within each MS1 cycle
- The Environment was constructed over the same chemical list and controller parameters as the upstream tasks, so the acquisition strategy is unchanged

## Limitations

- The min_intensity threshold 1.75E5 used here is set very high only so the notebook completes quickly; lowering it to e.g. 1E4 gives a more accurate simulation but slower runs and higher memory usage, and setting it to 0 includes all ROIs including noise
- No validation of ViMMS 2.0's simulation fidelity is presented in this paper; utility validation via bridging to real instruments is deferred to Davies et al. (2021)

## Evidence

- [results] Create an Environment object binding the mass spectrometer and controller, setting the retention-time range to (0, 1440) minutes: [evidence span withheld — anchor status: not_found]
- [methods] create an environment to run both the mass spec and controller: "create an environment to run both the mass spec and controller"
- [methods] The `Environment` class provides `write_mzML` to export the generated scans: "The `Environment` class provides `write_mzML` to export the generated scans"
- [results] the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"

---
name: topn-dda-controller-execution-in-silico
description: Use when a prepared simulated chemical mixture must be subjected to a
  Top-N data-dependent acquisition schedule to produce simulated MS1/MS2 scans for
  downstream evaluation.
when_to_use_negative:
- The goal is data-independent acquisition — ViMMS 2.0 also implements DIA strategies
  such as All-ion-fragmentation and SWATH-MS, for which TopNController is the wrong
  controller.
- Only chemical objects need to be generated (ChemicalMixtureCreator task); this skill
  concerns running the acquisition, not creating the chemical mixture.
- The target is execution on real hardware — running controllers on actual Thermo
  Tribrid Fusion instruments requires the separate Thermo Fisher IAPI bridging setup,
  not the simulated Environment path.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ViMMS 2.0
  role: Framework providing Environment, IndependentMassSpectrometer, TopNController
    and write_mzML used to execute and export the simulated Top-N DDA run
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which the Environment/controller execution is scripted (requires
    Python 3.0 or higher)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:ee590801a0a800862453ac8ca44069f29f50e26403725a323a3e54405d3bc2de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# topn-dda-controller-execution-in-silico

## Summary

Executes a simulated Top-N DDA LC-MS/MS acquisition in ViMMS 2.0 by binding an IndependentMassSpectrometer and a TopNController inside an Environment, then exporting the resulting scan list to an mzML file. This reproduces the paper's controller-in-environment execution pattern for generating in-silico fragmentation data.

## When to use

Use when a prepared simulated chemical mixture must be subjected to a Top-N data-dependent acquisition schedule to produce simulated MS1/MS2 scans for downstream evaluation.

## When NOT to use

- The goal is data-independent acquisition — ViMMS 2.0 also implements DIA strategies such as All-ion-fragmentation and SWATH-MS, for which TopNController is the wrong controller.
- Only chemical objects need to be generated (ChemicalMixtureCreator task); this skill concerns running the acquisition, not creating the chemical mixture.
- The target is execution on real hardware — running controllers on actual Thermo Tribrid Fusion instruments requires the separate Thermo Fisher IAPI bridging setup, not the simulated Environment path.

## Inputs

- Simulated chemical list produced via ChemicalMixtureCreator (upstream chemicals task output)
- Top-N DDA controller parameter set (TopNController, N=3, isolation window 1, mz_tol 10, rt_tol 15, min_ms1_intensity 1.75E5)
- Retention-time range specification, e.g. (0, 1440) minutes

## Outputs

- simulated_topn_run.mzML — an mzML file containing the accumulated MS1/MS2 scan list from the simulated run
- In-memory scan list of MS1 and Top-N MS2 scans held by the Environment

## How to apply

Load the simulated chemical list produced via ChemicalMixtureCreator and instantiate an IndependentMassSpectrometer over those chemicals in positive-ion mode. Instantiate a TopNController with the configured Top-N DDA parameters (N=3, isolation window 1, mz_tol 10, rt_tol 15, min_ms1_intensity 1.75E5), reusing the same parameter set as any prior controller configuration so the acquisition strategy is unchanged. Create an Environment object that binds the mass spectrometer and controller together as a single context, setting the retention-time range to (0, 1440) minutes and enabling the progress bar. Call env.run() so the controller steers fragmentation of the most intense MS1 ions under the Top-N schedule while the spectrometer generates MS1/MS2 scans. Finally, export the accumulated scan list via the Environment's write_mzML capability to produce simulated_topn_run.mzML. The min_ms1_intensity threshold of 1.75E5 is deliberately high to reduce processing time; lowering it (e.g. to 1E4) yields a more accurate simulation but slower runs and higher memory usage.

## Related tools

- **ViMMS 2.0** (Framework providing Environment, IndependentMassSpectrometer, TopNController and write_mzML used to execute and export the simulated Top-N DDA run) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which the Environment/controller execution is scripted (requires Python 3.0 or higher))

## Examples

```
env = Environment(mass_spec, controller, rt_range=(0, 1440), progress_bar=True); env.run(); env.write_mzML('simulated_topn_run.mzML', out_dir)
```

## Evaluation signals

- The output mzML file (simulated_topn_run.mzML) exists and is loadable as a standard mzML mass-spectrometry document.
- Scans in the mzML cover the full configured simulated retention-time range (0–1440 minutes).
- The MS2 events in the run match the configured Top-N schedule (N=3 fragmentations per MS1 cycle).
- Controller parameters (isolation window 1, mz_tol 10, rt_tol 15, min_ms1_intensity 1.75E5) are identical to those specified in the preceding controller-configuration step.
- ViMMS's provided evaluation methods can be applied to the simulation results without errors.

## Limitations

- The min_ms1_intensity threshold of 1.75E5 is set very high only so the notebook completes quickly; it reduces the realism of the simulation, and setting it to 0 would include all ROIs including noise.
- Lowering the threshold to e.g. 1E4 gives a more accurate simulation but makes runs much slower at higher memory usage.
- This paper does not itself validate ViMMS 2.0's utility; validation via bridging code running simulated controllers unchanged on real instruments is reported in a separate publication (Davies et al., 2021).

## Evidence

- [methods] Controllers are executed in the context of an environment bringing together input chemicals, mass spectrometer and controllers as a single context: "Controllers are executed in the context of an environment bringing together input chemicals, mass spectrometer and controllers as a single context (Figure 1D)."
- [methods] Environment write_mzML: "The `Environment` class provides `write_mzML` to export the generated scans"
- [methods] Top-N DDA strategy: "A standard fragmentation strategy used in data-dependant acquisition (DDA) is the Top-N strategy"
- [results] min_intensity threshold 1.75E5 high to reduce processing time: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [results] Lowering threshold slows simulation: "you should lower this threshold value to e.g. 1E4. However this results in more objects to be processed, which will make the simulation runs much slower at higher memory usage"

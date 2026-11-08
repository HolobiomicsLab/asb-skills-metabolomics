---
name: controller-mass-spectrometer-interface-wiring
description: Use when a chemical mixture (e.g. from ChemicalMixtureCreator) and a
  configured controller (e.g. Top-N DDA) already exist and the goal is to execute
  the simulated tandem-MS acquisition and produce an mzML scan file.
when_to_use_negative:
- The chemicals have not yet been generated — ChemicalMixtureCreator (or mzML-based
  chemical extraction) must run first, since the Environment needs a chemical list
  to simulate over.
- The goal is to run the controller on a real Thermo Tribrid Fusion instrument via
  IAPI bridging rather than in simulation — that path is described as validated in
  separate work (Davies et al., 2021), not by this simulated Environment wiring.
- You only need a different fragmentation schedule tuned; re-wiring the Environment
  does not change the strategy — adjust the controller parameters instead.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ViMMS 2.0
  role: Provides the Environment class that binds the mass spectrometer and controller
    into a single execution context and exposes write_mzML for exporting scans
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which the Environment/controller/mass-spec objects are instantiated
    and env.run() is invoked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:9f5aa0081693feae8a833491a299c09723a31a9d4fb1277620caeccb3dd7e5c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# controller-mass-spectrometer-interface-wiring

## Summary

Wires a fragmentation controller to an IndependentMassSpectrometer inside a ViMMS 2.0 Environment so the controller steers MS1/MS2 acquisition of the loaded chemical list, and exports the resulting scans via write_mzML. This is the execution step that turns a configured acquisition strategy into a simulated LC-MS/MS run file.

## When to use

Use when a chemical mixture (e.g. from ChemicalMixtureCreator) and a configured controller (e.g. Top-N DDA) already exist and the goal is to execute the simulated tandem-MS acquisition and produce an mzML scan file.

## When NOT to use

- The chemicals have not yet been generated — ChemicalMixtureCreator (or mzML-based chemical extraction) must run first, since the Environment needs a chemical list to simulate over.
- The goal is to run the controller on a real Thermo Tribrid Fusion instrument via IAPI bridging rather than in simulation — that path is described as validated in separate work (Davies et al., 2021), not by this simulated Environment wiring.
- You only need a different fragmentation schedule tuned; re-wiring the Environment does not change the strategy — adjust the controller parameters instead.

## Inputs

- Simulated chemical list produced via ChemicalMixtureCreator (from the upstream chemicals task)
- Configured controller instance (e.g. TopNController, N=3, isolation window 1, mz_tol 10, rt_tol 15, min_ms1_intensity 1.75E5)
- Ionisation mode (positive-ion mode in the demonstrated workflow)
- Retention-time range for the simulated run (e.g. 0–1440 minutes)

## Outputs

- mzML file of the simulated run containing the accumulated MS1/MS2 scan list (e.g. simulated_topn_run.mzML)
- In-memory scan list held by the Environment after env.run()

## How to apply

Load the simulated chemical list produced upstream via ChemicalMixtureCreator and instantiate an IndependentMassSpectrometer over those chemicals (positive-ion mode in the demonstrated workflow). Instantiate the controller — e.g. TopNController with N=3, isolation window 1, mz_tol 10, rt_tol 15, min_ms1_intensity 1.75E5 — reusing the exact parameter set fixed in the preceding configuration step so the acquisition strategy is unchanged. Create an Environment object binding the mass spectrometer and controller, set the retention-time range (here 0–1440 minutes) and enable the progress bar, then call env.run() so the controller drives Top-N fragmentation of the most intense MS1 ions while the spectrometer generates MS1/MS2 scans. Finally use the Environment's write_mzML capability to export the accumulated scan list (e.g. simulated_topn_run.mzML) for downstream evaluation. The rationale is ViMMS 2.0's modularity: controllers, chemicals and spectrometer are swappable components brought together by the Environment as a single execution context.

## Related tools

- **ViMMS 2.0** (Provides the Environment class that binds the mass spectrometer and controller into a single execution context and exposes write_mzML for exporting scans) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which the Environment/controller/mass-spec objects are instantiated and env.run() is invoked)

## Examples

```
env = Environment(mass_spec, controller, min_time=0, max_time=1440, progress_bar=True); env.run(); env.write_mzML('simulated_topn_run.mzML')
```

## Evaluation signals

- The written mzML file (simulated_topn_run.mzML) exists and loads as a standard mzML mass-spectrometry document.
- Scans in the mzML cover the full configured retention-time range (0–1440 minutes).
- MS2 events in the output match the configured Top-N schedule (N=3 fragments per MS1 cycle under the isolation window 1 / mz_tol 10 / rt_tol 15 settings).
- The same controller parameter set used upstream is reused unchanged, so the executed acquisition strategy is identical to the configured one.
- Fragments selected for MS2 correspond to the most intense MS1 peaks above min_ms1_intensity 1.75E5.

## Limitations

- min_intensity threshold 1.75E5 is set very high only to make the demo notebook complete quickly; lowering it (e.g. to 1E4) gives a more accurate but much slower, memory-heavier simulation, and 0 includes noise ROIs.
- The scan-level simulation is in-silico; validation of running the same controllers unchanged on real instruments via IAPI bridging is reported in a separate publication (Davies et al., 2021), not in this paper.
- No explicit ViMMS 2.0 version number is stated in the paper; behaviour should be pinned to the glasgowcompbio/vimms repository state.

## Evidence

- [full_text] Controllers are executed in the context of an environment bringing together input chemicals, mass spectrometer and controllers as a single context: "Controllers are executed in the context of an environment bringing together input chemicals, mass spectrometer and controllers as a single context (Figure 1D)."
- [methods] create an environment to run both the mass spec and controller: "create an environment to run both the mass spec and controller"
- [methods] The Environment class provides write_mzML to export the generated scans: "The `Environment` class provides `write_mzML` to export the generated scans"
- [results] min_intensity threshold is set very high so the notebook completes quickly: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [methods] A standard fragmentation strategy used in data-dependant acquisition (DDA) is the Top-N strategy: "A standard fragmentation strategy used in data-dependant acquisition (DDA) is the Top-N strategy"

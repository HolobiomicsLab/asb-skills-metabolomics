---
name: dynamic-exclusion-window-handling
description: Use when implementing or tuning a Top-N DDA controller in ViMMS 2.0 where
  precursors selected for MS2 must be suppressed from repeated selection via dynamic
  exclusion (mz_tol / rt_tol windows), or when choosing between Top-N and its dynamically-exclusion-weighted
  variant WeightedDEWController.
when_to_use_negative:
- The target acquisition strategy is data-independent (DIA) — e.g. all-ion-fragmentation
  or SWATH-MS — since dynamic exclusion is a Top-N DDA mechanism
- The goal is evaluating SmartROIController ROI-guided adaptive scoring, where fragmentation
  prioritisation is not driven by simple Top-N intensity ranking with exclusion windows
- Running controllers unchanged on a real Thermo Tribrid Fusion instrument is required
  — the paper defers that validation to Davies et al. (2021), not this workflow
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS 2.0
  role: Framework in which the Top-N Controller subclass with dynamic exclusion (mz_tol,
    rt_tol) is implemented via _process_scan and Scan Parameters, and run in an Environment
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Implementation language for overriding _process_scan and defining controller
    parameters
- name: numpy
  role: Dependency used by ViMMS for the underlying peak/scan data structures the
    controller processes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:5cd71b32ade4296b980cec9c99cb9b2eec036b0296116f35c1c2a4b73d85957d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  - numpy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dynamic-exclusion-window-handling

## Summary

Implements and configures dynamic exclusion when building Top-N DDA fragmentation controllers in ViMMS 2.0: after a precursor is selected for MS2, it is barred from re-selection within m/z (mz_tol) and retention-time (rt_tol) windows so the N most intense, not-yet-excluded ions are fragmented.

## When to use

Use when implementing or tuning a Top-N DDA controller in ViMMS 2.0 where precursors selected for MS2 must be suppressed from repeated selection via dynamic exclusion (mz_tol / rt_tol windows), or when choosing between Top-N and its dynamically-exclusion-weighted variant WeightedDEWController.

## When NOT to use

- The target acquisition strategy is data-independent (DIA) — e.g. all-ion-fragmentation or SWATH-MS — since dynamic exclusion is a Top-N DDA mechanism
- The goal is evaluating SmartROIController ROI-guided adaptive scoring, where fragmentation prioritisation is not driven by simple Top-N intensity ranking with exclusion windows
- Running controllers unchanged on a real Thermo Tribrid Fusion instrument is required — the paper defers that validation to Davies et al. (2021), not this workflow

## Inputs

- Simulated MS1 survey scan (peak list with m/z and intensity) from a ViMMS 2.0 Environment run
- Chemical mixture generated via ChemicalMixtureCreator (e.g. UniformMZFormulaSampler, min_mz=100, max_mz=500)
- Controller parameters: N, mz_tol, rt_tol, min_ms1_intensity (e.g. 1.75E5), isolation_window width

## Outputs

- List of Scan Parameters objects per scan cycle (N MS2 fragmentation tasks + 1 follow-up MS1 survey task)
- Simulated LC-MS/MS scan schedule executable by an IndependentMassSpectrometer
- Exportable mzML file of the resulting acquisition

## How to apply

Subclass the base Controller from vimms.Controller and override _process_scan so that, given an incoming MS1 survey scan, peaks are sorted by intensity and the N most intense ions above the min_ms1_intensity threshold that are NOT already dynamically excluded are selected; exclusion is honoured using the mz_tol (m/z window) and rt_tol (retention-time window) parameters. For each selected precursor, create an MS2 Scan Parameters object with the chosen isolation_window width and append it to the pending task list. After the N MS2 tasks, schedule a follow-up MS1 survey scan and return the complete task list to the Environment for execution against an IndependentMassSpectrometer (e.g. rt_range 0–1440 min). If repeated fragmentation of abundant ions is problematic, consider the WeightedDEWController, a Top-N variant with dynamic exclusion weighting. Rationale: exclusion windows prevent the controller wasting MS2 scans on the same high-intensity peaks across consecutive survey scans, which is the defining behaviour of the Top-N DDA strategy the paper describes.

## Related tools

- **ViMMS 2.0** (Framework in which the Top-N Controller subclass with dynamic exclusion (mz_tol, rt_tol) is implemented via _process_scan and Scan Parameters, and run in an Environment) — https://github.com/glasgowcompbio/vimms
- **Python** (Implementation language for overriding _process_scan and defining controller parameters)
- **numpy** (Dependency used by ViMMS for the underlying peak/scan data structures the controller processes)

## Examples

```
controller = TopNController(mz_tol, rt_tol, min_ms1_intensity=1.75e5, N=10); env = Environment(mass_spec, controller); env.run(); env.write_mzML('output.mzML')
```

## Evaluation signals

- The controller's task list contains exactly N MS2 Scan Parameters entries per cycle followed by an MS1 survey scan task
- Ions already fragmented within the mz_tol / rt_tol exclusion windows are absent from subsequent precursor selections in the schedule
- All scheduled MS2 precursors have MS1 intensities at or above the min_ms1_intensity threshold (1.75E5 in the paper's notebook fixture)
- The Environment completes the run and produces an mzML export whose scans correspond to the scheduled controller tasks
- Re-running with exclusion disabled causes visibly repeated fragmentation of the same top-intensity ions, confirming exclusion logic is active

## Limitations

- The paper sets the min_intensity threshold to 1.75E5 purely for notebook speed; lowering it to e.g. 1E4 gives a more accurate simulation but much slower runs and higher memory usage, and a threshold of 0 includes all ROIs including noise
- No validation of ViMMS 2.0 (including controller behaviour on real instruments) is presented in this paper; utility validation via bridging code is deferred to Davies et al. (2021)
- No explicit ViMMS 2.0 version number or changelog is stated in the paper, so reproducibility of controller behaviour depends on pinning the repository state

## Evidence

- [abstract] Fragmentation strategies are implemented as controllers that extend from the base Controller class: "Fragmentation strategies are implemented
as controllers that extend from the base Controller class"
- [methods] A standard fragmentation strategy used in data-dependant acquisition (DDA) is the Top-N strategy: "A standard fragmentation strategy used in data-dependant acquisition (DDA) is the Top-N strategy"
- [methods] WeightedDEWController – a Top-N variant with dynamic exclusion weighting: "`WeightedDEWController` – a Top‑N variant with dynamic exclusion weighting"
- [results] The min_intensity threshold (1.75E5) is set very high here so the notebook completes quickly: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [methods] create an environment to run both the mass spec and controller: "create an environment to run both the mass spec and controller"

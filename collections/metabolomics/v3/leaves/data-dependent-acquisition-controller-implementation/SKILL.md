---
name: data-dependent-acquisition-controller-implementation
description: Use when prototyping or modifying a DDA fragmentation strategy (e.g.
  Top-N, dynamic exclusion variants) for LC-MS/MS metabolomics and you need to encode
  the precursor-prioritisation logic as a ViMMS 2.0 controller class.
when_to_use_negative:
- The goal is data-independent acquisition (DIA) such as All-ion-fragmentation or
  SWATH-MS — these are separate filter strategies in ViMMS and do not use the Top-N
  precursor-prioritisation logic.
- You need instrument-side execution rather than simulation without the bridging layer
  — running on a real Thermo Tribrid Fusion requires the Thermo Fisher IAPI bridging
  code described in Davies et al. (2021), not this paper's controller-only workflow.
- You want a ready-made enhanced DDA controller — SmartROIController and WeightedDEWController
  are already provided as example implementations in ViMMS 2.0.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ViMMS 2.0
  role: Framework whose base Controller class is extended and whose Environment/IndependentMassSpectrometer
    execute the implemented fragmentation strategy in simulation
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Implementation language for the controller subclass and _process_scan override
    (requires Python 3.0 or higher)
- name: numpy
  role: Dependency used for peak intensity sorting and numerical handling in the simulation
    pipeline
- name: Thermo Fisher IAPI
  role: Instrument API via which simulated controllers can later be run unchanged
    on real Thermo Tribrid Fusion instruments through bridging code
  repo: https://github.com/thermofisherlsms/iapi
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:725bf20f9a8de111669fb593929dd340013e881f38da51057ed2be1e1e74335b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - Thermo Fisher IAPI
  - ViMMS 2.0
  - numpy
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# data-dependent-acquisition-controller-implementation

## Summary

Implement a custom Top-N data-dependent acquisition (DDA) fragmentation strategy in ViMMS 2.0 by subclassing the base Controller class and overriding _process_scan, which converts an incoming MS1 survey scan into a prioritised list of MS2 Scan Parameters for the Environment to execute.

## When to use

Use when prototyping or modifying a DDA fragmentation strategy (e.g. Top-N, dynamic exclusion variants) for LC-MS/MS metabolomics and you need to encode the precursor-prioritisation logic as a ViMMS 2.0 controller class.

## When NOT to use

- The goal is data-independent acquisition (DIA) such as All-ion-fragmentation or SWATH-MS — these are separate filter strategies in ViMMS and do not use the Top-N precursor-prioritisation logic.
- You need instrument-side execution rather than simulation without the bridging layer — running on a real Thermo Tribrid Fusion requires the Thermo Fisher IAPI bridging code described in Davies et al. (2021), not this paper's controller-only workflow.
- You want a ready-made enhanced DDA controller — SmartROIController and WeightedDEWController are already provided as example implementations in ViMMS 2.0.

## Inputs

- MS1 survey scan (peak list with m/z and intensity) delivered to _process_scan
- Chemical dataset generated via ChemicalMixtureCreator / UniformMZFormulaSampler (e.g. min_mz=100, max_mz=500)
- Controller parameters: N (top-N count), mz_tol, rt_tol, min_ms1_intensity (e.g. 1.75E5), isolation_window width
- IndependentMassSpectrometer instance and Environment (e.g. rt_range 0-1440)

## Outputs

- List of Scan Parameters: MS2 fragmentation tasks for each selected precursor plus a follow-up MS1 survey scan
- Simulated MS/MS acquisition trajectory executable by the Environment, exportable as mzML via Environment.write_mzML

## How to apply

Import the base Controller class from vimms.Controller and define a new subclass (e.g. TopNController). Override _process_scan so that, given an MS1 survey scan, peaks are sorted by intensity and the N most intense ions that are not dynamically excluded are selected, honouring the mz_tol and rt_tol exclusion windows and the min_ms1_intensity threshold (the paper's example uses 1.75E5, set high only so the notebook completes quickly; lowering to ~1E4 gives a more accurate but slower simulation). For each selected precursor, create MS2 scan parameters with the chosen isolation_window width and append them to the pending task list, then schedule a follow-up MS1 survey scan after the N MS2 tasks. Return the complete task list to the Environment, which runs it against an IndependentMassSpectrometer over a ChemicalMixtureCreator-generated test mixture (e.g. UniformMZFormulaSampler with min_mz=100, max_mz=500) over an rt_range such as 0-1440, and can export scans via write_mzML.

## Related tools

- **ViMMS 2.0** (Framework whose base Controller class is extended and whose Environment/IndependentMassSpectrometer execute the implemented fragmentation strategy in simulation) — https://github.com/glasgowcompbio/vimms
- **Python** (Implementation language for the controller subclass and _process_scan override (requires Python 3.0 or higher))
- **numpy** (Dependency used for peak intensity sorting and numerical handling in the simulation pipeline)
- **Thermo Fisher IAPI** (Instrument API via which simulated controllers can later be run unchanged on real Thermo Tribrid Fusion instruments through bridging code) — https://github.com/thermofisherlsms/iapi

## Examples

```
from vimms.Controller import Controller; class TopNController(Controller): ...  # override _process_scan, return [MS2 Scan Parameters] + [MS1]; env = Environment(mass_spec, controller, rt_range=(0, 1440))
```

## Evaluation signals

- The controller subclass returns a task list from _process_scan containing exactly N MS2 Scan Parameters per MS1 cycle, followed by an MS1 survey scan.
- Selected precursors respect dynamic exclusion: no ion within an already-fragmented precursor's mz_tol and rt_tol exclusion window is re-selected.
- No precursor below min_ms1_intensity (1.75E5 in the paper's quick-run example; 1E4 for higher-accuracy runs) is fragmented.
- Running the controller with an IndependentMassSpectrometer in an Environment over the ChemicalMixtureCreator test mixture completes without error and the Environment can write valid mzML output via write_mzML.

## Limitations

- Setting min_ms1_intensity to 0 includes all ROIs, including noise, causing very slow runs and high memory usage; the 1.75E5 threshold in the paper's example is deliberately high so the notebook completes quickly, at the cost of simulation accuracy.
- No validation of ViMMS 2.0 itself is presented in this paper; utility validation via bridging to real instruments is deferred to Davies et al. (2021).
- No quantitative performance benchmarks are reported in the paper; only modular extensibility claims are made.
- The paper does not state an explicit ViMMS 2.0 version number or provide a changelog for the repository.

## Evidence

- [abstract] Fragmentation strategies are implemented as controllers that extend from the base Controller class: "Fragmentation strategies are implemented
as controllers that extend from the base Controller class"
- [methods] Top-N is the standard DDA strategy implemented via controllers: "A standard fragmentation strategy used in data-dependant acquisition (DDA) is the Top-N strategy"
- [methods] Environment runs mass spec and controller: "create an environment to run both the mass spec and controller"
- [results] min_intensity threshold 1.75E5 set high to reduce processing time: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [readme] README: ViMMS simulates MS2 acquisition scan-level control: [evidence span withheld — anchor status: external_doc]

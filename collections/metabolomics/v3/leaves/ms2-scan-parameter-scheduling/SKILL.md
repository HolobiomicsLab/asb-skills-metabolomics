---
name: ms2-scan-parameter-scheduling
description: Use when you need to simulate or prototype a Top-N DDA acquisition method
  in ViMMS 2.0 — i.e. you have simulated chemicals or mzML-derived ROIs and must decide
  which MS1 precursors get fragmented in which MS2 scans at each point in the run.
when_to_use_negative:
- You want a data-independent acquisition (DIA) method such as All-ion-fragmentation
  or SWATH-MS — those are separate filter steps/controllers in ViMMS, not Top-N DDA
  scheduling.
- You need an adaptive/exclusion-weighted variant — use the provided SmartROIController
  or WeightedDEWController rather than writing a plain Top-N loop.
- You only need to replay existing acquisitions without changing the fragmentation
  strategy — ViMMS can virtually replay a previous LC-MS/MS analysis without implementing
  a new controller.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ViMMS 2.0
  role: Framework providing the base Controller class, Scan Parameters, Environment
    and IndependentMassSpectrometer used to implement and run the Top-N DDA scheduling
    controller
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which the TopNController subclass and _process_scan override are
    written (Python 3.0 or higher)
- name: Thermo Fisher IAPI
  role: Bridging layer that allows the same controller (scheduled Scan Parameters)
    to run unchanged on a real Thermo Tribrid Fusion instrument
  repo: https://github.com/thermofisherlsms/iapi
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:8b9cf437dd639895b82e0aa01225b56e02e3acf9b6d96a74c3ba601413253e22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - Thermo Fisher IAPI
  - ViMMS 2.0
  - numpy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ms2-scan-parameter-scheduling

## Summary

Implements a Top-N DDA fragmentation strategy as a ViMMS 2.0 controller: subclass the base Controller class, override _process_scan to select the N most intense MS1 peaks (honouring dynamic exclusion, mz_tol/rt_tol and min_ms1_intensity), and return scheduled MS2 tasks as Scan Parameters that the Environment executes on an IndependentMassSpectrometer.

## When to use

Use when you need to simulate or prototype a Top-N DDA acquisition method in ViMMS 2.0 — i.e. you have simulated chemicals or mzML-derived ROIs and must decide which MS1 precursors get fragmented in which MS2 scans at each point in the run.

## When NOT to use

- You want a data-independent acquisition (DIA) method such as All-ion-fragmentation or SWATH-MS — those are separate filter steps/controllers in ViMMS, not Top-N DDA scheduling.
- You need an adaptive/exclusion-weighted variant — use the provided SmartROIController or WeightedDEWController rather than writing a plain Top-N loop.
- You only need to replay existing acquisitions without changing the fragmentation strategy — ViMMS can virtually replay a previous LC-MS/MS analysis without implementing a new controller.

## Inputs

- MS1 survey scan (from an IndependentMassSpectrometer run in an Environment)
- Simulated chemical mixture generated with ChemicalMixtureCreator (e.g. with UniformMZFormulaSampler, min_mz=100, max_mz=500) or ROIs extracted from an existing mzML file
- Controller parameters: N (number of MS2 per cycle), isolation_window width, mz_tol, rt_tol, min_ms1_intensity

## Outputs

- List of Scan Parameters (scheduled MS2 fragmentation tasks plus follow-up MS1 survey scans) returned to the Environment
- Simulated MS/MS scan results exportable as an mzML file via Environment.write_mzML

## How to apply

Define a TopNController subclass of the base Controller class imported from vimms.Controller. Override _process_scan so that, given an incoming MS1 survey scan, peaks are sorted by intensity and the N most intense ions that are not dynamically excluded are picked, honouring the mz_tol and rt_tol exclusion windows and the min_ms1_intensity threshold (the paper's example uses 1.75E5 to keep simulations fast; lowering to e.g. 1E4 is more accurate but slower). For each selected precursor, create MS2 scan parameters with the chosen isolation_window width and append them to the pending task list. Schedule a follow-up MS1 survey scan after the N MS2 tasks, then return the complete task list to the Environment, which runs it against an IndependentMassSpectrometer (e.g. over an rt_range of 0–1440) and can write the resulting scans to mzML via Environment.write_mzML.

## Related tools

- **ViMMS 2.0** (Framework providing the base Controller class, Scan Parameters, Environment and IndependentMassSpectrometer used to implement and run the Top-N DDA scheduling controller) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which the TopNController subclass and _process_scan override are written (Python 3.0 or higher))
- **Thermo Fisher IAPI** (Bridging layer that allows the same controller (scheduled Scan Parameters) to run unchanged on a real Thermo Tribrid Fusion instrument) — https://github.com/thermofisherlsms/iapi

## Examples

```
from vimms.Controller import TopNController, Controller; env = Environment(mass_spec, my_topN_controller); env.run(); env.write_mzML('output.mzML')
```

## Evaluation signals

- The controller produces cycles of exactly N MS2 scan tasks followed by one MS1 survey scan in the returned task list.
- Selected precursors all have intensity above the min_ms1_intensity threshold (e.g. 1.75E5) and none is currently within the dynamic exclusion window defined by mz_tol/rt_tol.
- Each scheduled MS2 task carries the chosen isolation_window width and the correct precursor m/z from the parent MS1 scan.
- Running the Environment completes over the full rt_range (0–1440) and Environment.write_mzML emits a valid mzML file whose scan cycle structure matches the Top-N pattern.
- Simulation results evaluate successfully with ViMMS's provided evaluation methods.

## Limitations

- The min_ms1_intensity threshold trades speed for fidelity: 1.75E5 was set high only so the notebook completes quickly; lowering to e.g. 1E4 is more accurate but makes simulation runs much slower with higher memory usage, and setting it to 0 includes even noise ROIs.
- This paper itself presents no validation or benchmark results for ViMMS 2.0; utility validation via bridging is reported separately in Davies et al. (2021).
- Running the same controller on a physical Thermo Tribrid Fusion instrument requires the Thermo Fisher IAPI bridging code and is outside the pure in-silico workflow.

## Evidence

- [abstract] Fragmentation strategies are implemented as controllers that extend from the base Controller class: "Fragmentation strategies are implemented
as controllers that extend from the base Controller class"
- [methods] Top-N as standard DDA strategy: "A standard fragmentation strategy used in data-dependant acquisition (DDA) is the Top-N strategy"
- [methods] create environment to run mass spec and controller: "create an environment to run both the mass spec and controller"
- [results] min_intensity threshold 1.75E5 set high for speed: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [results] lowering to 1E4 gives more accurate but slower simulation: "you should lower this threshold value to e.g. 1E4. However this results in more objects to be processed, which will make the simulation runs much slower at higher memory usage"
- [methods] Environment provides write_mzML export: "The `Environment` class provides `write_mzML` to export the generated scans"
- [readme] ViMMS README: simulate fragmentation strategies in metabolomics: "a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics"

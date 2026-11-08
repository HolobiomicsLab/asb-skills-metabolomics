---
name: topn-precursor-selection-scoring
description: Use when a simulated LC-MS/MS acquisition must follow Top-N DDA logic
  — ranking MS1 peaks by intensity, honouring dynamic exclusion and an MS1 intensity
  threshold — to schedule MS2 fragmentation tasks in ViMMS 2.0.
when_to_use_negative:
- The desired acquisition is data-independent (DIA) such as All-ion-fragmentation
  or SWATH-MS — these are separate ViMMS controller strategies, not Top-N precursor
  ranking
- The goal is dynamic-exclusion-weighted or ROI-guided DDA behaviour better served
  by the provided WeightedDEWController or SmartROIController variants
- You need to run on a physical instrument without bridging — this skill is in-silico;
  execution on Thermo Tribrid Fusion instruments requires the additional IAPI bridging
  code
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3364
tools:
- name: ViMMS 2.0
  role: Framework hosting the base Controller class whose _process_scan is overridden
    to implement Top-N precursor selection, plus the Environment that executes the
    returned Scan Parameters
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Implementation language for the TopNController subclass and _process_scan
    override
- name: numpy
  role: Dependency of ViMMS used for peak intensity sorting and numeric handling during
    precursor ranking
- name: Thermo Fisher IAPI
  role: Bridging layer that would let the same simulated controller run unchanged
    on a Thermo Tribrid Fusion instrument
  repo: https://github.com/thermofisherlsms/iapi
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:395531a2ae4bc2cd6068ae51da56b0afd49920df60b7c70ad6d4dd7a4a5f08f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - Thermo Fisher IAPI
  - ViMMS 2.0
  - numpy
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3364
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# topn-precursor-selection-scoring

## Summary

Implements the standard Top-N DDA fragmentation strategy as a ViMMS 2.0 controller: given each incoming MS1 survey scan, score/select the N most intense non-excluded precursor ions and emit MS2 Scan Parameters for simulation.

## When to use

Use when a simulated LC-MS/MS acquisition must follow Top-N DDA logic — ranking MS1 peaks by intensity, honouring dynamic exclusion and an MS1 intensity threshold — to schedule MS2 fragmentation tasks in ViMMS 2.0.

## When NOT to use

- The desired acquisition is data-independent (DIA) such as All-ion-fragmentation or SWATH-MS — these are separate ViMMS controller strategies, not Top-N precursor ranking
- The goal is dynamic-exclusion-weighted or ROI-guided DDA behaviour better served by the provided WeightedDEWController or SmartROIController variants
- You need to run on a physical instrument without bridging — this skill is in-silico; execution on Thermo Tribrid Fusion instruments requires the additional IAPI bridging code

## Inputs

- Simulated chemical mixture created with ChemicalMixtureCreator plus a formula sampler such as UniformMZFormulaSampler (min_mz=100, max_mz=500), or chemicals extracted from existing mzML files
- MS1 survey scans produced by an IndependentMassSpectrometer during simulation
- Controller configuration: N (number of MS2 per MS1), isolation_window width, mz_tol, rt_tol, min_ms1_intensity

## Outputs

- Per-scan list of Scan Parameters (MS2 fragmentation tasks plus scheduled MS1 survey scans) returned from _process_scan
- Simulated MS2 fragmentation spectra executed by the Environment
- mzML file of generated scans exported via Environment.write_mzML

## How to apply

Subclass the base Controller class from vimms.Controller and override _process_scan, which is invoked per incoming MS1 survey scan. In _process_scan, filter peaks below min_ms1_intensity, sort the remaining peaks by intensity, and pick the N most intense ions that are not within an existing dynamic exclusion window (mz_tol for m/z, rt_tol for retention time). For each selected precursor, build MS2 scan parameters with the chosen isolation_window width and append them to the pending task list, then schedule a follow-up MS1 survey scan after the N MS2 tasks and return the full task list to the Environment. Generate the test chemical input with ChemicalMixtureCreator and UniformMZFormulaSampler (min_mz=100, max_mz=500), wire the controller to an IndependentMassSpectrometer inside an Environment over an rt_range of 0–1440, and run the simulation. Note the min_intensity threshold 1.75E5 used in the paper's examples is deliberately very high to keep runs fast; lower it (e.g. 1E4) for a more accurate but slower simulation.

## Related tools

- **ViMMS 2.0** (Framework hosting the base Controller class whose _process_scan is overridden to implement Top-N precursor selection, plus the Environment that executes the returned Scan Parameters) — https://github.com/glasgowcompbio/vimms
- **Python** (Implementation language for the TopNController subclass and _process_scan override)
- **numpy** (Dependency of ViMMS used for peak intensity sorting and numeric handling during precursor ranking)
- **Thermo Fisher IAPI** (Bridging layer that would let the same simulated controller run unchanged on a Thermo Tribrid Fusion instrument) — https://github.com/thermofisherlsms/iapi

## Examples

```
from vimms.Controller import TopNController
controller = TopNController(ionisation_mode, N=10, isolation_width=0.7, mz_tol=0.01, rt_tol=15, min_ms1_intensity=1.75E5)
```

## Evaluation signals

- After each MS1 survey scan, the controller's returned task list contains exactly N MS2 scan parameters followed by a scheduled follow-up MS1 scan
- No precursor within an active dynamic exclusion window (mz_tol, rt_tol) is re-selected until its exclusion interval expires
- Peaks below min_ms1_intensity (e.g. 1.75E5 in the paper's fast-run configuration) never appear as selected precursors
- Each generated MS2 task carries the configured isolation_window width around the precursor m/z
- The full simulated acquisition round-trips to a valid mzML file via Environment.write_mzML and reproduces the Top-N fragmentation pattern

## Limitations

- The 1.75E5 min_intensity threshold used in the paper's notebooks is set very high only so the simulation completes quickly; lowering it (e.g. 1E4) gives a more accurate simulation but slower runs and higher memory usage, and setting it to 0 includes noise ROIs as simulation input
- This paper itself reports no validation, benchmarks or quantitative performance data for controllers; validation via bridging onto real instruments is deferred to Davies et al. (2021)
- No explicit ViMMS 2.0 version number is stated in the paper, so behaviour should be pinned against the glasgowcompbio/vimms repository directly

## Evidence

- [abstract] Fragmentation strategies are implemented as controllers that extend from the base Controller class: "Fragmentation strategies are implemented
as controllers that extend from the base Controller class"
- [methods] Top-N is the standard DDA strategy: "A standard fragmentation strategy used in data-dependant acquisition (DDA) is the Top-N strategy"
- [results] min_intensity threshold rationale: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"
- [results] Lowering threshold trade-off: "you should lower this threshold value to e.g. 1E4. However this results in more objects to be processed, which will make the simulation runs much slower"
- [intro] ViMMS purpose statement: "a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics"
- [readme] Scan-level MS2 control simulation: [evidence span withheld — anchor status: external_doc]

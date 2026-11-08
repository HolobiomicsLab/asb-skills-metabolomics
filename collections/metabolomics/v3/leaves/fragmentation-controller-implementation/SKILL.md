---
name: fragmentation-controller-implementation
description: Use when novel MS/MS acquisition strategies must be prototyped in-silico
  and compared against a Top-N DDA baseline before testing on real Thermo Tribrid
  Fusion equipment.
when_to_use_negative:
- The goal is data-independent acquisition (DIA) such as All-ion-fragmentation or
  SWATH-MS rather than DDA-style controller strategies
- The task is real instrument acquisition, not in-silico simulation — instrument bridging
  requires the separate Thermo Fisher IAPI licence agreement and bridging code described
  in Davies et al. (2021)
- No chemical mixture has been prepared upstream; controllers need chemical objects
  to fragment
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ViMMS 2.0
  role: Framework providing the base Controller class, TopNController/SmartROIController/WeightedDEWController,
    IndependentMassSpectrometer, Environment and evaluation methods used to implement
    and benchmark fragmentation strategies
  repo: https://github.com/glasgowcompbio/vimms
- name: Thermo Fisher IAPI
  role: Bridging interface that lets the simulated controllers run unchanged on Thermo
    Tribrid Fusion instruments
  repo: https://github.com/thermofisherlsms/iapi
- name: pandas
  role: Dependency used to assemble per-controller evaluation reports into the aggregated
    comparative_metrics.csv table
- name: numpy
  role: Common dependency of ViMMS 2.0 used in the simulation and evaluation workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:adb565c7459e1e1be684a7573e0c000b738647125858b7112cea01a4c54886c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - IndependentMassSpectrometer
  - SmartROIController
  - Thermo Fisher IAPI
  - TopNController
  - ViMMS 2.0
  - WeightedDEWController
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# fragmentation-controller-implementation

## Summary

Implement DDA fragmentation strategies as controller classes in ViMMS 2.0 by extending the base Controller class (e.g. TopNController, SmartROIController, WeightedDEWController), then run each in a simulated Environment over the same chemical set to benchmark them comparatively.

## When to use

Use when novel MS/MS acquisition strategies must be prototyped in-silico and compared against a Top-N DDA baseline before testing on real Thermo Tribrid Fusion equipment.

## When NOT to use

- The goal is data-independent acquisition (DIA) such as All-ion-fragmentation or SWATH-MS rather than DDA-style controller strategies
- The task is real instrument acquisition, not in-silico simulation — instrument bridging requires the separate Thermo Fisher IAPI licence agreement and bridging code described in Davies et al. (2021)
- No chemical mixture has been prepared upstream; controllers need chemical objects to fragment

## Inputs

- Chemical mixture produced via ChemicalMixtureCreator (synthetic sampling, HMDB compounds e.g. hmdb_compounds.p, or extracted from mzML files such as Beer_multibeers_1_T10_POS.mzML)
- Controller implementations (TopNController baseline plus SmartROIController / WeightedDEWController or new subclasses of the base Controller class)
- Simulation parameters including min_intensity ROI threshold (e.g. 1.75E5, 1E4, or 0)

## Outputs

- Simulated scan data exportable as mzML via Environment.write_mzML
- Per-controller evaluation reports of times fragmented, precursor coverage and intensity statistics
- Aggregated comparative table, e.g. comparative_metrics.csv with columns controller, chemical, times fragmented, coverage, intensity summary

## How to apply

Implement the strategy as a Python class extending ViMMS's base Controller class; the framework ships SmartROIController (ROI-guided fragmentation with adaptive scoring) and WeightedDEWController (Top-N variant with dynamic exclusion weighting) as reference enhanced-DDA implementations. Load the chemicals produced upstream (e.g. via ChemicalMixtureCreator or extracted from mzML such as Beer_multibeers_1_T10_POS.mzML), instantiate an IndependentMassSpectrometer with the chosen controller, and run it inside an Environment so the mass spec and controller are executed together. Note that min_intensity thresholds control simulation fidelity: 1.75E5 makes runs fast for prototyping, 1E4 is more accurate but slower, and 0 includes all ROIs including noise. After each run, evaluate fragmentation outcomes (per-precursor times fragmented, coverage and intensity statistics) with the framework's evaluation methods, then aggregate per-controller reports into a single comparative table (e.g. comparative_metrics.csv with controller, chemical, times fragmented, coverage and intensity columns). Rationale: the modularity of ViMMS 2.0 lets controller implementations be swapped and run unchanged on both the simulator and, via Thermo Fisher IAPI bridging, actual Tribrid Fusion instruments.

## Related tools

- **ViMMS 2.0** (Framework providing the base Controller class, TopNController/SmartROIController/WeightedDEWController, IndependentMassSpectrometer, Environment and evaluation methods used to implement and benchmark fragmentation strategies) — https://github.com/glasgowcompbio/vimms
- **Thermo Fisher IAPI** (Bridging interface that lets the simulated controllers run unchanged on Thermo Tribrid Fusion instruments) — https://github.com/thermofisherlsms/iapi
- **pandas** (Dependency used to assemble per-controller evaluation reports into the aggregated comparative_metrics.csv table)
- **numpy** (Common dependency of ViMMS 2.0 used in the simulation and evaluation workflow)

## Examples

```
from vimms.Controller import TopNController, SmartROIController, WeightedDEWController; env = Environment(mass_spec, controller); env.run(); env.write_mzML('out.mzML')
```

## Evaluation signals

- comparative_metrics.csv exists, parses as well-formed CSV, and contains evaluation rows for all three controllers (Top-N, SmartROI, WeightedDEW)
- Per-chemical times_fragmented_summary values are non-negative integers consistent with the framework's reported evaluation metrics
- All controllers were run over the same chemical set, so the coverage/intensity comparison is like-for-like across controllers
- Environment.write_mzML output is produced and parseable as mzML for each controller run
- A newly implemented controller subclasses the base Controller class and runs unchanged inside the same Environment loop as the built-in controllers

## Limitations

- min_intensity threshold 1.75E5 was set high only so demo notebooks complete quickly; 1E4 is more accurate but slower with higher memory use, and 0 includes all ROIs including noise
- No quantitative validation or benchmark results for ViMMS 2.0 are reported in the paper itself; utility validation is deferred to Davies et al. (2021)
- No explicit software version number is stated for ViMMS 2.0 in the article
- Running IAPI-bridged controllers on real instruments requires a Fusion IAPI Software License Agreement

## Evidence

- [abstract] Fragmentation strategies are implemented as controllers that extend from the base Controller class: "Fragmentation strategies are implemented
as controllers that extend from the base Controller class"
- [abstract] demonstrate how novel fragmentation strategies could be rapidly implemented and validated in ViMMS 2.0: "demonstrate how novel fragmentation
strategies could be rapidly implemented and validated in ViMMS 2.0"
- [methods] create an environment to run both the mass spec and controller: "create an environment to run both the mass spec and controller"
- [methods] The `Environment` class provides `write_mzML` to export the generated scans: "The `Environment` class provides `write_mzML` to export the generated scans"
- [readme] a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics: "a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics"

---
name: controller-environment-orchestration
description: Use when you need to simulate a tandem MS acquisition over a set of chemical
  objects in ViMMS 2.0 — e.g. after generating chemicals with ChemicalMixtureCreator
  — and produce scan-level mzML output for a chosen controller.
when_to_use_negative:
- You already have real acquired mzML data and only want to analyse or evaluate it
  — ViMMS Environment orchestration simulates acquisitions rather than processing
  existing files.
- You need to run acquisition on actual hardware without the Thermo Fisher IAPI bridging
  code — simulated controllers run unchanged on instruments only via the IAPI bridge
  to Thermo Tribrid Fusion instruments.
- The fragmentation strategy is not representable as a Controller class extending
  the base Controller class.
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ViMMS 2.0
  role: Framework providing the base Controller class, IndependentMassSpectrometer,
    and Environment.write_mzML used to orchestrate and export simulated acquisitions
  repo: https://github.com/glasgowcompbio/vimms
- name: Thermo Fisher IAPI
  role: Bridging layer that lets the same simulated controllers run unchanged on Thermo
    Tribrid Fusion instruments instead of in the Environment simulator
  repo: https://github.com/thermofisherlsms/iapi
- name: Python
  role: Language in which controllers are implemented and the Environment orchestration
    script is written
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:593ebbebbf4b16e6e7fb12244130400cbf283179a523f05801f9923950a18e65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - Thermo Fisher IAPI
  - ViMMS 2.0
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# controller-environment-orchestration

## Summary

Orchestrates a ViMMS 2.0 simulated LC-MS/MS acquisition by pairing a fragmentation-strategy Controller (e.g. Top-N DDA or AIF DIA) with an IndependentMassSpectrometer inside an Environment, then exporting the generated scans to mzML.

## When to use

Use when you need to simulate a tandem MS acquisition over a set of chemical objects in ViMMS 2.0 — e.g. after generating chemicals with ChemicalMixtureCreator — and produce scan-level mzML output for a chosen controller.

## When NOT to use

- You already have real acquired mzML data and only want to analyse or evaluate it — ViMMS Environment orchestration simulates acquisitions rather than processing existing files.
- You need to run acquisition on actual hardware without the Thermo Fisher IAPI bridging code — simulated controllers run unchanged on instruments only via the IAPI bridge to Thermo Tribrid Fusion instruments.
- The fragmentation strategy is not representable as a Controller class extending the base Controller class.

## Inputs

- Chemical dataset (KnownChemical objects, e.g. produced by ChemicalMixtureCreator or extracted from an existing mzML file)
- Controller configuration parameters (e.g. AIF_params with ms1_source_cid_energy, Top-N settings)
- Retention-time range for the simulated acquisition

## Outputs

- Simulated scan-level LC-MS/MS acquisition written as an mzML file (via Environment.write_mzML)
- Controller-scheduled fragmentation scans (e.g. AIF/DIA scans or Top-N DDA MS2 scans)

## How to apply

Implement or select the fragmentation strategy as a controller class extending ViMMS's base Controller class (e.g. AIF for DIA with AIF_params including ms1_source_cid_energy=25, or a Top-N DDA controller). Build the chemicals input via ChemicalMixtureCreator (or extract from existing mzML), then instantiate an IndependentMassSpectrometer (with the desired polarity, e.g. positive) over those chemicals. Create an Environment to run both the mass spectrometer and the controller over the retention-time range, and after the run call the Environment's write_mzML method to export generated scans to an mzML file. Because controllers extend the same base class, the same controller code runs unchanged in simulation or, via Thermo Fisher IAPI bridging, on a real Thermo Tribrid Fusion instrument.

## Related tools

- **ViMMS 2.0** (Framework providing the base Controller class, IndependentMassSpectrometer, and Environment.write_mzML used to orchestrate and export simulated acquisitions) — https://github.com/glasgowcompbio/vimms
- **Thermo Fisher IAPI** (Bridging layer that lets the same simulated controllers run unchanged on Thermo Tribrid Fusion instruments instead of in the Environment simulator) — https://github.com/thermofisherlsms/iapi
- **Python** (Language in which controllers are implemented and the Environment orchestration script is written)

## Examples

```
from vimms.Controller import AifController; from vimms.MassSpec import IndependentMassSpectrometer; from vimms.Environment import Environment; env = Environment(IndependentMassSpectrometer('positive', chemicals, None), AifController(), min_rt, max_rt); env.run(); env.write_mzML('output.mzML')
```

## Evaluation signals

- The simulated run completes without error and the Environment produces a well-formed mzML file (e.g. loadable by pymzML/pyteomics).
- The exported mzML contains the expected scan pattern for the controller — e.g. AIF/DIA scans fragmenting an m/z range rather than single precursors, or the cyclic MS1/MS2 pattern of Top-N DDA.
- DIA-mode (AIF) output is kept distinct from DDA-mode (Top-N) mzML files when both are generated.
- Controller-scheduled scans reflect the configured parameters (e.g. ms1_source_cid_energy=25 for AIF) in the produced scans.
- The same controller object can be re-used unchanged against a real instrument via IAPI bridging, confirming it extends the base Controller class correctly.

## Limitations

- min_intensity thresholds trade accuracy for speed: 1.75E5 completes quickly, 1E4 is more accurate but much slower with higher memory use, and 0 includes all ROIs including noise.
- No validation or benchmark results for the simulation framework itself are reported in the paper; instrument-bridging validation is deferred to Davies et al. (2021).
- Real-instrument execution of simulated controllers requires a Fusion IAPI Software License Agreement and Thermo Fisher hardware; IAPI use with certain Tune versions requires patches (e.g. Tune 3.5, microscans issue on Tune 4.2/4.3).

## Evidence

- [abstract] Fragmentation strategies are implemented as controllers that extend from the base Controller class: "Fragmentation strategies are implemented
as controllers that extend from the base Controller class"
- [methods] Environment runs mass spec and controller: "create an environment to run both the mass spec and controller"
- [methods] write_mzML export: "The `Environment` class provides `write_mzML` to export the generated scans"
- [discussion] Bridging to real instruments: "additional bridging code that allows
simulated controllers to run unchanged on both the simulator and actual mass spectrometers."
- [abstract] generate chemical inputs: "generate chemical objects (produced synthetically or extracted from existing
mzML files)"

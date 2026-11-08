---
name: mass-spectrometry-dia-acquisition-simulation
description: Use when the analysis goal is to simulate data-independent acquisition
  (e.g. all-ion-fragmentation or SWATH-style) over a chemical mixture, rather than
  Top-N DDA precursor selection, and the input is a set of generated or mzML-derived
  chemical objects.
when_to_use_negative:
- The goal is data-dependent acquisition with precursor selection — use the Top-N
  DDA controller (or SmartROIController/WeightedDEWController variants) instead.
- The target is real instrument acquisition on a Thermo Tribrid Fusion instrument
  via the IAPI bridge — this skill covers in-silico simulation only, and the paper
  defers instrument validation to Davies et al. (2021).
- The input is already an acquired mzML file to analyse rather than a chemical mixture
  to simulate acquisition over.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS 2.0
  role: Provides the AIF controller, IndependentMassSpectrometer, and Environment
    used to simulate the DIA acquisition and write the mzML output.
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which the controllers are implemented and the simulation is run.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:56b9f75c7458da46aad4af3dcbd0a581c85710e49c9b84bedc3d329325463fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - ViMMS 2.0
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mass-spectrometry-dia-acquisition-simulation

## Summary

Simulates data-independent acquisition (DIA) LC-MS/MS runs in ViMMS 2.0 using the all-ion-fragmentation (AIF) controller, which fragments a full m/z range rather than selecting single precursor ions as in DDA. It produces simulated scans exportable to mzML for evaluating DIA-style fragmentation strategies in-silico.

## When to use

Use when the analysis goal is to simulate data-independent acquisition (e.g. all-ion-fragmentation or SWATH-style) over a chemical mixture, rather than Top-N DDA precursor selection, and the input is a set of generated or mzML-derived chemical objects.

## When NOT to use

- The goal is data-dependent acquisition with precursor selection — use the Top-N DDA controller (or SmartROIController/WeightedDEWController variants) instead.
- The target is real instrument acquisition on a Thermo Tribrid Fusion instrument via the IAPI bridge — this skill covers in-silico simulation only, and the paper defers instrument validation to Davies et al. (2021).
- The input is already an acquired mzML file to analyse rather than a chemical mixture to simulate acquisition over.

## Inputs

- KnownChemical objects generated via ChemicalMixtureCreator (e.g. with UniformMZFormulaSampler)
- Chemical objects extracted from an existing mzML file (e.g. Beer_multibeers_1_T10_POS.mzML)
- AIF_params configuration (e.g. ms1_source_cid_energy=25)
- Retention-time range for the simulated run

## Outputs

- Simulated DIA/AIF scan schedule produced by the controller in Environment
- mzML file containing AIF fragmentation scans (written via Environment.write_mzML)

## How to apply

Load the chemical dataset — either KnownChemical objects produced via ChemicalMixtureCreator (with a formula/mz sampler such as UniformMZFormulaSampler) or chemicals extracted from existing mzML files — as the simulation input. Instantiate the AIF controller from vimms.Controller, configured via AIF_params (the demonstrated configuration sets ms1_source_cid_energy to 25), which fragments all ions across an m/z range instead of individual precursors. Bind it to an IndependentMassSpectrometer running in positive polarity, create an Environment to run both the mass spec and the controller over the retention-time range, and let the controller schedule DIA fragmentation scans. Export the generated scans with the Environment's write_mzML method, keeping this DIA-mode mzML separate from any Top-N DDA output. Note that controller intensity thresholds shape run cost: min_intensity 1.75E5 completes quickly, lowering to 1E4 gives a more accurate but slower simulation, and 0 includes all ROIs including noise.

## Related tools

- **ViMMS 2.0** (Provides the AIF controller, IndependentMassSpectrometer, and Environment used to simulate the DIA acquisition and write the mzML output.) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which the controllers are implemented and the simulation is run.)

## Examples

```
from vimms.Controller import AIF; controller = AIF(AIF_params(ms1_source_cid_energy=25)); env = Environment(mass_spec, controller); env.run(); env.write_mzML('dia_aif_output.mzML')
```

## Evaluation signals

- The DIA run completes and produces a well-formed, parseable simulated mzML file.
- The exported mzML contains AIF fragmentation scans, distinguishable from a Top-N DDA run's output.
- MS2 scans fragment across a range of m/z values rather than targeting individual selected precursor ions.
- The output mzML is distinct from (and kept separate from) any DDA-mode mzML produced for the same chemical dataset.

## Limitations

- Validation of the simulator's utility via instrument bridging is reported in a separate publication (Davies et al., 2021), not in this paper.
- min_intensity threshold choices trade accuracy against speed: 1.75E5 completes quickly, 1E4 is more accurate but slower with higher memory usage, and 0 includes all ROIs including noise.
- The paper reports no benchmarks or quantitative performance data; ViMMS 2.0 is positioned as a framework for prototyping strategies, with real-instrument execution requiring additional bridging code.

## Evidence

- [methods] DIA filter step surfaced in methods: "several common data-independent acquisition (DIA) strategies such as All-ion-fragmentation"
- [methods] Environment runs controller and mass spec: "create an environment to run both the mass spec and controller"
- [methods] Environment exports scans to mzML: "The `Environment` class provides `write_mzML` to export the generated scans"
- [readme] ViMMS README on scan-level MS2 simulation: [evidence span withheld — anchor status: external_doc]
- [abstract] Chemical input generation: "generate chemical objects (produced synthetically or extracted from existing mzML files)"

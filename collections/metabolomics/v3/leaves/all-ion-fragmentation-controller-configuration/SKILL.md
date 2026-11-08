---
name: all-ion-fragmentation-controller-configuration
description: Use when simulating DIA-mode LC-MS/MS acquisition over a set of generated
  or extracted chemicals — e.g. you need fragmentation scans covering an m/z range
  rather than single-precursor DDA selections.
when_to_use_negative:
- The goal is data-dependent acquisition targeting single precursor ions — use the
  Top-N DDA controller instead of AIF
- A different DIA strategy is required, e.g. SWATH-MS sequential windowed acquisition
  rather than all-ion fragmentation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ViMMS 2.0
  role: Provides the AIF controller (vimms.Controller) and Environment for simulating
    DIA-mode all-ion fragmentation acquisition over generated chemicals
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which the simulation script (controller, spectrometer, Environment,
    write_mzML export) is written
- name: numpy
  role: Supporting dependency of ViMMS used for numerical handling in the simulation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:1f9925943f5cc6b741dfbcf9f7486f4dce7f81e2dfe451c94140868fc4f24442
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
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# all-ion-fragmentation-controller-configuration

## Summary

Configures and runs ViMMS 2.0's AIF (all-ion fragmentation) controller to simulate data-independent acquisition (DIA), where a range of m/z values is co-isolated and fragmented instead of individual precursors as in Top-N DDA.

## When to use

Use when simulating DIA-mode LC-MS/MS acquisition over a set of generated or extracted chemicals — e.g. you need fragmentation scans covering an m/z range rather than single-precursor DDA selections.

## When NOT to use

- The goal is data-dependent acquisition targeting single precursor ions — use the Top-N DDA controller instead of AIF
- A different DIA strategy is required, e.g. SWATH-MS sequential windowed acquisition rather than all-ion fragmentation

## Inputs

- Chemical dataset of KnownChemical objects generated via ChemicalMixtureCreator (e.g. with UniformMZFormulaSampler)
- AIF_params configuration (with ms1_source_cid_energy, e.g. 25)
- Retention-time range for the simulated run
- Polarity setting (positive) for the IndependentMassSpectrometer

## Outputs

- Simulated mzML file containing AIF/DIA fragmentation scans across the m/z range
- Scheduled DIA fragmentation scans produced by the AIF controller

## How to apply

Load the chemical dataset (e.g. KnownChemical objects from ChemicalMixtureCreator with a UniformMZFormulaSampler) as simulation input, then instantiate the AIF controller from vimms.Controller, which implements the all-ion-fragmentation DIA strategy and is configured via AIF_params with ms1_source_cid_energy set to 25. Pair the controller with an IndependentMassSpectrometer in positive polarity over the loaded chemicals, bind both in an Environment over the retention-time range, and run so the controller schedules DIA fragmentation scans that fragment all ions across the m/z range. Finally, export the generated scans with the Environment's write_mzML method, keeping this DIA-mode mzML separate from any Top-N DDA output so acquisition modes remain distinguishable.

## Related tools

- **ViMMS 2.0** (Provides the AIF controller (vimms.Controller) and Environment for simulating DIA-mode all-ion fragmentation acquisition over generated chemicals) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which the simulation script (controller, spectrometer, Environment, write_mzML export) is written)
- **numpy** (Supporting dependency of ViMMS used for numerical handling in the simulation)

## Examples

```
from vimms.Controller import AIF; from vimms.Common import POSITIVE; env = Environment(mass_spec, AIF(chemicals, AIF_params), progress_bar=True); env.run(); env.write_mzML(out_dir, 'dia_aif.mzML')
```

## Evaluation signals

- The DIA run completes without errors and produces a well-formed simulated mzML file
- The output mzML contains AIF fragmentation scans that fragment an m/z range rather than individually selected precursors
- The DIA-mode mzML is distinct from (and kept separate from) the Top-N DDA mzML output
- The AIF_params show ms1_source_cid_energy set to the intended value (25)
- Scans were generated over the full specified retention-time range by the controller-environment pairing

## Limitations

- Lower simulation thresholds (e.g. min_intensity 1E4 or 0) yield more accurate runs but much slower simulations at higher memory usage; 1.75E5 is fast but coarse
- Validation of ViMMS 2.0's bridging to real instruments is reported in a separate publication (Davies et al., 2021), not in the ViMMS 2.0 paper itself
- AIF is one of several DIA strategies; it does not implement windowed approaches such as SWATH-MS

## Evidence

- [methods] AIF controller configured via AIF_params with ms1_source_cid_energy 25: [evidence span withheld — anchor status: not_found]
- [methods] Filter step: all-ion-fragmentation as a DIA strategy: "several common data-independent acquisition (DIA) strategies such as All-ion-fragmentation"
- [methods] Environment runs controller and writes mzML: "create an environment to run both the mass spec and controller"
- [methods] write_mzML export method: "The `Environment` class provides `write_mzML` to export the generated scans"
- [abstract] Controllers extend the base Controller class: "Fragmentation strategies are implemented as controllers that extend from the base Controller class"

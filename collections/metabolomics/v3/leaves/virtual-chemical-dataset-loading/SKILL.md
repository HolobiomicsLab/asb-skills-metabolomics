---
name: virtual-chemical-dataset-loading
description: Use when a simulated LC-MS/MS run is needed over a pre-generated list
  of chemicals (e.g. KnownChemical objects from ChemicalMixtureCreator with a UniformMZFormulaSampler)
  rather than chemicals extracted from an existing mzML file.
when_to_use_negative:
- The input chemicals should be extracted from an existing real mzML file (e.g. Beer_multibeers_1_T10_POS.mzML)
  rather than loaded from a previously generated chemical dataset
- The goal is to run the strategy on an actual Thermo Tribrid Fusion instrument via
  IAPI bridging rather than in simulation
- The target strategy is ROI-based input extraction from mzML (e.g. extract_hmdb_metabolite
  or ROI extraction with min_intensity thresholds), not replaying generated chemicals
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: ViMMS 2.0
  role: Framework providing ChemicalMixtureCreator, controllers (AIF, Top-N), IndependentMassSpectrometer
    and Environment with write_mzML for the simulated acquisition
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Language in which the controller, mass spectrometer and Environment are instantiated
    and driven
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:0fb1bade49ecefd1f8d5068c982075e03d9938b331d439e177167d4563835831
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# virtual-chemical-dataset-loading

## Summary

Load a previously generated chemical dataset (KnownChemical objects created via ChemicalMixtureCreator) as simulation input into a ViMMS 2.0 controller–mass spectrometer Environment, so that fragmentation strategies such as the AIF DIA controller can be executed in-silico and exported as mzML.

## When to use

Use when a simulated LC-MS/MS run is needed over a pre-generated list of chemicals (e.g. KnownChemical objects from ChemicalMixtureCreator with a UniformMZFormulaSampler) rather than chemicals extracted from an existing mzML file.

## When NOT to use

- The input chemicals should be extracted from an existing real mzML file (e.g. Beer_multibeers_1_T10_POS.mzML) rather than loaded from a previously generated chemical dataset
- The goal is to run the strategy on an actual Thermo Tribrid Fusion instrument via IAPI bridging rather than in simulation
- The target strategy is ROI-based input extraction from mzML (e.g. extract_hmdb_metabolite or ROI extraction with min_intensity thresholds), not replaying generated chemicals

## Inputs

- List of KnownChemical objects generated via ChemicalMixtureCreator (e.g. sampled with a UniformMZFormulaSampler)
- Retention-time range to simulate
- Controller parameters (e.g. AIF_params with ms1_source_cid_energy = 25 for DIA mode)

## Outputs

- Simulated scan set (MS1 and MS2 fragmentation scans) scheduled by the controller
- Well-formed mzML file written via Environment.write_mzML (e.g. a DIA/AIF mzML kept distinct from a Top-N DDA mzML)

## How to apply

Treat the generated KnownChemical objects as the simulation input and instantiate the desired fragmentation strategy as a controller from vimms.Controller (e.g. the AIF controller for DIA all-ion fragmentation, configured via AIF_params with ms1_source_cid_energy set to 25, or a Top-N DDA controller). Bind the controller to an IndependentMassSpectrometer in the required polarity (e.g. positive) over the loaded chemicals. Create an Environment to run both the mass spec and the controller over the retention-time range, then export the generated scans with the Environment's write_mzML method, keeping DIA-mode output separate from DDA-mode output. The modular design means the same chemical dataset can be replayed against different controllers for comparison.

## Related tools

- **ViMMS 2.0** (Framework providing ChemicalMixtureCreator, controllers (AIF, Top-N), IndependentMassSpectrometer and Environment with write_mzML for the simulated acquisition) — https://github.com/glasgowcompbio/vimms
- **Python** (Language in which the controller, mass spectrometer and Environment are instantiated and driven)

## Examples

```
from vimms.Controller import AIF; from vimms.MassSpec import IndependentMassSpectrometer; from vimms.Environment import Environment; env = Environment(IndependentMassSpectrometer('pos', chemicals, None), AIF()); env.run(); env.write_mzML('dia_output.mzML')
```

## Evaluation signals

- The simulated run completes without errors and the Environment produces scan objects for the full retention-time range
- The exported mzML is well-formed and contains the expected fragmentation scans (e.g. AIF/DIA scans fragmenting an m/z range rather than single precursors)
- The DIA-mode mzML is distinct from the separately kept Top-N DDA mzML (different scan schedule/fragmentation pattern)
- Chemical identities in the output traces correspond to the loaded KnownChemical objects from the ChemicalMixtureCreator dataset

## Limitations

- High min_intensity thresholds (e.g. 1.75E5) speed up simulation but reduce accuracy; lowering to ~1E4 is more accurate but much slower with higher memory usage; threshold 0 includes all ROIs including noise
- This paper itself reports no validation of ViMMS 2.0's utility; validation via bridging to real instruments is deferred to Davies et al. (2021)
- No explicit ViMMS 2.0 version number or changelog is stated in the article

## Evidence

- [methods] use the ChemicalMixtureCreator class to produce chemical datasets for simulation: "use the [`Chemical Mixture Creator`][vimms.Chemicals.ChemicalMixtureCreator] class to produce chemical datasets for simulation"
- [methods] create an environment to run both the mass spec and controller: "create an environment to run both the mass spec and controller"
- [methods] The Environment class provides write_mzML to export the generated scans: "The `Environment` class provides `write_mzML` to export the generated scans"
- [methods] several common data-independent acquisition (DIA) strategies such as All-ion-fragmentation: "several common data-independent acquisition (DIA) strategies such as All-ion-fragmentation"
- [readme] a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics: "a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics"

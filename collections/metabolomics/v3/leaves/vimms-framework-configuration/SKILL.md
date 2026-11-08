---
name: vimms-framework-configuration
description: 'Use when you need to stand up or reconfigure a ViMMS 2.0 simulation:
  installing the framework in a Python 3.0+ virtualenv, choosing a chemical source
  (HMDB-sampled synthetic or UnknownChemical extraction from an existing mzML like
  Beer_multibeers_1_T10_POS.'
when_to_use_negative:
- Input is an experimental mzML you want to analyze directly for metabolite identification
  — ViMMS is a simulator of the acquisition process, not a data-analysis pipeline.
- You need to run fragmentation strategies unchanged on an actual Thermo Tribrid Fusion
  instrument — that requires the separate IAPI bridging code (thermofisherlsms/iapi)
  rather than the simulator configuration alone.
- A validated pickled chemical list already exists for your mzML — regenerating chemicals
  via ChemicalMixtureFromMZML is redundant unless the ROI threshold or source file
  changed.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: ViMMS 2.0
  role: 'The framework being configured: modular, extensible Python package whose
    chemical sources, controllers, mass spec and Environment components are assembled
    to simulate fragmentation strategies.'
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Runtime language required by ViMMS (Python 3.0 or higher); controllers extend
    base Controller classes in Python.
- name: numpy / scipy / pandas
  role: Common dependency packages installed with the framework that back the chemical
    and spectral simulation machinery.
- name: Thermo Fisher IAPI
  role: Optional bridging layer allowing controllers configured in simulation to run
    unchanged on Thermo Tribrid Fusion instruments.
  repo: https://github.com/thermofisherlsms/iapi
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:f488d972d87cf317aa6cf13b723da2f06fa8023c07efd3d88f67757fa9318b43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - Thermo Fisher IAPI
  - ViMMS 2.0
  - numpy
  - numpy / scipy / pandas
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# vimms-framework-configuration

## Summary

Configure and set up the ViMMS 2.0 Python framework to simulate tandem MS fragmentation strategies in LC-MS metabolomics, from dependency installation through chemical-source selection (synthetic sampling or mzML ROI extraction) to running controllers in an Environment and exporting mzML.

## When to use

Use when you need to stand up or reconfigure a ViMMS 2.0 simulation: installing the framework in a Python 3.0+ virtualenv, choosing a chemical source (HMDB-sampled synthetic or UnknownChemical extraction from an existing mzML like Beer_multibeers_1_T10_POS.mzML), and wiring a controller (Top-N, SmartROI, WeightedDEW) into an Environment.

## When NOT to use

- Input is an experimental mzML you want to analyze directly for metabolite identification — ViMMS is a simulator of the acquisition process, not a data-analysis pipeline.
- You need to run fragmentation strategies unchanged on an actual Thermo Tribrid Fusion instrument — that requires the separate IAPI bridging code (thermofisherlsms/iapi) rather than the simulator configuration alone.
- A validated pickled chemical list already exists for your mzML — regenerating chemicals via ChemicalMixtureFromMZML is redundant unless the ROI threshold or source file changed.

## Inputs

- Python 3.0+ virtual environment (installed via poetry)
- Chemical source: HMDB compound database (e.g. hmdb_compounds.p) for synthetic sampling, or an existing mzML file (e.g. Beer_multibeers_1_T10_POS.mzML) for ROI-based UnknownChemical extraction
- Optional ground-truth feature list, e.g. multibeers_2_aligned.csv from MZmine alignment
- Fragmentation strategy defined as a Controller subclass (Top-N, WeightedDEWController, SmartROIController, AIF, SWATH-MS)

## Outputs

- Pickled list of chemical objects (synthetic chemicals or UnknownChemical objects with m/z, RT and chromatogram attributes from extracted ROIs)
- Simulated MS2 scans exported as an mzML file via Environment.write_mzML
- Evaluation results from ViMMS's built-in evaluation methods

## How to apply

Clone the framework (git clone https://github.com/glasgowcompbio/vimms.git), create a fresh virtual environment, and install all required packages with `poetry install`; ViMMS requires Python 3.0 or higher and depends on common packages such as numpy, scipy and pandas. Decide the chemical source: use ChemicalMixtureCreator to produce synthetic chemical datasets (compounds sampled from databases such as HMDB, e.g. hmdb_compounds.p), or instantiate ChemicalMixtureFromMZML to extract ROIs from an existing mzML file and convert each picked ROI into an UnknownChemical, then call sample() with ms_levels=2 (n_chemicals=None to keep all ROIs) and persist the list with save_obj. Implement or select a fragmentation strategy as a controller class extending the base Controller class — Top-N DDA, All-ion-fragmentation, SWATH-MS, WeightedDEWController, or SmartROIController. Create an Environment to run both the mass spec and the controller, and use Environment.write_mzML to export the generated scans. Tune the min_intensity ROI threshold deliberately: 1.75E5 was set high in the paper's notebooks so runs complete quickly, 1E4 gives a more accurate simulation but slower runs and higher memory use, and 0 includes all ROIs including noise.

## Related tools

- **ViMMS 2.0** (The framework being configured: modular, extensible Python package whose chemical sources, controllers, mass spec and Environment components are assembled to simulate fragmentation strategies.) — https://github.com/glasgowcompbio/vimms
- **Python** (Runtime language required by ViMMS (Python 3.0 or higher); controllers extend base Controller classes in Python.)
- **numpy / scipy / pandas** (Common dependency packages installed with the framework that back the chemical and spectral simulation machinery.)
- **Thermo Fisher IAPI** (Optional bridging layer allowing controllers configured in simulation to run unchanged on Thermo Tribrid Fusion instruments.) — https://github.com/thermofisherlsms/iapi

## Examples

```
git clone https://github.com/glasgowcompbio/vimms.git && cd vimms && poetry install
```

## Evaluation signals

- The output pickle loads and its entries are UnknownChemical objects with m/z, RT and chromatogram attributes populated from the extracted ROIs (for the ChemicalMixtureFromMZML path).
- The Environment runs the configured controller without errors and the generated scans are exported by write_mzML as a readable mzML file.
- Evaluation of simulation results succeeds using the methods ViMMS provides for this purpose.
- min_intensity threshold behaves as documented: 1.75E5 runs fast, 1E4 is more accurate but slower with higher memory usage, 0 includes noise ROIs.
- Dependencies resolve cleanly under poetry install in a new virtual environment (numpy, scipy, pandas present, Python >= 3.0).

## Limitations

- min_intensity is a speed/accuracy trade-off: 1.75E5 was chosen in the paper's notebooks only to make runs complete quickly; lowering it to 1E4 makes simulation much slower at higher memory usage, and 0 includes all ROIs including noise.
- No validation of ViMMS 2.0 is presented in this paper — utility validation via bridging code on real instruments is deferred to Davies et al. (2021).
- No explicit software version number is stated for ViMMS 2.0 in the paper's discussion, and no changelog is documented for the repository.
- The framework requires Python 3.0 or higher and its modular components are meant to be swapped out, so configurations are code-level (controllers, chemical sources) rather than declarative config files.

## Evidence

- [methods] installation via poetry: "create a new virtual environment and install all required packages: $ poetry install"
- [methods] Environment runs controller and writes mzML: "create an environment to run both the mass spec and controller"
- [methods] write_mzML export: "The `Environment` class provides `write_mzML` to export the generated scans"
- [abstract] chemical sources: "generate chemical objects (produced synthetically or extracted from existing mzML files)"
- [results] min_intensity trade-off: "you should lower this threshold value to e.g. 1E4. However this results in more objects to be processed, which will make the simulation runs much slower"
- [readme] controller design: "Fragmentation strategies are implemented as controllers that extend from the base Controller class"

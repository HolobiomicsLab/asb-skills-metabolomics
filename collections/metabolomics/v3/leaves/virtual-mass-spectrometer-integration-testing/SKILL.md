---
name: virtual-mass-spectrometer-integration-testing
description: Use when you need to prototype, compare, or optimise an LC-MS/MS acquisition
  strategy (Top-N DDA, DIA, SWATH-MS) in silico before instrument time, or verify
  a controller can move unchanged from simulation to a Thermo Tribrid Fusion mass
  spectrometer.
when_to_use_negative:
- You already have real acquired LC-MS/MS data and only need downstream processing
  rather than simulating the acquisition process itself
- You need published validation benchmarks of the simulator — this paper defers utility
  validation to Davies et al. (2021)
- You target instruments outside the Thermo Tribrid/Exactive/Exploris series, since
  the bridging path relies on Thermo Fisher IAPI
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ViMMS 2.0
  role: Framework that simulates tandem MS acquisition, hosts controller classes in
    an Environment, and writes simulated scans to mzML
  repo: https://github.com/glasgowcompbio/vimms
- name: Thermo Fisher IAPI
  role: Bridging layer through which simulated controllers run unchanged on Thermo
    Tribrid Fusion instruments for integration testing
  repo: https://github.com/thermofisherlsms/iapi
- name: vimms-data
  role: Repository providing example data (e.g. example_data.zip, hmdb_compounds.p)
    used as simulation input
  repo: https://github.com/glasgowcompbio/vimms-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:98c4ae264d3becb9f846a6f0bde0acd88146215e51bf76c6db6abade2a60b9fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/glasgowcompbio/vimms
  tools:
  - Python
  - Thermo Fisher IAPI
  - ViMMS 2.0
  - numpy
  - vimms-data
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Virtual mass spectrometer integration testing

## Summary

Simulate LC-MS/MS fragmentation strategies end-to-end in ViMMS 2.0 by running controller classes against a virtual mass spectrometer inside an Environment, then export the resulting scans as mzML — with the same controller code reusable, via Thermo Fisher IAPI bridging, on a real Thermo Tribrid Fusion instrument.

## When to use

Use when you need to prototype, compare, or optimise an LC-MS/MS acquisition strategy (Top-N DDA, DIA, SWATH-MS) in silico before instrument time, or verify a controller can move unchanged from simulation to a Thermo Tribrid Fusion mass spectrometer.

## When NOT to use

- You already have real acquired LC-MS/MS data and only need downstream processing rather than simulating the acquisition process itself
- You need published validation benchmarks of the simulator — this paper defers utility validation to Davies et al. (2021)
- You target instruments outside the Thermo Tribrid/Exactive/Exploris series, since the bridging path relies on Thermo Fisher IAPI

## Inputs

- Chemical mixture generated via ChemicalMixtureCreator (synthetic sampling, e.g. UniformMZFormulaSampler with min_mz=100, max_mz=500) or extracted from existing mzML files
- Existing mzML data files, e.g. Beer_multibeers_1_T10_POS.mzML, for replaying prior LC-MS/MS analyses
- Compound database, e.g. hmdb_compounds.p (HMDB-derived)
- Controller subclass implementing the fragmentation strategy (e.g. TopNController, SmartROIController, WeightedDEWController)
- Acquisition parameters: N, min_ms1_intensity, mz_tol, rt_tol, isolation_window, rt_range

## Outputs

- List of Scan Parameters tasks returned by _process_scan (MS2 fragmentation scans plus scheduled MS1 survey scans)
- Simulated MS1/MS2 scans exported as an mzML file via Environment.write_mzML
- Evaluation of simulation results using ViMMS's evaluation methods

## How to apply

Implement the fragmentation strategy as a Python class extending the base Controller class from vimms.Controller and override _process_scan: given an incoming MS1 survey scan, sort peaks by intensity, pick the N most intense ions that pass the min_ms1_intensity threshold and are not under dynamic exclusion (honouring the mz_tol and rt_tol windows), and return a list of Scan Parameters — MS2 scan tasks with the chosen isolation_window, followed by a scheduled follow-up MS1 scan. Generate a test chemical dataset with ChemicalMixtureCreator (e.g. via UniformMZFormulaSampler with min_mz=100, max_mz=500) or extract chemicals from existing mzML files such as Beer_multibeers_1_T10_POS.mzML, then wire the controller to an IndependentMassSpectrometer inside an Environment (e.g. rt_range 0–1440) and run the simulation. Use min_ms1_intensity=1.75E5 for fast runs, or lower it to 1E4 for a more accurate but slower simulation. Export the simulated scans with Environment.write_mzML and evaluate the acquisition results with ViMMS's evaluation methods. For integration testing on hardware, the same controller object can be handed to bridging code using Thermo Fisher IAPI to drive a Tribrid Fusion instrument without code changes.

## Related tools

- **ViMMS 2.0** (Framework that simulates tandem MS acquisition, hosts controller classes in an Environment, and writes simulated scans to mzML) — https://github.com/glasgowcompbio/vimms
- **Thermo Fisher IAPI** (Bridging layer through which simulated controllers run unchanged on Thermo Tribrid Fusion instruments for integration testing) — https://github.com/thermofisherlsms/iapi
- **vimms-data** (Repository providing example data (e.g. example_data.zip, hmdb_compounds.p) used as simulation input) — https://github.com/glasgowcompbio/vimms-data

## Examples

```
from vimms.Controller import TopNController; from vimms.Environment import Environment; env = Environment(mass_spec, controller, min_time=0, max_time=1440); env.run(); env.write_mzML('out.mzML')
```

## Evaluation signals

- For each MS1 survey scan, _process_scan returns a list of Scan Parameters containing at most N MS2 tasks followed by one scheduled MS1 scan
- No selected precursor falls inside the mz_tol/rt_tol dynamic-exclusion window of an already fragmented ion, and all selected precursors exceed the min_ms1_intensity threshold
- The run's output mzML file opens in standard mzML tooling and reproduces the expected MS1–MS2 scan ordering
- The identical controller object executes without modification in both the ViMMS simulator and (via IAPI bridging) on a Thermo Tribrid Fusion instrument
- Evaluation methods report non-degenerate acquisition behaviour (MS2 scans actually scheduled), distinguishing a genuine run from a trivial pass-through

## Limitations

- min_ms1_intensity=1.75E5 is set very high only so example notebooks complete quickly; lowering to 1E4 gives a more accurate simulation but much slower runs and higher memory usage, and a threshold of 0 includes even noise ROIs
- This paper reports no validation, benchmarks, or quantitative performance data for ViMMS 2.0 itself; validation is deferred to Davies et al. (2021)
- No changelog or explicit software version number is stated for ViMMS 2.0 in the paper
- Thermo IAPI is provided as-is for scholarly research with no guaranteed ongoing maintenance; certain Tune versions (3.5, and 4.2/4.3 microscans issue) require patches

## Evidence

- [abstract] ViMMS 2.0 as in-silico framework: "ViMMS 2.0 is a software framework that can be used to develop new fragmentation strategies in metabolomics completely in-silico"
- [abstract] Controllers as the strategy interface: "Fragmentation strategies are implemented as controllers that extend from the base Controller class"
- [methods] Running the simulation: "create an environment to run both the mass spec and controller"
- [discussion] Simulator-to-instrument bridging: "additional bridging code that allows simulated controllers to run unchanged on both the simulator and actual mass spectrometers."
- [readme] README scope statement: "a flexible and modular framework designed to simulate fragmentation strategies in tandem mass spectrometry-based metabolomics."

---
name: simulated-mzml-export
description: Use when a ViMMS 2.0 Environment run (TopNController DDA or AIF DIA simulation
  over generated chemicals) has finished and the resulting scan list must be persisted
  as a well-formed mzML document for downstream evaluation.
when_to_use_negative:
- The scans originate from a real instrument acquisition rather than a ViMMS simulation
  — write_mzML only exports scans accumulated by the Environment.
- The controller has not yet been run via env.run(), so no scan list exists to export.
- The goal is to modify or rewrite an existing real-data mzML file (e.g. Beer_multibeers_1_T10_POS.mzML)
  rather than export a simulated run.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ViMMS 2.0
  role: Framework whose Environment class provides write_mzML to export the generated
    scans from a simulated LC-MS/MS run
  repo: https://github.com/glasgowcompbio/vimms
- name: Python
  role: Runtime in which the ViMMS Environment, controllers and write_mzML export
    are executed
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.21105/joss.03990
    title: 'ViMMS 2.0: A framework to develop, test and optimise fragmentation strategies
      in LC-MS metabolomics'
  source_content_hash: sha256:c080e09cb34dcaa8cf1ff1e8b9ee0da879d4d6c8091f8cba6ac8b7c0d517f5c1
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
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# simulated-mzml-export

## Summary

Export scans accumulated during a ViMMS 2.0 simulated LC-MS/MS run (e.g. Top-N DDA or AIF/DIA) to a standard mzML file using the Environment's write_mzML method, producing a loadable downstream artifact of the in-silico acquisition.

## When to use

Use when a ViMMS 2.0 Environment run (TopNController DDA or AIF DIA simulation over generated chemicals) has finished and the resulting scan list must be persisted as a well-formed mzML document for downstream evaluation.

## When NOT to use

- The scans originate from a real instrument acquisition rather than a ViMMS simulation — write_mzML only exports scans accumulated by the Environment.
- The controller has not yet been run via env.run(), so no scan list exists to export.
- The goal is to modify or rewrite an existing real-data mzML file (e.g. Beer_multibeers_1_T10_POS.mzML) rather than export a simulated run.

## Inputs

- ViMMS Environment object holding the completed scan list from a simulated run (IndependentMassSpectrometer + controller over KnownChemical objects)
- Target output filename/path for the mzML file (e.g. simulated_topn_run.mzML)

## Outputs

- A well-formed simulated mzML file containing the MS1/MS2 scans generated during the run
- Separate mzML files per acquisition mode (Top-N DDA vs AIF DIA) when multiple controllers were run

## How to apply

After env.run() completes over the IndependentMassSpectrometer and its controller (e.g. Top-N DDA with N=3, isolation window 1, mz_tol 10, rt_tol 15, min_ms1_intensity 1.75E5, over an RT range of (0, 1440) minutes), call the Environment's write_mzML capability to write the accumulated scan list, e.g. to simulated_topn_run.mzML. Keep outputs from different acquisition modes separate — e.g. write the AIF/DIA run (AIF_params with ms1_source_cid_energy 25) to its own mzML distinct from the Top-N DDA mzML. Validate that the file exists, parses as a standard mzML mass-spectrometry document, and contains scans spanning the full simulated RT range with MS2 events matching the configured controller schedule (N=3 Top-N MS2 events for DDA, all-ion fragmentation scans for AIF).

## Related tools

- **ViMMS 2.0** (Framework whose Environment class provides write_mzML to export the generated scans from a simulated LC-MS/MS run) — https://github.com/glasgowcompbio/vimms
- **Python** (Runtime in which the ViMMS Environment, controllers and write_mzML export are executed)

## Examples

```
env = Environment(ms1, controller, min_time=0, max_time=1440, progress_bar=True); env.run(); env.write_mzML('simulated_topn_run.mzML')
```

## Evaluation signals

- The mzML file exists and is loadable as a standard mzML mass-spectrometry document (e.g. via pyteomics/pymzML readers).
- The exported scans cover the full simulated retention-time range configured for the Environment (e.g. (0, 1440) minutes).
- The MS2 events in the file match the configured acquisition schedule — e.g. Top-N with N=3 fragmentation events per MS1 cycle for the DDA run.
- AIF/DIA-mode output is stored as a separate mzML file, distinct from the Top-N DDA mzML, and contains all-ion-fragmentation scans rather than single-precursor MS2.
- min_ms1_intensity used in the run is documented (1.75E5 for fast runs; lowering to ~1E4 yields more scans and a larger mzML).

## Limitations

- Setting min_intensity threshold to 0 includes all ROIs including noise, which slows simulation and increases memory usage — the resulting mzML will contain many more low-intensity events.
- The 1.75E5 min_intensity threshold used in examples is set high only to reduce processing time; the exported mzML is therefore a simplified, less accurate simulation unless the threshold is lowered to e.g. 1E4.
- No explicit ViMMS 2.0 version number is stated in the paper, so reproducibility of the exported mzML depends on pinning the repository revision.

## Evidence

- [methods] The Environment class provides write_mzML to export the generated scans: "The `Environment` class provides `write_mzML` to export the generated scans"
- [methods] create an environment to run both the mass spec and controller: "create an environment to run both the mass spec and controller"
- [readme] extract the scan results as mzML files from simulated runs: [evidence span withheld — anchor status: external_doc]
- [results] min_intensity threshold 1.75E5 was set high to reduce processing time: "the `min_intensity` threshold (1.75E5) is set very high here so the notebook completes quickly"

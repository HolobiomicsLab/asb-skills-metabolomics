---
name: python-scientific-package-setup
description: Use when a workflow requires running G-Aligner (or reproducing its benchmark
  evaluations) and the Python environment has not yet been provisioned — i.e., before
  executing data/evaluate_metapro_galigner.py or the comparison scripts.
when_to_use_negative:
- The G-Aligner environment is already installed and the goal is only to rerun alignment
  on feature CSVs — skip setup and execute the driver script directly.
- The user wants to align features with MZmine2, OpenMS, or XCMS instead of G-Aligner;
  those tools have their own installation paths and the G-Aligner third_party build
  is unnecessary.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: The scientific package whose environment is being set up; provides the graph-based
    feature alignment pipeline for untargeted LC-MS metabolomics data
  repo: https://github.com/CSi-Studio/G-Aligner
- name: Python
  role: Runtime used for both the third_party setup.py build step and the pip installation
    of requirements.txt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:496353eb7c6ab37b0e284ddec4a585754b693971b233799e306b36fa10093611
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# python-scientific-package-setup

## Summary

Install the G-Aligner execution environment by compiling the bundled third_party C++ extensions and installing the Python dependency list, so that graph-based LC-MS feature alignment scripts from CSi-Studio/G-Aligner can be run on benchmark feature CSVs.

## When to use

Use when a workflow requires running G-Aligner (or reproducing its benchmark evaluations) and the Python environment has not yet been provisioned — i.e., before executing data/evaluate_metapro_galigner.py or the comparison scripts.

## When NOT to use

- The G-Aligner environment is already installed and the goal is only to rerun alignment on feature CSVs — skip setup and execute the driver script directly.
- The user wants to align features with MZmine2, OpenMS, or XCMS instead of G-Aligner; those tools have their own installation paths and the G-Aligner third_party build is unnecessary.

## Inputs

- G-Aligner repository checkout (CSi-Studio/G-Aligner) with third_party/ and requirements.txt
- Local Python environment (version appropriate to the user's system and hardware)
- Zenodo benchmark data archive (doi 10.5281/zenodo.8313034) unpacked into the data/ directory layout (MTBLS562, QE_HF, TripleTOF_6600 with *_results_* folders)

## Outputs

- Installed G-Aligner package with compiled third_party extensions and all pip dependencies resolved
- Environment capable of executing evaluate_metapro_galigner.py, metapro_result_comparison.py and software_result_comparison.py

## How to apply

First prepare a Python environment appropriate for the local system and hardware, since G-Aligner relies on compiled extensions. From the repository root ROOT_PATH, run `cd ROOT_PATH/third_party/` then `python setup.py install` to build the bundled third-party components (including OBI-Warp dependencies used for retention time coarse registration), then `cd ROOT_PATH` and `pip install -r requirements.txt` for the remaining Python dependencies. Place the Zenodo download (doi 10.5281/zenodo.8313034) under `data/` in the layout expected by the demo scripts (MTBLS562, QE_HF, TripleTOF_6600 folders plus their *_results_* subfolders). Only after this setup should `python data/evaluate_metapro_galigner.py` be executed with adjusted parameters. Verify the installation by confirming the benchmark scripts run end-to-end and write alignment results to the `experiment` folder.

## Related tools

- **G-Aligner** (The scientific package whose environment is being set up; provides the graph-based feature alignment pipeline for untargeted LC-MS metabolomics data) — https://github.com/CSi-Studio/G-Aligner
- **Python** (Runtime used for both the third_party setup.py build step and the pip installation of requirements.txt)

## Examples

```
cd ROOT_PATH/third_party/ && python setup.py install; cd ROOT_PATH && pip install -r requirements.txt
```

## Evaluation signals

- `python setup.py install` in third_party/ completes without compilation errors, and `pip install -r requirements.txt` reports no unresolved dependencies.
- The Zenodo data can be placed in the documented layout: data/ contains MTBLS562, QE_HF, TripleTOF_6600 and their metapro/mzmine2/openms/xcms results folders as in the README tree.
- `python data/evaluate_metapro_galigner.py` runs end-to-end and writes alignment results into the `experiment` folder.
- Benchmark reproduction scripts (`python data/metapro_result_comparison.py`, `python data/software_result_comparison.py`) execute without import or path errors.
- Resulting consensus feature groups on the three benchmark datasets contain no ambiguous matched groups, consistent with reported G-Aligner behavior.

## Limitations

- G-Aligner's main limitation is computing speed: the multidimensional assignment problem is NP-Hard and requires substantial computing time even with the two provided acceleration methods.
- GPU acceleration is not yet implemented, so environment/hardware preparation cannot rely on GPU parallelization.
- No changelog is available in the repository, so dependency version drift between the requirements list and upstream packages must be checked manually.

## Evidence

- [readme] Install the dependencies. Here we use ROOT_PATH to represent the root path of G-Aligner.: "Install the dependencies. Here we use ROOT_PATH to represent the root path of G-Aligner."
- [readme] Prepare the python environment based on your system and hardware.: "Prepare the python environment based on your system and hardware."
- [methods] Feature alignment results are saved in ```experiment``` folder.: "Feature alignment results are saved in ```experiment``` folder."
- [methods] The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034): "The mzML files of the metabolomics datasets can be downloaded at [Zenodo](https://doi.org/10.5281/zenodo.8313034)"
- [abstract] G-Aligner is open-source and freely available at https://github.com/CSi-Studio/G-Aligner under a permissive license: "G-Aligner is open-source and freely available at https://github.com/CSi-Studio/G-Aligner under a permissive license"

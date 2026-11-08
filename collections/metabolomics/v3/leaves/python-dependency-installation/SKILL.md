---
name: python-dependency-installation
description: Use when setting up the G-Aligner repository (CSi-Studio/G-Aligner) to
  reproduce LC-MS feature alignment benchmarks and its Python dependencies must be
  installed before running alignment or evaluation scripts.
when_to_use_negative:
- Alignment or evaluation scripts already run without missing-import errors in the
  current environment
- The goal is only to download benchmark data (mzML files and manual annotations from
  Zenodo 10.5281/zenodo.8313034) without executing G-Aligner itself
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: G-Aligner
  role: Target package being installed; its setup.py and requirements.txt define the
    dependency set
  repo: https://github.com/CSi-Studio/G-Aligner
- name: pip
  role: Installs Python dependencies listed in the repository's requirements.txt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-023-05525-4
    title: 'G-Aligner: a graph-based feature alignment method for untargeted LC–MS-based
      metabolomics'
  source_content_hash: sha256:38b95a8d2c57da4d45b3cd5296c23703cc0f5241be177c2378dac41d1125d5af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSi-Studio/G-Aligner
  tools:
  - G-Aligner
  - MZmine2
  - pip
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# python-dependency-installation

## Summary

Installs G-Aligner and its third-party dependencies in a prepared Python environment using the repository's setup.py (for third_party packages) and pip with requirements.txt, so that feature alignment and evaluation scripts can be run.

## When to use

Use when setting up the G-Aligner repository (CSi-Studio/G-Aligner) to reproduce LC-MS feature alignment benchmarks and its Python dependencies must be installed before running alignment or evaluation scripts.

## When NOT to use

- Alignment or evaluation scripts already run without missing-import errors in the current environment
- The goal is only to download benchmark data (mzML files and manual annotations from Zenodo 10.5281/zenodo.8313034) without executing G-Aligner itself

## Inputs

- Cloned G-Aligner repository (CSi-Studio/G-Aligner) at ROOT_PATH
- third_party/ directory containing bundled dependency setup.py
- requirements.txt at repository root
- Prepared Python environment (per host system and hardware)

## Outputs

- Installed G-Aligner package and third_party dependencies in the Python environment
- Runnable data/ evaluation and benchmark scripts (e.g. evaluate_metapro_galigner.py)

## How to apply

First prepare a Python environment appropriate to the host system and hardware. Change directory to ROOT_PATH/third_party/ and run `python setup.py install` to install the third_party dependencies bundled with the repository, then from ROOT_PATH run `pip install -r requirements.txt` for the remaining Python dependencies. Once installed, the repository's data/ scripts (e.g. evaluate_metapro_galigner.py, metapro_result_comparison.py, software_result_comparison.py) can be executed from ROOT_PATH. Correct installation is verified by these scripts running without import errors and writing feature alignment results into the `experiment` folder.

## Related tools

- **G-Aligner** (Target package being installed; its setup.py and requirements.txt define the dependency set) — https://github.com/CSi-Studio/G-Aligner
- **pip** (Installs Python dependencies listed in the repository's requirements.txt)

## Examples

```
cd ROOT_PATH/third_party && python setup.py install && cd ROOT_PATH && pip install -r requirements.txt
```

## Evaluation signals

- `python setup.py install` in ROOT_PATH/third_party/ and `pip install -r requirements.txt` in ROOT_PATH both complete without errors
- Running `python data/metapro_result_comparison.py` or `python data/evaluate_metapro_galigner.py` from ROOT_PATH does not raise ModuleNotFoundError/ImportError
- Feature alignment results are written to the `experiment` folder after running an evaluation script
- All dependencies in requirements.txt resolve in the prepared Python environment

## Limitations

- Environment preparation is system- and hardware-dependent; the README gives no pinned interpreter version or prebuilt binaries
- Third-party dependencies must be installed separately via setup.py in third_party/ before pip requirements, so install order matters

## Evidence

- [readme] Install the dependencies. Here we use ROOT_PATH to represent the root path of G-Aligner.: "Install the dependencies. Here we use ROOT_PATH to represent the root path of G-Aligner."
- [readme] python setup.py install ... pip install -r requirements.txt: "```cd ROOT_PATH\third_party\``` ```python setup.py install``` ```cd ROOT_PATH``` ```pip install -r requirements.txt```"
- [readme] Prepare the python environment based on your system and hardware.: "1. Prepare the python environment based on your system and hardware."

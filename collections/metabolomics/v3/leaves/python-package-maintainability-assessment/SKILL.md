---
name: python-package-maintainability-assessment
description: Use when evaluating a Python package claimed to replace an existing webserver
  or pipeline on the same task, and you must benchmark runtime, memory, scalability
  and maintainability on a shared benchmark (e.g. NIST 555-spectra).
when_to_use_negative:
- Input is already an annotation-propagation result and you only need to interpret
  rankings rather than benchmark the implementation
- The task requires reproducing NAP's local first-neighbor propagation semantics exactly,
  since random-walk propagation reaches nodes not directly connected to spectral matches
- No shared benchmark network exists, so runtime comparisons against the NAP webserver
  baseline would not be comparable
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: ChemWalker
  role: Python package under assessment; its random-walk propagation is timed and
    profiled
  repo: https://github.com/computational-chemical-biology/ChemWalker
- name: Network Annotation Propagation (NAP)
  role: Predecessor tool serving as the baseline for runtime, scalability and annotation-quality
    comparison
- name: pyrwr
  role: Implements the random walk with restart whose computation is timed during
    the assessment
  repo: https://github.com/jinhongjung/pyrwr
- name: Python
  role: Language of the standalone package whose maintainability (conda environment,
    pip installation from git) is evaluated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad078
    title: 'Improving annotation propagation on molecular networks through random
      walks: introducing ChemWalker'
  source_content_hash: sha256:ec0ab8352d087501fe6ec8c4c2077462935df488d77b2fbedc4a5e306d617b68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/computational-chemical-biology/ChemWalker
  tools:
  - ChemWalker
  - Network Annotation Propagation (NAP)
  - Python
  - pyrwr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 18
---

# python-package-maintainability-assessment

## Summary

Assess whether a reimplemented Python tool (here ChemWalker, a random-walk annotation-propagation library over molecular networks) matches the performance of an established predecessor (NAP) while improving running time, scalability and maintainability as a standalone package.

## When to use

Use when evaluating a Python package claimed to replace an existing webserver or pipeline on the same task, and you must benchmark runtime, memory, scalability and maintainability on a shared benchmark (e.g. NIST 555-spectra).

## When NOT to use

- Input is already an annotation-propagation result and you only need to interpret rankings rather than benchmark the implementation
- The task requires reproducing NAP's local first-neighbor propagation semantics exactly, since random-walk propagation reaches nodes not directly connected to spectral matches
- No shared benchmark network exists, so runtime comparisons against the NAP webserver baseline would not be comparable

## Inputs

- Processed NIST 555-spectra molecular network file
- MetFrag candidate-structure fragmentation results (from MetFrag2.3-CL.jar)
- MetFusion scoring weights used to build the weighted graph
- environment.yml specifying the package's conda environment

## Outputs

- comparison_results.csv with runtime, scalability and maintainability columns for ChemWalker vs NAP
- Wall-clock running time and memory usage measurements for random-walk propagation
- Statistical comparison of top-10 candidate classification vs the NAP baseline

## How to apply

Recreate the package's dedicated conda environment from its environment.yml (conda env create -f environment.yml; conda activate chemwalker) and install the package from its GitHub repository with pip install git+...ChemWalker.git. Run the package's core computation — ChemWalker's random-walk propagation (via pyrwr) over the weighted molecular network built from MetFrag candidate-structure fragmentation and MetFusion scoring — on the NIST 555-spectra benchmark, recording wall-clock time, memory usage and number of nodes processed. Compare the top-10 candidate classification against the established tool (NAP) statistically, and assemble measured runtime, scalability and maintainability metrics into a comparison table (comparison_results.csv). Judge success when the measured runtime supports the qualitative claim that the package improves on the predecessor in running time, scalability and maintainability while annotation performance is not statistically worse.

## Related tools

- **ChemWalker** (Python package under assessment; its random-walk propagation is timed and profiled) — https://github.com/computational-chemical-biology/ChemWalker
- **Network Annotation Propagation (NAP)** (Predecessor tool serving as the baseline for runtime, scalability and annotation-quality comparison)
- **pyrwr** (Implements the random walk with restart whose computation is timed during the assessment) — https://github.com/jinhongjung/pyrwr
- **Python** (Language of the standalone package whose maintainability (conda environment, pip installation from git) is evaluated)

## Examples

```
conda env create -f environment.yml; conda activate chemwalker; pip install git+https://github.com/computational-chemical-biology/ChemWalker.git
```

## Evaluation signals

- conda env create -f environment.yml completes and conda activate chemwalker activates a working environment
- pip install git+https://github.com/computational-chemical-biology/ChemWalker.git succeeds in that environment
- Top-10 candidate classification on the NIST 555-spectra benchmark is not statistically differentiable from NAP's
- Measured wall-clock running time and memory usage support the reported improvement over the NAP webserver baseline
- comparison_results.csv is populated with runtime, scalability and maintainability columns for both tools

## Limitations

- ChemWalker's performance claim is qualitative in the article's discussion; a direct NAP runtime re-measurement is not part of the released package, so comparisons rely on the reported webserver baseline
- Random-walk propagation via pyrwr is defined on positively weighted networks only, so the MetFusion-weighted graph must contain positive edge weights
- ChemWalker was tested on the specific MetFrag2.3-CL.jar build; other MetFrag releases are untested

## Evidence

- [abstract] ChemWalker improves on NAP in running time, scalability and maintainability, and is available as a standalone python package: "Compared with NAP, ChemWalker has a series of improvements, on running time, scalability and maintainability and is available as a standalone python package"
- [readme] Environment creation and package installation define maintainability: "conda env create -f environment.yml
conda activate chemwalker 
pip install git+https://github.com/computational-chemical-biology/ChemWalker.git"
- [readme] pyrwr implements the random walk being timed: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random"
- [intro] pyrwr is the random-walk implementation used by ChemWalker: "ChemWalker is a python package to propagate spectral library match identities through candidate structures provided by _in silico_ fragmentation, using [random walk]"
- [methods] MetFrag provides the candidate-structure annotations feeding the benchmark: "ChemWalker uses MetFrag for in silico annotation"

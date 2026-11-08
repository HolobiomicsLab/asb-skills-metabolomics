---
name: conda-environment-dependency-setup
description: Use when a published tool's README specifies a conda environment with
  pinned Python version and channel configuration (defaults, bioconda, conda-forge,
  plus extra channels like etetoolkit) that must be created before running the tool's
  pipeline.
when_to_use_negative:
- The tool is already installable from Bioconda as a package (the TAMPA README notes
  a Bioconda pull request was still pending, so the manual conda-create route was
  required); do not hand-build environments when a maintained package exists
- The target tool only provides pip/setuptools requirements files rather than a conda
  channel recipe — a plain virtualenv workflow is simpler
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TAMPA
  role: The tool whose documented CAMIViz conda environment (python=3.7 with numpy,
    ete3, seaborn, pandas, matplotlib, biom-format) is being set up; TAMPA cannot
    run until this environment is activated
  repo: https://github.com/dkoslicki/TAMPA.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:8e1623263faf76982d84ad5cd793ec2de3f2af23c7827032fc97703e7da92d07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - TAMPA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# conda-environment-dependency-setup

## Summary

Reproducibly provision the Python environment required to run a published bioinformatics tool, here the CAMIViz conda environment (python=3.7, numpy, ete3, seaborn, pandas, matplotlib, biom-format) needed to execute TAMPA's ete3-based tree plotting.

## When to use

Use when a published tool's README specifies a conda environment with pinned Python version and channel configuration (defaults, bioconda, conda-forge, plus extra channels like etetoolkit) that must be created before running the tool's pipeline.

## When NOT to use

- The tool is already installable from Bioconda as a package (the TAMPA README notes a Bioconda pull request was still pending, so the manual conda-create route was required); do not hand-build environments when a maintained package exists
- The target tool only provides pip/setuptools requirements files rather than a conda channel recipe — a plain virtualenv workflow is simpler

## Inputs

- Git repository URL of the published tool (github.com/dkoslicki/TAMPA)
- README-specified dependency list (python=3.7, numpy, ete3, seaborn, pandas, matplotlib, biom-format)
- Conda channel configuration (defaults, bioconda, conda-forge, etetoolkit)

## Outputs

- Activated conda environment named CAMIViz with python 3.7 and all TAMPA dependencies installed
- A working installation from which 'python src/tampa.py' runs without missing-module errors

## How to apply

Clone the tool's repository (e.g. github.com/dkoslicki/TAMPA.git) and cd into it so the environment is created alongside the code and bundled data. Configure conda channels in the documented order — defaults, bioconda, conda-forge — because channel priority affects which builds of bioinformatics dependencies are resolved. Create the environment with a single 'conda create' command that lists the extra channel (-c etetoolkit for ete3), the pinned Python version (3.7), and the exact dependency set (numpy, ete3, seaborn, pandas, matplotlib, biom-format), naming it as the README specifies (CAMIViz). Activate the environment before every run so the ete3 tree-building pipeline and plotting backends resolve to the pinned versions; verify by running the tool's example command and confirming it produces the expected plot without import errors.

## Related tools

- **TAMPA** (The tool whose documented CAMIViz conda environment (python=3.7 with numpy, ete3, seaborn, pandas, matplotlib, biom-format) is being set up; TAMPA cannot run until this environment is activated) — https://github.com/dkoslicki/TAMPA.git

## Examples

```
conda create -c etetoolkit -y -n CAMIViz python=3.7 numpy ete3 seaborn pandas matplotlib biom-format && conda activate CAMIViz
```

## Evaluation signals

- conda env list shows an environment named CAMIViz created with python 3.7 and the README's dependency list
- conda activate CAMIViz succeeds and 'python -c "import ete3, seaborn, pandas, matplotlib, biom"' completes without ImportError
- Running the README example command 'python src/tampa.py -i data/mad_yalow_0.profile.txt ...' executes to completion rather than failing on a missing module
- The conda channel configuration reflects the documented order: defaults, bioconda, conda-forge added via 'conda config --add channels', with etetoolkit passed via '-c' at creation time

## Limitations

- The Bioconda install path was not yet available at the time of writing ('Waiting for pull request to get merged'), so only the manual conda-create recipe was supported
- Python is pinned to 3.7; upgrading the interpreter may break the ete3 dependency, which requires the extra etetoolkit channel
- The environment must be created inside the cloned repository directory per the README workflow, which ties it to that checkout

## Evidence

- [readme] conda create -c etetoolkit -y -n CAMIViz python=3.7 numpy  ete3  seaborn pandas matplotlib biom-format: "conda create -c etetoolkit -y -n CAMIViz python=3.7 numpy  ete3  seaborn pandas matplotlib biom-format"
- [readme] conda config --add channels defaults; conda config --add channels bioconda; conda config --add channels conda-forge: "conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge"
- [methods] git clone https://github.com/dkoslicki/TAMPA.git: "git clone https://github.com/dkoslicki/TAMPA.git"
- [readme] The following commands create a conda environment inside the repository with the dependencies.: "The following commands create a conda environment inside the repository with the dependencies."

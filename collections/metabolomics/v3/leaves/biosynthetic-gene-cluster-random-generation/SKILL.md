---
name: biosynthetic-gene-cluster-random-generation
description: Use when you need synthetic PKS/NRPS/hybrid BGCs of controlled module
  count (e.g. 1-10 modules, 10 BGCs per type) to benchmark rendering speed, validate
  drawing correctness/readability, or stress-test biosynthetic pathway visualization
  pipelines.
when_to_use_negative:
- When you need real, experimentally characterized BGCs — use the MIBiG database instead,
  as randomly generated clusters are synthetic and unvalidated against known products
- When you need mature natural product structures from raw antiSMASH output — RAIChU
  can generate scaffolds but not mature natural products from raw antiSMASH output
- When regio- and stereochemistry of tailoring enzymes must be predicted automatically
  — users must specify their reaction targets
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Provides the cluster generation functionality to randomly produce PKS/NRPS/hybrid
    BGCs and the spaghetti-diagram drawing call that is timed
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: Scripting environment (Python 3.9.12) used to drive generation, time rendering
    with time.perf_counter(), and aggregate timings with pandas
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:0248e4c5eac835ab14038001a10f225e4fbe79b83c1eaeb937646b82149abea2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - Python
  - RAIChU
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# biosynthetic-gene-cluster-random-generation

## Summary

Generates random PKS, NRPS, and PKS/NRPS hybrid biosynthetic gene clusters (1-10 modules) using RAIChU's cluster generation functionality, enabling benchmarking tasks such as the paper's drawing-speed assessment where 10 BGCs per module number (1-10) and module type are produced and timed.

## When to use

Use when you need synthetic PKS/NRPS/hybrid BGCs of controlled module count (e.g. 1-10 modules, 10 BGCs per type) to benchmark rendering speed, validate drawing correctness/readability, or stress-test biosynthetic pathway visualization pipelines.

## When NOT to use

- When you need real, experimentally characterized BGCs — use the MIBiG database instead, as randomly generated clusters are synthetic and unvalidated against known products
- When you need mature natural product structures from raw antiSMASH output — RAIChU can generate scaffolds but not mature natural products from raw antiSMASH output
- When regio- and stereochemistry of tailoring enzymes must be predicted automatically — users must specify their reaction targets

## Inputs

- Module type specification (NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS)
- Module count range (1-10 modules per BGC)
- Number of BGCs to generate per combination (e.g. 10 for speed assessment, 1000 per type for validation)
- Python 3.9.12 conda environment with raichu installed via pip

## Outputs

- Randomly generated BGC definitions consumable by RAIChU's drawing call
- Spaghetti diagram renderings (.svg files; .gif animations of all generated clusters per validation set archived on Zenodo, doi 10.5281/zenodo.10987931)
- Timing table (e.g. drawing_time_results.csv) with mean and standard deviation of drawing seconds per module type and module number

## How to apply

Set up a conda environment with Python 3.9.12 and install raichu via pip (RAIChU requires this Python version on the main branch; a biopython_independent branch exists for newer versions). Use RAIChU's cluster generation functionality to randomly generate BGCs for each of the 5 module types (NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS) and each module count from 1 to 10, generating 10 BGCs per combination (500 clusters for a speed study; the paper used 1000 per type for validation). Wrap each spaghetti-diagram rendering call in Python's time.perf_counter() to measure drawing time per cluster, then aggregate timings with pandas (mean and standard deviation per module type and per module number) and export to CSV. Expect PKS clusters to draw fastest (paper average 2.73 s per PKS cluster, vs 5.21 s for hybrids and 9.62 s for NRPS). Validate that all timing rows are present, finite, and non-negative, and that the qualitative speed ordering matches the published result.

## Related tools

- **RAIChU** (Provides the cluster generation functionality to randomly produce PKS/NRPS/hybrid BGCs and the spaghetti-diagram drawing call that is timed) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Scripting environment (Python 3.9.12) used to drive generation, time rendering with time.perf_counter(), and aggregate timings with pandas) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- All 500 (5 types x 10 counts x 10 BGCs) timing rows are present with non-negative, finite durations
- Qualitative speed ordering matches the paper: PKS fastest (mean 2.73 s per cluster), hybrid NRPS/PKS intermediate (5.21 s), NRPS slowest (9.62 s)
- Aggregated statistics (mean and standard deviation per module type and per module number) are computed and written to the CSV without missing groups
- Rendering succeeds without unexpected errors for every generated cluster, mirroring the paper's 100% drawing correctness on randomly generated sets

## Limitations

- Timing results are machine- and environment-dependent; only the qualitative speed ordering (PKS < hybrid < NRPS) is expected to reproduce, not the absolute per-cluster seconds
- Random generation requires Python 3.9.12 on the main branch; newer Python versions require the biopython_independent branch
- Cationic cascade reactions in terpenes necessitate programmatic shortcuts, potentially leading to simplified representations of reaction intermediates

## Evidence

- [results] randomly generating spaghetti diagrams for 1–10 module BGCs, generating 10 BGCs per module number (1–10) and module type: "randomly generating spaghetti diagrams for 1–10 module BGCs, generating 10 BGCs per module number (1–10) and module type"
- [results] RAIChU performed fastest for PKS clusters, with an average drawing time of 2.73 s per cluster: "RAIChU performed fastest for PKS clusters, with an average drawing time of 2.73 s per cluster"
- [readme] create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [results] we randomly generated 1000 NRPS BGCs, 1000 cis-AT PKS BGCs, 1000 trans-AT PKS BGCs and 1000 hybrid cis-AT PKS/trans-AT PKS BGCs and 1000 hybrid NRPS/PKS BGCs: [evidence span withheld — anchor status: not_found]
- [discussion] RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output: "As RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, predicted and real-world structures may not have a high fingerprinting similarity"

---
name: pks-nrps-biosynthetic-cluster-generation
description: Use when you need validation sets of modular NRPS, cis-AT PKS, trans-AT
  PKS, or hybrid PKS/NRPS BGC architectures of known module/domain composition to
  test a biosynthetic drawing or prediction tool.
when_to_use_negative:
- You need visualisation of real, sequence-derived BGCs — use RAIChU on antiSMASH
  predictions of MIBiG or user BGCs instead of random generation
- Your target compound class is not modular PKS/NRPS (e.g. RiPPs or terpenes), where
  RAIChU's random PKS/NRPS cluster generator does not apply
- You require predicted regio- and stereochemistry of tailoring enzymes — RAIChU cannot
  predict these and users must specify reaction targets
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RAIChU
  role: Randomly generates PKS/NRPS cluster architectures and renders them as spaghetti
    diagrams for correctness/readability validation
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: PIKAChU
  role: Enforces chemical correctness of the elongation and tailoring reaction intermediates
    in the rendered diagrams
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: Python
  role: Runtime environment (Python 3.9.12 via conda) in which RAIChU is installed
    and run as a pip package
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:2e3566b85e30f0258190f3d4a295ea7b74f86d090f169190587c636dd8b3f6c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - PIKAChU
  - Python
  - RAIChU
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pks-nrps-biosynthetic-cluster-generation

## Summary

Generates randomised in silico PKS/NRPS biosynthetic gene cluster architectures with RAIChU to benchmark structure-drawing correctness and readability, producing 'spaghetti diagrams' whose chemical intermediates are enforced by PIKAChU.

## When to use

Use when you need validation sets of modular NRPS, cis-AT PKS, trans-AT PKS, or hybrid PKS/NRPS BGC architectures of known module/domain composition to test a biosynthetic drawing or prediction tool.

## When NOT to use

- You need visualisation of real, sequence-derived BGCs — use RAIChU on antiSMASH predictions of MIBiG or user BGCs instead of random generation
- Your target compound class is not modular PKS/NRPS (e.g. RiPPs or terpenes), where RAIChU's random PKS/NRPS cluster generator does not apply
- You require predicted regio- and stereochemistry of tailoring enzymes — RAIChU cannot predict these and users must specify reaction targets

## Inputs

- Random generation parameters: BGC category (NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS) and elongation module counts sampled uniformly from 0–11
- Number of clusters to generate per category (1000 in the published validation, 5000 total)

## Outputs

- 5000 randomly generated modular BGC cluster definitions spanning five PKS/NRPS categories
- Rendered spaghetti diagrams (.svg files and .gif animations of all generated clusters per validation set, archived at 10.5281/zenodo.10987931)
- Per-cluster-type correctness and readability percentages (e.g. validation_metrics.csv)

## How to apply

Install RAIChU in a conda environment with Python 3.9.12 ('pip install raichu') and use its random cluster generation functionality to sample 1000 BGCs per category — NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, and hybrid NRPS/PKS — drawing elongation module counts uniformly across the 0–11 module range. Render each generated cluster as a spaghetti diagram with RAIChU, using PIKAChU to enforce chemical correctness of the elongation and tailoring reaction intermediates. Manually assess each diagram for drawing correctness (depicted structures match the encoded module/domain composition) and drawing readability (legible bond angles, no text/structure overlap), tallying pass/fail per cluster type. The published benchmark this protocol reproduces is 100% correctness and ~97.6% readability (97.62% on the random sets; 97.66% abstracted across 5000 PKS/NRPS systems).

## Related tools

- **RAIChU** (Randomly generates PKS/NRPS cluster architectures and renders them as spaghetti diagrams for correctness/readability validation) — https://github.com/BTheDragonMaster/RAIChU
- **PIKAChU** (Enforces chemical correctness of the elongation and tailoring reaction intermediates in the rendered diagrams) — https://github.com/BTheDragonMaster/RAIChU
- **Python** (Runtime environment (Python 3.9.12 via conda) in which RAIChU is installed and run as a pip package) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- Correctness: every rendered diagram's depicted structures must match the encoded module/domain composition of the generated cluster (100% expected on random sets)
- Readability: each diagram must show legible bond angles with no text/structure overlap (~97.6% expected, e.g. 97.62% on the randomly generated sets)
- Coverage check: the validation set must contain 1000 clusters for each of the five BGC categories, totalling 5000
- Runtime sanity: average drawing time on the order of seconds per cluster (2.73 s per PKS cluster in the paper) — large deviations indicate misconfiguration
- PIKAChU check: elongation and tailoring reaction intermediates must be chemically valid (PIKAChU enforces chemical correctness)

## Limitations

- Regio- and stereochemistry of tailoring enzymes cannot currently be predicted; users must specify their reaction targets
- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy (as observed in the CMC-thuggacins)
- Some reactions, such as cationic cascade reactions in terpenes, require programmatic shortcuts that may lead to simplified representations of reaction intermediates; a summarized pathway view is offered instead

## Evidence

- [results] Random generation protocol: "we randomly generated 1000 NRPS BGCs, 1000 cis-AT PKS BGCs, 1000 trans-AT PKS BGCs"
- [results] Validation benchmark: "We achieved 97.62% readability and 100% accuracy on all randomly generated sets of clusters"
- [results] PIKAChU role: "we implemented the most commonly occurring elongation and tailoring reactions using PIKAChU, a tool which enforces chemical correctness"
- [results] Manual assessment criteria: "We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures"
- [readme] Installation: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"

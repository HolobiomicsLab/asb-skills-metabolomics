---
name: spaghetti-diagram-correctness-assessment
description: Use when validating that automated biosynthetic pathway visualisations
  correctly depict the encoded module/domain composition and remain legible, e.g.
  benchmarking a drawing tool on randomly generated NRPS, cis-AT PKS, trans-AT PKS,
  or hybrid PKS/NRPS BGCs.
when_to_use_negative:
- The goal is predicting natural product structures from raw antiSMASH output rather
  than assessing diagram quality — RAIChU generates scaffolds but not mature natural
  products, and regio/stereochemistry of tailoring enzymes must be user-specified
- Validation of structures against characterised MIBiG products is needed — the article
  relied on manual curation of only a few select examples and fingerprinting-based
  automatic validation was considered but not implemented
- Speed benchmarking rather than correctness/readability assessment — use the separate
  protocol generating 10 BGCs per module number (1-10) and module type to measure
  drawing time (2.73 s per PKS cluster)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Renders spaghetti diagrams of randomly generated PKS/NRPS clusters and provides
    random cluster generation functionality for the validation sets
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: PIKAChU
  role: Enforces chemical correctness of the elongation and tailoring reaction intermediates
    depicted in rendered diagrams
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: antiSMASH
  role: Source of real-world BGC predictions (MIBiG) optionally run through RAIChU
    for additional validation
- name: Python
  role: Runtime (Python 3.9.12 conda environment) and pip-installable scripting integration
    for the validation workflow
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:b3cc8af3ce9730acdf85803189b788193cc56606ada48a64b215e23b03d4de29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - PIKAChU
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spaghetti-diagram-correctness-assessment

## Summary

A validation protocol for RAIChU-generated 'spaghetti diagrams' of PKS/NRPS biosynthetic pathways, in which randomly generated modular cluster architectures are rendered and manually scored for cluster drawing correctness (100% target) and drawing readability (97.66% target) across five BGC categories.

## When to use

Use when validating that automated biosynthetic pathway visualisations correctly depict the encoded module/domain composition and remain legible, e.g. benchmarking a drawing tool on randomly generated NRPS, cis-AT PKS, trans-AT PKS, or hybrid PKS/NRPS BGCs.

## When NOT to use

- The goal is predicting natural product structures from raw antiSMASH output rather than assessing diagram quality — RAIChU generates scaffolds but not mature natural products, and regio/stereochemistry of tailoring enzymes must be user-specified
- Validation of structures against characterised MIBiG products is needed — the article relied on manual curation of only a few select examples and fingerprinting-based automatic validation was considered but not implemented
- Speed benchmarking rather than correctness/readability assessment — use the separate protocol generating 10 BGCs per module number (1-10) and module type to measure drawing time (2.73 s per PKS cluster)

## Inputs

- Randomly generated modular BGC architectures (1000 each of NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS) spanning 0-11 elongation modules
- RAIChU-rendered spaghetti diagram .svg files for each generated cluster
- antiSMASH predictions of real-world BGCs from the MIBiG database (optional real-world validation set)

## Outputs

- Per-cluster-type correctness and readability percentages (e.g. validation_metrics.csv with pass/fail tallies)
- Rendered spaghetti diagrams as separate .svg files and .gif animations per validation set (archived at 10.5281/zenodo.10987931)

## How to apply

Set up a conda environment with Python 3.9.12 and install RAIChU via pip ('pip install raichu'). Use RAIChU's random cluster generation functionality to sample 1000 BGCs for each of five categories (NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS), drawing elongation module counts uniformly across the 0-11 module range per the Methods validation protocol. Render spaghetti diagrams for all 5000 clusters with RAIChU, using PIKAChU to enforce chemical correctness of the elongation and tailoring reaction intermediates. Manually assess each diagram on the correctness of the depicted structures (matching the encoded module/domain composition) and on drawing readability (legible bond angles, no text/structure overlap), tallying pass/fail per cluster type, then compute per-type percentages and compare against the reported benchmarks of 100% correctness and 97.66% readability (97.62% on the random sets).

## Related tools

- **RAIChU** (Renders spaghetti diagrams of randomly generated PKS/NRPS clusters and provides random cluster generation functionality for the validation sets) — https://github.com/BTheDragonMaster/RAIChU
- **PIKAChU** (Enforces chemical correctness of the elongation and tailoring reaction intermediates depicted in rendered diagrams) — https://github.com/BTheDragonMaster/RAIChU
- **antiSMASH** (Source of real-world BGC predictions (MIBiG) optionally run through RAIChU for additional validation)
- **Python** (Runtime (Python 3.9.12 conda environment) and pip-installable scripting integration for the validation workflow) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- Cluster drawing correctness reaches 100%: every rendered structure matches the encoded module/domain composition in each of the five BGC categories
- Drawing readability reaches ~97.6% (97.62% on random sets; abstract rounds to 97.66% across 5000 systems): legible bond angles and no text/structure overlap
- Validation sets are balanced: exactly 1000 clusters per BGC type, module counts drawn uniformly across the 0-11 range
- PIKAChU reports no chemical-correctness violations for elongation or tailoring reaction intermediates
- Per-type percentages are tabulated (e.g. written to validation_metrics.csv) and match or exceed the reported benchmarks

## Limitations

- Regio- and stereochemistry of most tailoring enzymes cannot be predicted; users must specify reaction targets
- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy (e.g. the CMC-thuggacins)
- Cationic cascade reactions in terpenes necessitate programmatic shortcuts, so RAIChU offers a summarized pathway view showing only initial and final intermediates; assessment should account for such simplified representations
- The benchmark of 97.66% readability is rounded from 97.62% observed on the randomly generated sets; results depend on manual assessment, which is subjective for bond-angle legibility

## Evidence

- [abstract] RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems"
- [results] We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures: "We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures"
- [results] we randomly generated 1000 NRPS BGCs, 1000 cis-AT PKS BGCs, 1000 trans-AT PKS BGCs, 1000 hybrid cis-AT PKS/trans-AT PKS BGCs and 1000 hybrid NRPS/PKS BGCs: "we randomly generated 1000 NRPS BGCs, 1000 cis-AT PKS BGCs, 1000 trans-AT PKS BGCs, 1000 hybrid cis-AT PKS/trans-AT PKS BGCs and 1000 hybrid NRPS/PKS BGCs"
- [results] we implemented the most commonly occurring elongation and tailoring reactions using PIKAChU, a tool which enforces chemical correctness of reaction intermediates: "we implemented the most commonly occurring elongation and tailoring reactions using PIKAChU, a tool which enforces chemical correctness of reaction intermediates"
- [readme] First, create a new environment with python version 3.9.12, for instance with conda: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"

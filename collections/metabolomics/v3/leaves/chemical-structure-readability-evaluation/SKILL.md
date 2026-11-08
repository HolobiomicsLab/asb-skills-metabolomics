---
name: chemical-structure-readability-evaluation
description: Use when a tool renders many predicted biosynthetic cluster diagrams
  (PKS, NRPS, or hybrid 'spaghetti diagrams') and you must quantify how many are structurally
  correct and visually legible.
when_to_use_negative:
- Validating chemical product structures against known compounds (e.g. MIBiG entries)
  — the article notes fingerprinting-based automatic validation of MIBiG BGCs was
  considered but not implemented, and predicted scaffolds may not match mature natural
  products from raw antiSMASH output
- Assessing prediction accuracy of BGC detection/annotation itself (use antiSMASH/PRISM
  evaluation rather than diagram readability scoring)
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: RAIChU
  role: Generates the spaghetti diagrams of PKS/NRPS clusters that are scored for
    correctness and readability; installed via pip in a Python 3.9.12 conda environment
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: PIKAChU
  role: Enforces chemical correctness of elongation and tailoring reaction intermediates
    in the rendered diagrams prior to readability assessment
- name: antiSMASH
  role: Source of BGC detection/annotation results that users can upload and edit,
    or use to obtain real-world MIBiG BGC predictions for rendering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:7fc52c8509ee9581fac1de83427a12d0fc4c2509e645759fe81642e2f0945967
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - PIKAChU
  - Python
  - RAIChU
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chemical-structure-readability-evaluation

## Summary

A validation protocol for automatically generated biosynthetic 'spaghetti diagrams' in which each rendered cluster is manually scored for drawing correctness (depicted structures match encoded module/domain composition) and readability (legible bond angles, no text/structure overlap), benchmarked against RAIChU's reported 100% correctness and 97.66% readability.

## When to use

Use when a tool renders many predicted biosynthetic cluster diagrams (PKS, NRPS, or hybrid 'spaghetti diagrams') and you must quantify how many are structurally correct and visually legible.

## When NOT to use

- Validating chemical product structures against known compounds (e.g. MIBiG entries) — the article notes fingerprinting-based automatic validation of MIBiG BGCs was considered but not implemented, and predicted scaffolds may not match mature natural products from raw antiSMASH output
- Assessing prediction accuracy of BGC detection/annotation itself (use antiSMASH/PRISM evaluation rather than diagram readability scoring)

## Inputs

- Rendered cluster diagrams (spaghetti diagrams, e.g. .svg files) for modular PKS/NRPS BGCs
- Encoded module/domain composition of each cluster (from RAIChU cluster definitions or antiSMASH predictions)
- Randomly generated BGC sets (e.g. 1000 per type: NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS)

## Outputs

- Per-cluster pass/fail tallies for drawing correctness and drawing readability
- Per-BGC-type correctness and readability percentages (e.g. written to validation_metrics.csv)
- Aggregate benchmark comparison (target: 100% correctness, ~97.66% readability)

## How to apply

Generate the diagrams with RAIChU (installed via 'pip install raichu' in a Python 3.9.12 conda environment), using PIKAChU to enforce chemical correctness of elongation and tailoring reaction intermediates. Randomly sample clusters across the categories being validated — the reference protocol generated 1000 BGCs each for NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, and hybrid NRPS/PKS types (5000 total), drawing module counts uniformly over 0–11 elongation modules. Manually assess each diagram on three characteristics: correctness of the depicted structures against the encoded module/domain composition, legibility of bond angles, and absence of text/structure overlap, tallying pass/fail per cluster type. Readability = fraction of diagrams with both acceptable bond angles and no overlap; correctness = fraction where structures match the input cluster architecture. Compare computed percentages against the published benchmarks of 100% correctness and 97.62–97.66% readability, and optionally record average drawing time (2.73 s per PKS cluster in the reference run).

## Related tools

- **RAIChU** (Generates the spaghetti diagrams of PKS/NRPS clusters that are scored for correctness and readability; installed via pip in a Python 3.9.12 conda environment) — https://github.com/BTheDragonMaster/RAIChU
- **PIKAChU** (Enforces chemical correctness of elongation and tailoring reaction intermediates in the rendered diagrams prior to readability assessment)
- **antiSMASH** (Source of BGC detection/annotation results that users can upload and edit, or use to obtain real-world MIBiG BGC predictions for rendering)

## Examples

```
conda create -n raichu python==3.9.12 && conda activate raichu && pip install raichu
```

## Evaluation signals

- Computed drawing correctness across all randomly generated sets equals 100% (depicted structures match encoded module/domain composition for every diagram)
- Computed drawing readability falls at or near the reported benchmark (97.62% on the random sets; abstract rounds this to 97.66% across 5000 PKS/NRPS systems)
- Validation corpus matches the protocol: 1000 BGCs per each of the five categories (NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS), module counts spanning the 0–11 range
- Every rendered diagram was individually assessed on all three characteristics (structure correctness, bond angles, overlap) with pass/fail tallies per cluster type
- Rendering completes without errors and with plausible runtime (reference: average 2.73 s per PKS cluster)

## Limitations

- Readability assessment is manual, so scores are subjective and not reproducible without a written rubric; only a few select MIBiG examples (10 NRPS, 10 PKS, 10 NRPS/PKS hybrids) were manually curated rather than automated validation of all MIBiG BGCs
- RAIChU can generate natural product scaffolds but not mature natural products from raw antiSMASH output, so predicted and real-world structures may show low fingerprinting similarity if used for automated structure-level validation
- Programmatic shortcuts for reactions such as cationic cascade reactions in terpenes may yield simplified representations of reaction intermediates, affecting what a readability/correctness check observes

## Evidence

- [abstract] RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"
- [results] Random validation corpus and per-type breakdown: "we randomly generated 1000 NRPS BGCs, 1000 cis-AT PKS BGCs, 1000 trans-AT PKS BGCs, 1000 hybrid cis-AT PKS/trans-AT PKS BGCs and 1000 hybrid NRPS/PKS BGCs"
- [results] Manual assessment criteria for each spaghetti diagram: "We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures"
- [results] Reported readability/accuracy benchmark on random clusters: "We achieved 97.62% readability and 100% accuracy on all randomly generated sets of clusters"
- [readme] Installation procedure for reproducing the rendering environment: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"

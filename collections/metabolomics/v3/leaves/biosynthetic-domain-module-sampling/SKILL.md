---
name: biosynthetic-domain-module-sampling
description: Use when you need an unbiased benchmark set of modular PKS/NRPS cluster
  architectures (NRPS, cis-AT PKS, trans-AT PKS, hybrid PKS types) to stress-test
  automated drawing, prediction, or annotation pipelines on combinatorial module compositions.
when_to_use_negative:
- When validating against experimentally verified, real-world BGCs — instead run RAIChU
  on antiSMASH predictions of MIBiG database clusters
- When the pipeline requires mature natural product structures from raw antiSMASH
  output — RAIChU generates scaffolds, not fully matured products, and cannot predict
  regio/stereochemistry of tailoring enzymes
- When sampling non-modular BGC classes (e.g. RiPPs, terpenes) whose biosynthetic
  logic is not represented by PKS/NRPS elongation modules
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: RAIChU
  role: Randomly generates PKS/NRPS cluster architectures and renders them as spaghetti
    diagrams for validation
  repo: https://github.com/BTheDragonMaster/RAIChU
- name: PIKAChU
  role: Enforces chemical correctness of the elongation and tailoring reaction intermediates
    during rendering
- name: Python
  role: Environment (Python 3.9.12 via conda, raichu installed via pip) in which the
    random generation and rendering are scripted
  repo: https://pypi.org/project/raichu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00898-x
    title: 'RAIChU: automating the visualisation of natural product biosynthesis'
  source_content_hash: sha256:6dc434b7b07aaa77d189cffd03aa38930a974f492c4d18534248ba4979393001
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BTheDragonMaster/RAIChU
  tools:
  - PIKAChU
  - Python
  - RAIChU
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# biosynthetic-domain-module-sampling

## Summary

Randomly sample modular PKS/NRPS biosynthetic gene cluster architectures (1000 per category across NRPS, cis-AT PKS, trans-AT PKS, hybrid cis/trans-AT PKS, and hybrid NRPS/PKS) with elongation module counts drawn uniformly over the 0–11 module range, to build a benchmark set for validating automated biosynthetic pathway visualisation with RAIChU.

## When to use

Use when you need an unbiased benchmark set of modular PKS/NRPS cluster architectures (NRPS, cis-AT PKS, trans-AT PKS, hybrid PKS types) to stress-test automated drawing, prediction, or annotation pipelines on combinatorial module compositions.

## When NOT to use

- When validating against experimentally verified, real-world BGCs — instead run RAIChU on antiSMASH predictions of MIBiG database clusters
- When the pipeline requires mature natural product structures from raw antiSMASH output — RAIChU generates scaffolds, not fully matured products, and cannot predict regio/stereochemistry of tailoring enzymes
- When sampling non-modular BGC classes (e.g. RiPPs, terpenes) whose biosynthetic logic is not represented by PKS/NRPS elongation modules

## Inputs

- Randomly generated PKS/NRPS BGC specifications produced by RAIChU's random cluster generation functionality (module counts 0–11, drawn uniformly)
- Five BGC categories: NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, hybrid NRPS/PKS (1000 clusters each, 5000 total)

## Outputs

- 5000 rendered RAIChU 'spaghetti diagram' SVG visualisations of randomly generated PKS/NRPS clusters (archived as .svg and .gif files, e.g. Zenodo 10.5281/zenodo.10987931)
- Per-cluster-type correctness and readability tally (e.g. validation_metrics.csv with percentages: 100% correctness, ~97.62–97.66% readability)

## How to apply

Use RAIChU's random cluster generation functionality to sample 1000 BGCs for each of the five categories: NRPS, cis-AT PKS, trans-AT PKS, hybrid cis-AT/trans-AT PKS, and hybrid NRPS/PKS, drawing elongation module counts uniformly across the 0–11 module range per the Methods validation protocol. Render each generated cluster as a 'spaghetti diagram' with RAIChU, using PIKAChU to enforce chemical correctness of the elongation and tailoring reaction intermediates. Manually assess each rendered diagram for drawing correctness (depicted structures match the encoded module/domain composition) and drawing readability (legible bond angles, no text/structure overlap), tallying pass/fail per cluster type. Compute per-type correctness and readability percentages and compare against the reported benchmarks of 100% correctness and 97.66% readability (97.62% on random sets). Average drawing time (~2.73 s per PKS cluster) provides a secondary performance check.

## Related tools

- **RAIChU** (Randomly generates PKS/NRPS cluster architectures and renders them as spaghetti diagrams for validation) — https://github.com/BTheDragonMaster/RAIChU
- **PIKAChU** (Enforces chemical correctness of the elongation and tailoring reaction intermediates during rendering)
- **Python** (Environment (Python 3.9.12 via conda, raichu installed via pip) in which the random generation and rendering are scripted) — https://pypi.org/project/raichu

## Examples

```
conda create -n raichu python==3.9.12; conda activate raichu; pip install raichu
```

## Evaluation signals

- Cluster drawing correctness of 100%: every rendered structure matches the encoded module/domain composition of the randomly generated cluster
- Drawing readability of ~97.62% on random sets (97.66% reported overall): legible bond angles and no text/structure overlap
- Sample-size invariant: exactly 1000 clusters per category (5 categories, 5000 total), with module counts spanning the 0–11 range uniformly
- Rendering completes without errors for all 5000 clusters (cf. MIBiG whole-database error check); average drawing time ~2.73 s per PKS cluster

## Limitations

- Double bond stereochemistry following dehydration reactions in PKS systems is a source of discrepancy (e.g. CMC-thuggacins)
- Readability is not 100% — a small fraction (~2.4%) of randomly generated diagrams fail manual readability assessment
- Assessment is manual (correctness, bond angles, overlap), which does not scale automatically; fingerprinting-based automatic validation was considered but not implemented

## Evidence

- [abstract] RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database: "RAIChU's cluster drawing correctness (100%) and drawing readability (97.66%) were validated on 5000 randomly generated PKS/NRPS systems, and on the MIBiG database"
- [results] 1000 clusters per BGC category: "we randomly generated 1000 NRPS BGCs, 1000 cis-AT PKS BGCs, 1000 trans-AT PKS BGCs, 1000 hybrid cis-AT PKS/trans-AT PKS BGCs and 1000 hybrid NRPS/PKS BGCs"
- [results] Manual readability/correctness assessment criteria: "We manually assessed each spaghetti diagram on three characteristics: the correctness of the depicted structures"
- [methods] Install raichu via pip in a Python 3.9.12 environment: "First, create a new environment with python version 3.9.12, for instance with conda: conda create -n raichu python==3.9.12"
- [results] PIKAChU enforces chemical correctness of intermediates: "we implemented the most commonly occurring elongation and tailoring reactions using PIKAChU, a tool which enforces chemical correctness of reaction intermediates"

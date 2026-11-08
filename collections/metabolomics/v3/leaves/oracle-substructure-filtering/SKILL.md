---
name: oracle-substructure-filtering
description: Use when benchmarking ModiFinder on MS/MS compound pairs whose true modification
  site is known, and shifted-peak substructure assignments must be restricted to those
  containing that site before scoring.
when_to_use_negative:
- The modification site of the unknown compound is genuinely unknown — oracle filtering
  requires ground truth and would leak the answer
- The goal is de novo fragment annotation rather than restricting shifted-peak assignments
  to a known modification site
- Pairs fail benchmark eligibility checks (e.g., precursor mass >= 2000 Da or precursor
  mass difference >= 50%), since the oracle sweep is defined on the eligible 12,909-pair
  benchmark
edam_operation: http://edamontology.org/operation_0321
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ModiFinder
  role: Provides the oracle mode that filters substructures lacking the true modification
    site from shifted-peak assignments, plus generate_probabilities() and BasicEvaluationEngine.evaluate_single()
    for scoring
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: MAGMaAnnotationEngine performs the breadth-first combinatorial fragmentation
    (depth 2-4) whose substructure assignments the oracle step filters
- name: MS/MS library search
  role: Source of the GNPS spectral-library compound pairs (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE,
    GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) on which the oracle
    filter is evaluated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:1a29075615efe719d0af4c620cdee78eb4fcfc1df1f9fa31fe0d40996de9609c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - MS/MS library search
  - ModiFinder
  edam_operation: http://edamontology.org/operation_0321
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# oracle-substructure-filtering

## Summary

An evaluation-mode filtering step in ModiFinder that removes any substructure assigned to a shifted MS/MS peak when that substructure does not contain the true modification site, enabling upper-bound (oracle) assessment of modification-site localization on benchmark compound pairs.

## When to use

Use when benchmarking ModiFinder on MS/MS compound pairs whose true modification site is known, and shifted-peak substructure assignments must be restricted to those containing that site before scoring.

## When NOT to use

- The modification site of the unknown compound is genuinely unknown — oracle filtering requires ground truth and would leak the answer
- The goal is de novo fragment annotation rather than restricting shifted-peak assignments to a known modification site
- Pairs fail benchmark eligibility checks (e.g., precursor mass >= 2000 Da or precursor mass difference >= 50%), since the oracle sweep is defined on the eligible 12,909-pair benchmark

## Inputs

- Benchmark MS/MS compound pairs (known/unknown analogs) from GNPS spectral libraries
- Known compound structure with the true modification site annotated
- MS/MS peak lists preprocessed (peaks <1% of base peak removed, normalized to Euclidean norm 1)
- Substructure-to-peak assignments from MAGMa combinatorial fragmentation at 40 ppm tolerance

## Outputs

- Filtered substructure assignment sets per shifted peak (only substructures containing the true modification site)
- Per-pair counts of annotated shifted peaks after oracle filtering
- Likelihood probabilities over atoms of the known molecule (ModiFinder.generate_probabilities())
- MF-O / Average-distance evaluation scores per pair (BasicEvaluationEngine.evaluate_single())

## How to apply

Load benchmark MS/MS pairs from GNPS spectral libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) as ModiFinder Compound objects after peak preprocessing (drop peaks <1% of base peak, normalize to Euclidean norm 1). Annotate substructures via the MAGMaAnnotationEngine breadth-first combinatorial fragmentation (default maximum depth 2; sweep depths 2, 3, 4 to test fragmentation-depth effects) with 40 ppm m/z tolerance for substructure-to-peak assignment. Then instantiate ModiFinder in oracle mode: for each peak aligned as shifted in the known/unknown MS/MS bipartite matching, delete from its substructure assignment set any substructure that does not contain the true modification site. After filtering, count annotated shifted peaks per pair via the peak-to-fragment map, generate likelihood probabilities with ModiFinder.generate_probabilities(), and score with BasicEvaluationEngine.evaluate_single() using the Average-distance method. This oracle step measures the explainable ceiling of the approach — it cannot be used for genuine unknowns, only for controlled benchmark sweeps such as the 12,909-pair evaluation.

## Related tools

- **ModiFinder** (Provides the oracle mode that filters substructures lacking the true modification site from shifted-peak assignments, plus generate_probabilities() and BasicEvaluationEngine.evaluate_single() for scoring) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (MAGMaAnnotationEngine performs the breadth-first combinatorial fragmentation (depth 2-4) whose substructure assignments the oracle step filters)
- **MS/MS library search** (Source of the GNPS spectral-library compound pairs (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) on which the oracle filter is evaluated)

## Examples

```
from modifinder import Compound, ModiFinder; from modifinder.engines import MAGMaAnnotationEngine; mf = ModiFinder(known, unknown, annotation_engine=MAGMaAnnotationEngine(breaks=4), oracle=True); mf.generate_probabilities()
```

## Evaluation signals

- After oracle filtering, each remaining substructure assigned to a shifted peak contains the true modification site — spot-check assignments against the annotated known structure
- Depth sweep reproduces the reported 19% increase in annotated shifted peaks from depth 2 to depth 4
- Benchmark retention matches the reported 8,033 of 12,909 pairs (62%) retaining at least one explainable shifted peak
- MF-O evaluation scores are higher at depths 3-4 than at depth 2, as reported
- Per-pair depth sweep table and depth-level summary are fully populated in fragmentation_depth_sweep_results.csv with no missing pair entries

## Limitations

- Oracle mode presupposes knowledge of the true modification site, so results are an upper bound and not achievable on real unknowns
- The default MAGMa fragmentation depth of 2 may under-annotate shifted peaks; deeper depths (3-4) increase annotated shifted peaks but were evaluated only up to depth 4 in the sweep
- Filtering depends on upstream preprocessing thresholds (peaks <1% of base peak removed, 40 ppm substructure tolerance), so deviations shift which shifted peaks retain explainable substructures

## Evidence

- [readme] ModiFinder purpose: [evidence span withheld — anchor status: not_found]

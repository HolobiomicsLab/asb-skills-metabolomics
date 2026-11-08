---
name: msms-fragmentation-depth-sweep
description: Use when you need to assess how deeply MAGMa breadth-first fragmentation
  (depth 2 vs 3 vs 4) affects ModiFinder's shifted-peak annotation coverage and MF-O
  evaluation scores on an MS/MS analog pair benchmark.
when_to_use_negative:
- You only need modification-site prediction for new unknown compounds rather than
  a depth sensitivity analysis of the annotation engine
- Your peak pairs lack at least one explainable shifted peak after oracle filtering
  — the depth sweep is only meaningful on the 8,033-pair (62%) retained subset
- You are using a fragmentation engine other than MAGMa's breadth-first approach,
  since depth semantics differ across engines (e.g., MetFrag, MIDAS, MS-Finder)
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Orchestrates MS/MS peak alignment, oracle-mode substructure filtering, likelihood
    generation (generate_probabilities), and BasicEvaluationEngine scoring across
    depth settings
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Provides the MAGMaAnnotationEngine performing breadth-first combinatorial
    fragmentation whose breaks parameter (2, 3, 4) is swept
- name: MS/MS library search
  role: Source of the GNPS spectral library compound pairs forming the 12,909-pair
    benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:4edb98066192275af9cdb7d04b4b799365704e49d4b9c10c71180514797f669e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - MS/MS library search
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# msms-fragmentation-depth-sweep

## Summary

Sweeps the MAGMa combinatorial fragmentation depth (2, 3, 4) used by ModiFinder's substructure annotation and measures how fragmentation depth changes the number of annotated shifted MS/MS peaks and modification-site localization scores across a benchmark of compound pairs.

## When to use

Use when you need to assess how deeply MAGMa breadth-first fragmentation (depth 2 vs 3 vs 4) affects ModiFinder's shifted-peak annotation coverage and MF-O evaluation scores on an MS/MS analog pair benchmark.

## When NOT to use

- You only need modification-site prediction for new unknown compounds rather than a depth sensitivity analysis of the annotation engine
- Your peak pairs lack at least one explainable shifted peak after oracle filtering — the depth sweep is only meaningful on the 8,033-pair (62%) retained subset
- You are using a fragmentation engine other than MAGMa's breadth-first approach, since depth semantics differ across engines (e.g., MetFrag, MIDAS, MS-Finder)

## Inputs

- MS/MS compound pairs from GNPS spectral libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB), 12,909 pairs total
- Known compound structures with true modification site annotations (for oracle mode)
- MAGMaAnnotationEngine configuration (breaks/depth parameter)

## Outputs

- Per-pair fragmentation depth sweep table (fragmentation_depth_sweep_results.csv)
- Depth-level summary of annotated shifted peak counts and MF-O evaluation scores
- Verification of the 19% increase in annotated shifted peaks from depth 2 to depth 4

## How to apply

Load the GNPS spectral library benchmark compound pairs (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) with the ModiFinder Compound class using the standard preprocessing: drop peaks below 1% of the base peak, normalize peaks to a Euclidean norm of 1, and use 40 ppm substructure m/z tolerance. For each pair, instantiate ModiFinder in oracle mode (filtering out substructures that lack the true modification site) with the MAGMaAnnotationEngine at breaks=2, 3, and 4 in turn. After each annotation, count annotated shifted peaks per pair from the peak-to-fragment map, then generate likelihoods via ModiFinder.generate_probabilities() and score with BasicEvaluationEngine.evaluate_single() using the Average-distance method. The rationale is that deeper fragmentation assigns more substructures to peaks, which is verified by a reported 19% increase in annotated shifted peaks from depth 2 to depth 4 and higher MF-O scores at depths 3-4 versus depth 2; the eligible benchmark after the explainable-shifted-peak filter retains 8,033 of 12,909 pairs (62%).

## Related tools

- **ModiFinder** (Orchestrates MS/MS peak alignment, oracle-mode substructure filtering, likelihood generation (generate_probabilities), and BasicEvaluationEngine scoring across depth settings) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Provides the MAGMaAnnotationEngine performing breadth-first combinatorial fragmentation whose breaks parameter (2, 3, 4) is swept)
- **MS/MS library search** (Source of the GNPS spectral library compound pairs forming the 12,909-pair benchmark)

## Examples

```
from modifinder import Compound, MAGMaAnnotationEngine, BasicEvaluationEngine; pair = Compound.load_gnps_pair(...); annotator = MAGMaAnnotationEngine(breaks=4); modifinder = ModiFinder(known=known, unknown=unknown, mode='oracle', annotator=annotator); modifinder.generate_probabilities(); BasicEvaluationEngine.evaluate_single(modifinder)
```

## Evaluation signals

- Annotated shifted peak counts should increase monotonically with depth, totaling approximately a 19% increase from depth 2 to depth 4
- MF-O evaluation scores (BasicEvaluationEngine, Average-distance method) should be higher at depths 3-4 than at depth 2
- The filtered benchmark must retain 8,033 of 12,909 pairs (62%) after requiring at least one explainable shifted peak
- Per-pair output CSV must contain one row per pair per depth with peak counts and scores; verify column schema and pair count
- Oracle-mode invariant: no substructure assigned to a shifted peak may lack the true modification site

## Limitations

- Deeper fragmentation (depth 4) assigns more substructures per peak, which may dilute localization specificity even as coverage rises
- The benchmark requires pairs to pass eligibility checks (precursor mass < 2000 Da, precursor mass difference < 50%), so results do not generalize to pairs outside those bounds
- Oracle mode presumes the true modification site is known, so the sweep measures annotation coverage rather than real-world prediction accuracy on unknowns

## Evidence

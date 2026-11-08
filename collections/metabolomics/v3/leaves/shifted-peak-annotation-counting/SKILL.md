---
name: shifted-peak-annotation-counting
description: Use when quantifying how much explanatory signal (annotated shifted peaks)
  a benchmark of MS/MS analog pairs retains at increasing fragmentation depths, e.g.
  to justify a depth parameter or measure pair retention.
when_to_use_negative:
- The goal is modification site localization on a single unknown compound rather than
  quantifying annotation coverage across a benchmark of pairs
- Spectra lack a structurally related known/unknown pair or the true modification
  site is unknown, since oracle-mode shifted-peak filtering cannot be applied
- Precursor masses violate benchmark eligibility checks (precursor mass ≥ 2000 Da,
  or precursor mass difference ≥ 50%)
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ModiFinder
  role: 'Core framework: Compound class loads GNPS library pairs; oracle mode filters
    shifted-peak substructures; generate_probabilities() produces likelihoods'
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: MAGMaAnnotationEngine performs breadth-first combinatorial fragmentation at
    breaks=2, 3, 4 to assign substructures to shifted peaks
- name: MS/MS library search
  role: Source of the benchmark spectral library pairs (GNPS-MSMLS, NIH, BERKELEY-LAB
    libraries) loaded for counting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:4aa480225da7387f819cb02c3b96f2716dc6793000344da75a331e229ce558d5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - MS/MS library search
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# shifted-peak-annotation-counting

## Summary

Counts how many shifted MS/MS peaks in each known/unknown compound pair can be annotated with a substructure containing the modification site, by sweeping combinatorial fragmentation depth (MAGMa breaks=2–4) over a benchmark of structurally related pairs.

## When to use

Use when quantifying how much explanatory signal (annotated shifted peaks) a benchmark of MS/MS analog pairs retains at increasing fragmentation depths, e.g. to justify a depth parameter or measure pair retention.

## When NOT to use

- The goal is modification site localization on a single unknown compound rather than quantifying annotation coverage across a benchmark of pairs
- Spectra lack a structurally related known/unknown pair or the true modification site is unknown, since oracle-mode shifted-peak filtering cannot be applied
- Precursor masses violate benchmark eligibility checks (precursor mass ≥ 2000 Da, or precursor mass difference ≥ 50%)

## Inputs

- Benchmark MS/MS compound pairs (12,909 known/unknown analog pairs) from GNPS spectral libraries: GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB
- Known and unknown compound MS/MS spectra and structures consumable by the ModiFinder Compound class
- True modification site annotations for oracle-mode filtering

## Outputs

- Per-pair depth sweep table (annotated shifted peak counts at breaks=2, 3, 4 per pair)
- Depth-level summary (e.g. 19% increase in annotated shifted peaks from depth 2 to 4; 8,033/12,909 pairs, 62%, with at least one explainable shifted peak)
- fragmentation_depth_sweep_results.csv
- MF-O evaluation scores per depth from BasicEvaluationEngine (Average-distance method)

## How to apply

Load benchmark MS/MS compound pairs from the GNPS spectral libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) with the ModiFinder Compound class, applying the standard preprocessing: drop peaks below 1% of the base peak, normalize peaks to a Euclidean norm of 1, and use a 40 ppm substructure m/z tolerance. For each pair, instantiate ModiFinder in oracle mode (so substructures lacking the true modification site are filtered from shifted peaks) with the MAGMaAnnotationEngine, repeating the annotation at breaks=2, 3, and 4. After annotation at each depth, count the annotated shifted peaks per pair from the peak-to-fragment map, generate likelihood probabilities with ModiFinder.generate_probabilities(), and score predictions with BasicEvaluationEngine.evaluate_single() using the default Average-distance method. Aggregate the per-depth results and verify the reported trends: a 19% increase in annotated shifted peaks from depth 2 to depth 4, higher MF-O evaluation scores at depths 3–4 versus depth 2, and 8,033 of 12,909 pairs (62%) retaining at least one explainable shifted peak after filtering. Write the per-pair depth sweep table and per-depth summary to fragmentation_depth_sweep_results.csv.

## Related tools

- **ModiFinder** (Core framework: Compound class loads GNPS library pairs; oracle mode filters shifted-peak substructures; generate_probabilities() produces likelihoods) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (MAGMaAnnotationEngine performs breadth-first combinatorial fragmentation at breaks=2, 3, 4 to assign substructures to shifted peaks)
- **MS/MS library search** (Source of the benchmark spectral library pairs (GNPS-MSMLS, NIH, BERKELEY-LAB libraries) loaded for counting)

## Examples

```
from modifinder import Compound, MAGMaAnnotationEngine; mf = ModiFinder(known, unknown, annotation_engine=MAGMaAnnotationEngine(breaks=4), oracle=True); mf.run(); counts = sum(len(p) for p in mf.peak_to_fragment_map.values())
```

## Evaluation signals

- Annotated shifted peak counts increase monotonically with depth, reproducing the reported 19% increase from depth 2 to depth 4
- After filtering for at least one explainable shifted peak, 8,033 of 12,909 pairs (62%) are retained
- MF-O evaluation scores at depths 3–4 are higher than at depth 2 (BasicEvaluationEngine, Average-distance method)
- Output CSV contains both the per-pair depth sweep table and the depth-level summary
- Preprocessing invariants hold: peaks <1% of base peak removed and spectra normalized to Euclidean norm 1; substructure m/z matches within 40 ppm

## Limitations

- Oracle mode requires knowing the true modification site, so counting results are not achievable on fully unknown analogs
- Deeper fragmentation (breaks=4) yields more annotated shifted peaks but results were reported at depth 2 as the default maximum (a maximum depth of 2 in the base method)
- Findings are specific to this 12,909-pair benchmark built from the four GNPS libraries under the eligibility filters (precursor mass < 2000 Da, precursor mass difference < 50%)

## Evidence

- [readme] README confirms tool identity and version of record: [evidence span withheld — anchor status: not_found]

---
name: csv-result-table-generation
description: 'Use when a scoring/filtering analysis must be tabulated row-per-threshold:
  e.g. fraction of correct ZODIAC molecular formula annotations among compounds whose
  score exceeds each cutoff (0.7, 0.8, 0.9, 0.95, 0.99), written as a CSV.'
when_to_use_negative:
- Input is raw mzML/mzXML spectra that have not yet been processed with OpenMS and
  annotated by SIRIUS/ZODIAC — no scores exist to threshold
- The goal is structure identification (CSI:FingerID database search) rather than
  tabulating score-threshold accuracy of formula annotations
- No ground-truth formulas from spectral library search/manual validation are available,
  so 'fraction correct' cannot be computed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3173
tools:
- name: ZODIAC
  role: Source of the marginal (posterior) scores per compound that are thresholded
    in the accuracy table
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: SIRIUS
  role: Provides the top-ranked molecular formula candidates (e.g. top 50 per compound)
    that ZODIAC reranks; its correctness vs ground truth is tabulated
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42256-020-00234-6
    title: Database-independent molecular formula annotation using Gibbs sampling
      through ZODIAC
  source_content_hash: sha256:2a41e041c80714c5a3ddc43219813bfe66231bd84b3f3ee3b77ef2340008968b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/boecker-lab/sirius
  tools:
  - SIRIUS
  - ZODIAC
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# csv-result-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Consolidates parameter-threshold evaluation results — here, ZODIAC molecular formula annotation accuracy at a series of score cutoffs — into a structured per-threshold CSV table (e.g. zodiac_threshold_accuracy.csv) suitable for validation against reported values.

## When to use

Use when a scoring/filtering analysis must be tabulated row-per-threshold: e.g. fraction of correct ZODIAC molecular formula annotations among compounds whose score exceeds each cutoff (0.7, 0.8, 0.9, 0.95, 0.99), written as a CSV.

## When NOT to use

- Input is raw mzML/mzXML spectra that have not yet been processed with OpenMS and annotated by SIRIUS/ZODIAC — no scores exist to threshold
- The goal is structure identification (CSI:FingerID database search) rather than tabulating score-threshold accuracy of formula annotations
- No ground-truth formulas from spectral library search/manual validation are available, so 'fraction correct' cannot be computed

## Inputs

- Deposited SIRIUS/ZODIAC result files (from https://bio.informatik.uni-jena.de/data/) containing per-compound ZODIAC marginal scores for top-ranked molecular formula candidates
- Ground-truth molecular formulas per compound established by spectral library search and manual validation

## Outputs

- zodiac_threshold_accuracy.csv with one row per score threshold (threshold, fraction correct, fraction of compounds retained)

## How to apply

Download the deposited SIRIUS and ZODIAC result files from https://bio.informatik.uni-jena.de/data/. Parse the ZODIAC marginal (posterior) score assigned to each compound's top-ranked molecular formula candidate, and load the ground-truth formulas established by spectral library search plus manual validation. For each threshold (0.7, 0.8, 0.9, 0.95, 0.99), subset compounds whose ZODIAC score meets the cutoff, compute the fraction whose annotation matches the ground truth, and also record the fraction of compounds retained. Emit one row per threshold to zodiac_threshold_accuracy.csv. Validation hinges on the reported finding: accuracy at threshold 0.9 must exceed 96.5% correct annotations while retaining 52.05%–88.24% of compounds per dataset.

## Related tools

- **ZODIAC** (Source of the marginal (posterior) scores per compound that are thresholded in the accuracy table) — https://bio.informatik.uni-jena.de/software/sirius/
- **SIRIUS** (Provides the top-ranked molecular formula candidates (e.g. top 50 per compound) that ZODIAC reranks; its correctness vs ground truth is tabulated) — https://github.com/sirius-ms/sirius

## Examples

```
rows=[(t, df[df.zodiac_score>=t].formula.eq(df.ground_truth).mean()) for t in (0.7,0.8,0.9,0.95,0.99)]; pd.DataFrame(rows,columns=['threshold','accuracy']).to_csv('zodiac_threshold_accuracy.csv',index=False)
```

## Evaluation signals

- Accuracy at ZODIAC score threshold 0.9 exceeds 96.5% correct molecular formula annotations
- Fraction of compounds retained at threshold 0.9 falls in the 52.05%–88.24% range across the five datasets (dendroides, NIST1950, tomato, diatoms, mice stool)
- CSV contains exactly one row per evaluated threshold and every row's accuracy is non-decreasing as the threshold increases
- Each compound's ZODIAC annotation is matched against a non-missing ground-truth formula from spectral library search/manual validation before computing fractions

## Limitations

- A complete evaluation of novel molecular formula annotations would require full structural elucidation, which is experimentally infeasible — accuracy is only measurable for compounds with library-based ground truth
- Reproducing the underlying results requires a modified version of OpenMS 2.4.0 plus a separate patch file, not stock OpenMS
- Library-hit anchors have only a small effect on molecular formula annotations, so results may differ slightly depending on whether anchors were integrated

## Evidence

- [readme] SIRIUS framework scope: [evidence span withheld — anchor status: external_doc]

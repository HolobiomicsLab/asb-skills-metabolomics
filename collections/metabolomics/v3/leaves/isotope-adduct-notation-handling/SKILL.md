---
name: isotope-adduct-notation-handling
description: Use when candidate ion formulas must be generated for a MALDI matrix
  (e.g., DHB, C7H6O4) by enumerating intact and dehydrated matrix molecules with proton
  losses and Na/K adducts, to build a custom annotation database.
when_to_use_negative:
- When annotating biological metabolites rather than MALDI matrix cluster/off-sample
  ions — the enumeration is specific to matrix cluster chemistry
- When the matrix is not DHB or a matrix whose cluster model is not described in Keller
  and Li (2000); the (M, M-H2O, -H, +K, +Na) space must be redefined for other matrices
- When handling isotopic peak fine structure or isotopologue distributions — this
  model yields monoisotopic elemental formula strings only, not isotope envelope calculations
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: METASPACE
  role: Target annotation platform for which the custom DHB matrix-cluster molecular
    database of predicted formulas was built
  repo: https://github.com/metaspace2020/metaspace/
- name: manual expert review
  role: Validation of the predicted matrix cluster formulas alongside the combinatorial
    enumeration
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:ec35e27a65a8e37d64382bacd222450ddac6d67fc6ff757c76e02aa001cfa966
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - manual expert review
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# isotope-adduct-notation-handling

## Summary

A skill for enumerating matrix cluster and adduct ion formulas using the combinatorial notation n*M + p*(M-H2O) - x*H + y*K + z*Na, as used to build a custom DHB matrix-cluster molecular database for METASPACE annotation of MALDI-imaging datasets.

## When to use

Use when candidate ion formulas must be generated for a MALDI matrix (e.g., DHB, C7H6O4) by enumerating intact and dehydrated matrix molecules with proton losses and Na/K adducts, to build a custom annotation database.

## When NOT to use

- When annotating biological metabolites rather than MALDI matrix cluster/off-sample ions — the enumeration is specific to matrix cluster chemistry
- When the matrix is not DHB or a matrix whose cluster model is not described in Keller and Li (2000); the (M, M-H2O, -H, +K, +Na) space must be redefined for other matrices
- When handling isotopic peak fine structure or isotopologue distributions — this model yields monoisotopic elemental formula strings only, not isotope envelope calculations

## Inputs

- Base matrix molecule elemental formula M (e.g., C7H6O4 for 2,5-dihydroxybenzoic acid, DHB)
- Cluster-generating model specification from Keller and Li (2000) defining the (n, p, x, y, z) ranges
- Adduct species definitions: dehydrated unit (M-H2O), proton loss (-H), potassium (+K), sodium (+Na)

## Outputs

- CSV file of predicted matrix cluster molecular formulas ('DHB mc mol formulas, predicted.csv') containing 353 formula rows
- Normalized elemental formula strings for each valid (n, p, x, y, z) combination

## How to apply

Set the base matrix molecule M to the matrix elemental formula (for 2,5-dihydroxybenzoic acid, M = C7H6O4) following the Keller and Li (2000) cluster-generating model. Enumerate the combinatorial space of n intact M molecules and p water-loss species (M-H2O), then apply cation/adduct modifications -x*H + y*K + z*Na over the valid integer ranges of (n, p, x, y, z). For each valid combination, sum the element counts across all terms, normalize the resulting chemical formula into standard elemental notation (e.g., C, H, O, K, Na counts merged and ordered), and emit one formula string per combination. Export the full enumerated set to a CSV file such as 'DHB mc mol formulas, predicted.csv'. In this project the enumeration produced exactly 353 predicted molecular formulas, which were made available in the DHB matrix clusters directory of the metaspace2020/offsample repository and used for METASPACE annotation of MALDI-imaging data.

## Related tools

- **METASPACE** (Target annotation platform for which the custom DHB matrix-cluster molecular database of predicted formulas was built) — https://github.com/metaspace2020/metaspace/
- **manual expert review** (Validation of the predicted matrix cluster formulas alongside the combinatorial enumeration)

## Examples

```
python generate_dhb_clusters.py --base C7H6O4 --output 'DHB mc mol formulas, predicted.csv'  # yields 353 formulas as n*C7H6O4 + p*(M-H2O) - x*H + y*K + z*Na
```

## Evaluation signals

- The output CSV 'DHB mc mol formulas, predicted.csv' exists in the 'DHB matrix clusters' directory and parses correctly
- The CSV contains exactly 353 molecular formula rows, matching the reported count in the repository README
- Each formula string uses consistent normalized elemental notation (merged element counts for C, H, O, K, Na)
- Spot-check individual (n, p, x, y, z) combinations: element counts equal n*C7H6O4 + p*(C7H4O3) - x*H + y*K + z*Na
- All enumerated formulas carry non-negative integer element counts (no chemically invalid negative counts)

## Limitations

- The enumeration is grounded in the Keller and Li (2000) model for DHB; other matrices require a new cluster-generating model
- The model covers only proton-loss and Na/K cation adducts of intact and dehydrated DHB units; other adduct chemistries are out of scope
- Off-sample matrix cluster signals must be recognized and filtered downstream (e.g., by the METASPACE off-sample filter); formula enumeration alone does not solve pixel-level segmentation of off-sample regions

## Evidence

- [other] candidate DHB matrix cluster ion formulas are enumerated as n*M + p*(M-H2O) - x*H + y*K + z*Na, where M is the DHB matrix molecular formula C7H6O4: [evidence span withheld — anchor status: not_found]
- [readme] README: DHB matrix clusters generated per Keller and Li (2000), 353 formulas: "We have generated DHB matrix clusters according to [(Keller and Li, 2000)](./DHB%20matrix%20clusters/Keller%20and%20Li%2C%202000.pdf). This resulted in 353 molecular formulas available"
- [intro] Intro: derived properties of matrix clusters using a cluster-generating model for DHB: "we investigated the off-sample images corresponding to the most popular MALDI matrix (2,5-dihydroxybenzoic acid, DHB) [18] and derived properties of matrix clusters using a cluster-generating"

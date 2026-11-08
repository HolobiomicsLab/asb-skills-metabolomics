---
name: spectral-cosine-similarity-scoring-of-structural-permutations
description: Decision support for benchmarking modification-site localization when
  a simulated in-silico fragmentation alternative to ModiFinder's likelihood scoring
  is needed, using MS2-unknown plus an S-known structure with a known modification
  substructure.
when_to_use_negative:
- When ModiFinder's alignment-based likelihood scoring (MF-R/MF-O) is available —
  in the article's benchmark this CFM-ID permutation approach scored on average below
  even the Random Choice and Random Distribution baselines.
- When the modification substructure cannot be inferred from the known-vs-unknown
  structure pair (no defined modification to permute).
- When precursor/adduct conditions differ from the [M+H]+ trained CFM-ID models used
  here (e.g., negative-mode or non-protonated adducts).
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: CFM-ID
  role: Predicts MS/MS spectra for each permuted analog structure so cosine similarity
    against MS2-unknown can score modification-site candidates
  repo: hub.docker.com/r/wishartlab/cfmid
- name: ModiFinder
  role: Provides the benchmark framework and the Average-distance evaluation method
    against which the CFM-ID permutation scores are compared
  repo: github.com/Wang-Bioinformatics-Lab/ModiFinder_base
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:8c448718eb43f381812d343525b2850441f99e3cf9a7b1f557bf33df7cede4db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Spectral cosine-similarity scoring of structural permutations

## Summary

Baseline alternative for modification-site localization: permute a modification substructure across the known analog's bonds, predict an MS/MS spectrum for each permuted structure with CFM-ID, and score each prediction against the unknown's MS2 spectrum by cosine similarity to rank candidate modification sites.

## When to use

Decision support for benchmarking modification-site localization when a simulated in-silico fragmentation alternative to ModiFinder's likelihood scoring is needed, using MS2-unknown plus an S-known structure with a known modification substructure.

## When NOT to use

- When ModiFinder's alignment-based likelihood scoring (MF-R/MF-O) is available — in the article's benchmark this CFM-ID permutation approach scored on average below even the Random Choice and Random Distribution baselines.
- When the modification substructure cannot be inferred from the known-vs-unknown structure pair (no defined modification to permute).
- When precursor/adduct conditions differ from the [M+H]+ trained CFM-ID models used here (e.g., negative-mode or non-protonated adducts).

## Inputs

- MS2-unknown spectrum (preprocessed: peaks <1% of base peak removed, normalized to Euclidean norm 1)
- S-known structure of the structurally related known analog
- Modification substructure derived from the unknown-vs-known structural difference
- Bond table of S-known (matching bond types first, fallback to other types)

## Outputs

- Per-atom likelihood scores over S-known (cosine similarity of each permuted structure's CFM-ID-predicted MS/MS vs MS2-unknown)
- Permutation-to-predicted-spectrum mapping from the CFM-ID run
- Average-distance evaluation scores comparable against RC, RD, MF-R, MF-O baselines

## How to apply

Derive the modification substructure from the difference between the unknown and known analog structures, then exhaustively permute it across bonds of S-known, trying matching bond types first and falling back to other bond types so every atom can serve as a candidate site. For each permuted analog, run the CFM-ID docker container (wishartlab/cfmid) with prob_thresh 0.001 and the [M+H]+ trained models to predict its MS/MS spectrum. Compute cosine similarity between each predicted spectrum and the MS2-unknown spectrum, and use that similarity as the atom's likelihood score for being the modification site. Compare these scores to ModiFinder's MF-R and MF-O distributions and to the Random Choice and Random Distribution baselines using the Average-distance evaluation (exponential decay by 2D graph distance normalized by graph diameter). In the article's benchmark this CFM-ID approach scored on average worse than the RC and RD random baselines.

## Related tools

- **CFM-ID** (Predicts MS/MS spectra for each permuted analog structure so cosine similarity against MS2-unknown can score modification-site candidates) — hub.docker.com/r/wishartlab/cfmid
- **ModiFinder** (Provides the benchmark framework and the Average-distance evaluation method against which the CFM-ID permutation scores are compared) — github.com/Wang-Bioinformatics-Lab/ModiFinder_base

## Examples

```
docker run wishartlab/cfmid cfm-id --predict --prob_thresh 0.001 --model [M+H]+ permuted_structure.mol
```

## Evaluation signals

- Every atom of S-known receives exactly one likelihood score from a cosine similarity value in [0, 1].
- The number of valid permuted structures equals the number of feasible bond placements of the modification substructure (matching-type placements plus fallback placements).
- Reproduced average deltas (CFM-ID vs MF-R) should fall near the reported +0.266, and CFM-ID scores should on average be below both RC and RD.
- All cosine similarities are computed against the same normalized MS2-unknown spectrum (peaks <1% of base peak removed, Euclidean norm 1).
- Aggregate win fractions when comparing CFM-ID against MF-R/MF-O should be reported per the pairwise Average-distance comparison, with CFM-ID losing on average.

## Limitations

- In the article's benchmark, the CFM-ID permutation approach scored worse than the Random Choice and Random Distribution baselines on average, so it is a weak alternative rather than a competitive method.
- Requires an explicit modification substructure to permute; it cannot operate when the modification is unknown or ambiguous.
- Depends on CFM-ID's trained [M+H]+ models and the 0.001 prob_thresh setting; other adducts or thresholds may degrade prediction quality.
- Cosine similarity of a single predicted spectrum is a coarse score — it rewards a constant-score algorithm under simple top-1 accuracy, which is why the Average-distance metric is used instead.

## Evidence

- [methods] CFM-ID alternative by permuting the modification substructure (derived from the unknown compound's structure) across S-known with matching then fallback bond types: [evidence span withheld — anchor status: not_found]
- [methods] the docker container provided [https://hub.docker.com/r/wishartlab/cfmid is used with 0.001 for 'prob_thresh': [evidence span withheld — anchor status: not_found]
- [results] The Average-distance evaluation is adopted as the main evaluation metric (referred to as 'Evaluation Score') because it balances proximity cover and ambiguity cover: [evidence span withheld — anchor status: not_found]
- [readme] SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites and other "small molecules of biological interest".: [evidence span withheld — anchor status: external_doc]

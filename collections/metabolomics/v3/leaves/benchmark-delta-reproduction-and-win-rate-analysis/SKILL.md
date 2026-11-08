---
name: benchmark-delta-reproduction-and-win-rate-analysis
description: Use when you need to independently reproduce a paper's headline benchmark
  numbers (average deltas and pairwise win rates) by re-running random and tool-based
  baselines on the identical filtered analog-pair benchmark and comparing aggregated
  metrics.
when_to_use_negative:
- When the benchmark pairs are not pre-filtered to the exact eligibility rules (precursor
  < 2000 Da, < 50% mass difference, shared M+H adduct, single-site modification) —
  deltas will not be comparable to the reported values
- When evaluating top-1 accuracy or ranking-only metrics, since a constant-score algorithm
  can saturate top-1 accuracy and Average-distance is required to differentiate it
  from informative likelihood distributions
- When the unknown compound's modification substructure is unavailable or ambiguous,
  because CFM-ID baseline permutations and the Average-distance decay-weighted scoring
  depend on mapping the modification site onto the S-known graph
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Generates the MF-R (random) and MF-O (oracle) likelihood distributions whose
    deltas and win rates are being reproduced; also provides the Average-distance
    Evaluation Score used for all comparisons.
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: CFM-ID
  role: 'Provides the site-localization alternative baseline: permutes the modification
    substructure across S-known, predicts MS/MS for each permuted analog in the docker
    container (0.001 prob_thresh, [M+H]+ models), and scores cosine similarity against
    MS2-unknown as its likelihood.'
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: SIRIUS
  role: Supplies predicted fragment formulas used to refine substructure assignments
    for the MS2-known and MS2-unknown spectra before scoring, ensuring the baselines
    and ModiFinder variants operate on the same peak-substructure mappings.
  repo: https://github.com/boecker-lab/sirius/releases
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:28ceb99799177d09f33d49f76244ac648fc887611005425d57e5581209d52a07
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - CFM-ID
  - ModiFinder
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Benchmark-delta reproduction and win-rate analysis

## Summary

Reproduce the comparative performance deltas reported by the ModiFinder paper — average Evaluation-Score gains of ModiFinder-random (MF-R) and ModiFinder-oracle (MF-O) over Random Choice (RC), Random Distribution (RD), and CFM-ID site-localization baselines, plus the pairwise win fractions (81%/80% MF-R vs RC/RD; 85% MF-O) — by re-running each baseline on the same eligibility-filtered GNPS analog pairs and aggregating per-pair Average-distance scores into a baseline_comparison_results.csv table.

## When to use

Use when you need to independently reproduce a paper's headline benchmark numbers (average deltas and pairwise win rates) by re-running random and tool-based baselines on the identical filtered analog-pair benchmark and comparing aggregated metrics.

## When NOT to use

- When the benchmark pairs are not pre-filtered to the exact eligibility rules (precursor < 2000 Da, < 50% mass difference, shared M+H adduct, single-site modification) — deltas will not be comparable to the reported values
- When evaluating top-1 accuracy or ranking-only metrics, since a constant-score algorithm can saturate top-1 accuracy and Average-distance is required to differentiate it from informative likelihood distributions
- When the unknown compound's modification substructure is unavailable or ambiguous, because CFM-ID baseline permutations and the Average-distance decay-weighted scoring depend on mapping the modification site onto the S-known graph

## Inputs

- Eligibility-filtered benchmark analog pairs (precursor < 2000 Da; precursor mass difference < 50% of the smaller compound; shared M+H adduct; structures differing at exactly one modification site) from GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, and BERKELEY-LAB
- S-known structure with all atom positions for random-site simulation and CFM-ID substructure permutation
- MS2-known and MS2-unknown peak lists (intensities < 1% of base peak removed; peaks normalized to Euclidean norm 1) for the cosine-similarity scoring of the CFM-ID baseline
- CFM-ID docker container (wishartlab/cfmid) with 0.001 prob_thresh and [M+H]+ trained models

## Outputs

- Per-pair Average-distance Evaluation Scores for RC, RD, CFM-ID, MF-R and MF-O likelihood distributions
- Aggregated average deltas: MF-R vs RC (+0.181), MF-R vs RD (+0.180), MF-O vs RC (+0.208), MF-O vs RD (+0.207), MF-R vs CFM-ID (+0.266)
- Pairwise win fractions: MF-R beats RC in 81% of pairs, MF-R beats RD in 80% of pairs, MF-O beats RC/RD in 85% of pairs
- baseline_comparison_results.csv containing all deltas, win rates and per-pair scores

## How to apply

1. Start from the eligibility-filtered analog pairs (precursor < 2000 Da, precursor mass difference < 50% of the smaller compound, shared M+H adduct, structures differing at exactly one modification site) drawn from GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, and BERKELEY-LAB libraries. 2. Simulate the two random baselines per pair: Random Choice assigns the modification site by uniformly picking one atom, and Random Distribution draws each atom's likelihood score from a Normal(0,1) distribution. 3. Run the CFM-ID alternative: permute the modification substructure across S-known with matching then fallback bond types, predict MS/MS for each permuted analog via the CFM-ID docker container with prob_thresh 0.001 and [M+H]+ trained models, and score each permutation's cosine similarity against MS2-unknown as its likelihood. 4. Score RC, RD, CFM-ID, MF-R and MF-O likelihood distributions on every pair using the Average-distance evaluation (exponential decay by 2D graph distance normalized by graph diameter), which was adopted because simple top-1 accuracy would reward a constant-score algorithm. 5. Aggregate the average deltas (MF-R vs RC/RD, MF-O vs RC/RD, MF-R vs CFM-ID) and the pairwise win fractions, then write all deltas, win rates and per-pair scores to baseline_comparison_results.csv. 6. Validate the reproduction against the reported values: +0.181/+0.180 for MF-R, +0.208/+0.207 for MF-O, and +0.266 for MF-R vs CFM-ID, with CFM-ID scoring below the random baselines on average.

## Related tools

- **ModiFinder** (Generates the MF-R (random) and MF-O (oracle) likelihood distributions whose deltas and win rates are being reproduced; also provides the Average-distance Evaluation Score used for all comparisons.) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **CFM-ID** (Provides the site-localization alternative baseline: permutes the modification substructure across S-known, predicts MS/MS for each permuted analog in the docker container (0.001 prob_thresh, [M+H]+ models), and scores cosine similarity against MS2-unknown as its likelihood.) — https://hub.docker.com/r/wishartlab/cfmid
- **SIRIUS** (Supplies predicted fragment formulas used to refine substructure assignments for the MS2-known and MS2-unknown spectra before scoring, ensuring the baselines and ModiFinder variants operate on the same peak-substructure mappings.) — https://github.com/boecker-lab/sirius/releases

## Examples

```
python baseline_comparison.py --pairs filtered_pairs.tsv --sknown structures.sdf --ms2known spectra.mgf --ms2unknown spectra.mgf --cfmid-docker wishartlab/cfmid --prob-thresh 0.001 --adduct [M+H]+ --output baseline_comparison_results.csv
```

## Evaluation signals

- Reproduced MF-O vs RC and MF-O vs RD average deltas fall within tolerance of +0.208 and +0.207 respectively
- Reproduced MF-R vs RC and MF-R vs RD average deltas fall within tolerance of +0.181 and +0.180, and the MF-R win rates land near 81% (RC) and 80% (RD)
- CFM-ID site-localization approach scores worse than the RC and RD random baselines on average across the same filtered analog pairs
- baseline_comparison_results.csv contains a per-pair score for every filtered analog pair and the aggregate deltas/win rates match the reported values
- Every pair in the benchmark satisfies all four eligibility checks (precursor < 2000 Da, mass difference < 50%, shared M+H adduct, single-site structural difference) before scoring

## Limitations

- The deltas and win rates are sensitive to the strict pair-eligibility filters; using looser or different filters will not reproduce the reported numbers.
- Simple top-1 accuracy is unsuitable as the comparison metric because a constant-score algorithm that assigns every atom the same score can achieve perfect top-1 accuracy; only the Average-distance metric with exponential decay by 2D graph distance normalized by graph diameter captures proximity and ambiguity simultaneously.
- CFM-ID's performance is not guaranteed to be worse than random for all pairs; the reported +0.266 delta is an aggregate over the entire benchmark, and per-pair variation exists.
- The reproduced CFM-ID baseline depends on matching-then-fallback bond-type permutation and the 0.001 prob_thresh / [M+H]+ trained models; alternative settings will change its likelihood distribution and thus the deltas.

## Evidence

- [methods] Average-distance evaluation adopted because it balances proximity and ambiguity: [evidence span withheld — anchor status: not_found]
- [methods] Random baselines defined per pair: [evidence span withheld — anchor status: not_found]
- [methods] CFM-ID baseline procedure and parameters: [evidence span withheld — anchor status: not_found]
- [results] Win rates reported: [evidence span withheld — anchor status: not_found]
- [methods] CFM-ID docker configuration: [evidence span withheld — anchor status: not_found]
- [readme] README confirms ModiFinder tool availability: [evidence span withheld — anchor status: not_found]

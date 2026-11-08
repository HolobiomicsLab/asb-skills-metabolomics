---
name: random-baseline-simulation-for-site-localization
description: Decision support for benchmarking modification site-localization methods
  on MS/MS analog pairs, when top-1 accuracy would be mis and a proximity+ambiguity-balanced
  null baseline is needed.
when_to_use_negative:
- The analysis goal is plain top-1 hit accuracy on spectra-structure matches — this
  skill addresses site localization on analog pairs, where a constant-score algorithm
  would tie for first place under top-1 accuracy
- Benchmark pairs violate the eligibility checks (precursor ≥ 2000 Da, mass difference
  ≥ 50% of the smaller compound, different adducts, or structures differing at more
  than one modification site)
- The MS/MS peak alignment requires unshifted peak intensity comparison as the primary
  signal rather than likelihood distributions over atoms
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ModiFinder
  role: Provides the MF-R and MF-O likelihood distributions and the Average-distance
    scoring implementation against which the random baselines are compared
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: CFM-ID
  role: Runs as a docker container (prob_thresh 0.001, [M+H]+ trained models) to predict
    MS/MS for each permuted analog so permutation cosine similarities form the site-localization
    alternative
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: SIRIUS
  role: v5.6.3 runnable script processes the mgf of unknowns before baseline scoring
    in the benchmark pipeline
  repo: https://github.com/boecker-lab/sirius/releases
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:e37d73e5c60f032ba4e9fecd80558ba1af3069962b597428e42d1ed829c5426c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - CFM-ID
  - ModiFinder
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# random-baseline-simulation-for-site-localization

## Summary

Reproduces the ModiFinder paper's null baselines for modification site localization: Random Choice (uniform atom pick) and Random Distribution (Normal(0,1) atom scores), plus the CFM-ID permutation alternative, all scored with the Average-distance (proximity/ambiguity cover) metric on the GNPS-derived analog benchmark.

## When to use

Decision support for benchmarking modification site-localization methods on MS/MS analog pairs, when top-1 accuracy would be misleading and a proximity+ambiguity-balanced null baseline is needed.

## When NOT to use

- The analysis goal is plain top-1 hit accuracy on spectra-structure matches — this skill addresses site localization on analog pairs, where a constant-score algorithm would tie for first place under top-1 accuracy
- Benchmark pairs violate the eligibility checks (precursor ≥ 2000 Da, mass difference ≥ 50% of the smaller compound, different adducts, or structures differing at more than one modification site)
- The MS/MS peak alignment requires unshifted peak intensity comparison as the primary signal rather than likelihood distributions over atoms

## Inputs

- Eligibility-filtered analog pairs (S-known structure, unknown structure with exactly one modification site, MS2-known and MS2-unknown spectra, precursor masses and adduct)
- Known compound 2D molecular graph (S-known) for atom enumeration
- Modification substructure derived from the unknown compound's structure (for the CFM-ID permutation arm)
- GNPS library spectra from GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB

## Outputs

- Per-pair Average-distance scores for RC, RD, CFM-ID, MF-R, and MF-O likelihood distributions
- Average deltas vs baselines (reported ≈ +0.181/+0.180 MF-R vs RC/RD, +0.208/+0.207 MF-O, +0.266 MF-R vs CFM-ID)
- Pairwise win fractions (e.g. MF-R beats RC/RD in ~81%/80% of pairs)
- baseline_comparison_results.csv with per-pair scores, deltas, and win rates

## How to apply

Load the eligibility-filtered analog pairs (precursor < 2000 Da, mass difference < 50% of the smaller compound, shared M+H adduct, structures differing at exactly one modification site) built from the GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, and BERKELEY-LAB libraries. For each pair simulate Random Choice by uniformly picking one atom of S-known as the modification site, and Random Distribution by drawing each atom's likelihood from Normal(0,1). Run the CFM-ID alternative by permuting the modification substructure across S-known (matching bond types first, then fallback), predicting MS/MS per permutation via the CFM-ID docker container with prob_thresh 0.001 and [M+H]+ trained models, and scoring each permutation's cosine similarity against MS2-unknown as its likelihood. Score every likelihood distribution with the Average-distance evaluation, which exponentially decays by 2D graph distance normalized by graph diameter, and aggregate average deltas and pairwise win fractions. The rationale: Average-distance is adopted as the 'Evaluation Score' because it balances proximity cover and ambiguity cover — top-1 accuracy would reward a constant-score algorithm.

## Related tools

- **ModiFinder** (Provides the MF-R and MF-O likelihood distributions and the Average-distance scoring implementation against which the random baselines are compared) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **CFM-ID** (Runs as a docker container (prob_thresh 0.001, [M+H]+ trained models) to predict MS/MS for each permuted analog so permutation cosine similarities form the site-localization alternative) — https://hub.docker.com/r/wishartlab/cfmid
- **SIRIUS** (v5.6.3 runnable script processes the mgf of unknowns before baseline scoring in the benchmark pipeline) — https://github.com/boecker-lab/sirius/releases

## Examples

```
python scripts/baseline_comparison.py --pairs analog_benchmark.csv --baselines random_choice,random_distribution,cfm_id --out baseline_comparison_results.csv
```

## Evaluation signals

- Reproduced MF-O vs RC and MF-O vs RD average deltas fall within tolerance of +0.208 and +0.207; MF-R deltas within +0.181/+0.180
- MF-R win rates against RC and RD land near 81% and 80% of pairs; MF-O near 85%
- The CFM-ID approach scores worse than Random Choice and Random Distribution on average (negative delta, consistent with the reported +0.266 advantage of MF-R)
- Every pair's RC distribution is a single uniformly picked atom and the RD distribution draws one Normal(0,1) sample per atom, so scores vary per pair but are unbiased over repeated draws
- Per-pair Average-distance values are bounded by the exponential-decay/graph-diameter normalization, and all five methods are scored on the identical filtered pair set

## Limitations

- Average-distance rewards distributions concentrated near the true site without punishing diffuse near-miss mass, so it must be reported alongside the ambiguity-cover trade-off it embodies
- Random Choice and Random Distribution are stochastic; deltas carry sampling variance and should be verified against the reported means rather than single-seed values
- The CFM-ID arm depends on docker-container availability, prob_thresh 0.001 settings, and [M+H]+ trained models; bond-type fallback permutations can miss valid sites when no matching bond type exists
- Baseline simulation is only meaningful on the eligibility-filtered pair set (single modification site, shared M+H adduct, < 2000 Da precursors); extending to multi-site or different-adduct pairs is out of scope

## Evidence

- [methods] Average-distance balances proximity and ambiguity when scoring: [evidence span withheld — anchor status: not_found]
- [methods] Random Choice and Random Distribution baselines are defined per pair: [evidence span withheld — anchor status: not_found]
- [methods] CFM-ID docker container parameters used in the benchmark arm: [evidence span withheld — anchor status: not_found]
- [methods] Average-distance scoring uses exponential decay by 2D graph distance: [evidence span withheld — anchor status: not_found]

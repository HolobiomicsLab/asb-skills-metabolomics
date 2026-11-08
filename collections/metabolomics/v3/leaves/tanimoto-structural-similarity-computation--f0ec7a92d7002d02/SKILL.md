---
name: tanimoto-structural-similarity-computation
description: Use when MS/MS spectra carry associated chemical structures (e.g. GNPS/MGF
  entries) and you need pairwise structural similarity scores to evaluate spectral
  similarity measures or to bin/pair spectra for benchmarking.
when_to_use_negative:
- Input spectra lack associated chemical structures, so no fingerprints can be computed
  (Tanimoto requires structure annotations)
- You need a spectral (peak-based) similarity such as Cosine or Modified Cosine rather
  than a structural ground truth
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: RDKit
  role: Generates Daylight fingerprints from compound structures and computes the
    Tanimoto score for each query-reference pair
- name: TransExION
  role: Repository script ms_compute_structural_similarity.py wraps the Tanimoto computation
    and emits .ref similarity matrices consumed by training and testing scripts
  repo: https://github.com/banhdzui/TransExION
- name: pyteomics
  role: Listed as a repository dependency supporting MGF spectrum parsing
  repo: https://github.com/banhdzui/TransExION
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:7859edd604b1838ecea6c53b8d5a338377458f927e0a969ddc81d872cd1929bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/banhdzui/TransExION
  tools:
  - Cosine similarity
  - MS2DeepScore
  - Matplotlib
  - Modified Cosine similarity
  - PyTorch
  - RDKit
  - Spec2Vec
  - TransExION
  - pyteomics
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# tanimoto-structural-similarity-computation

## Summary

Compute Tanimoto structural similarity between compounds of MS/MS query and reference spectra from Daylight fingerprints using RDKit, providing the ground-truth structural similarity against which predicted spectral similarity (e.g. TransExION, Cosine, Spec2Vec, MS2DeepScore) is validated.

## When to use

Use when MS/MS spectra carry associated chemical structures (e.g. GNPS/MGF entries) and you need pairwise structural similarity scores to evaluate spectral similarity measures or to bin/pair spectra for benchmarking.

## When NOT to use

- Input spectra lack associated chemical structures, so no fingerprints can be computed (Tanimoto requires structure annotations)
- You need a spectral (peak-based) similarity such as Cosine or Modified Cosine rather than a structural ground truth

## Inputs

- MGF-format MS/MS spectrum files with associated compound structures (e.g. GNPS_MassBank_train.mgf, GNPS_MassBank_test.mgf)
- Spectrum pair definitions (query-reference pairs to score)

## Outputs

- Structural similarity matrix file (.ref) of pairwise Tanimoto scores, e.g. GNPS_MassBank_train.ref
- Per-pair Tanimoto ground-truth scores used for binned evaluation (10 equal-width bins, 0.0–1.0, k<=3 pairs per bin) and precision-recall analysis at cut-offs 0.6/0.7/0.8/0.9

## How to apply

For each query-reference spectrum pair, obtain the compound structures from the spectra metadata, generate Daylight-type fingerprints with RDKit, and compute the Tanimoto score as the fingerprint intersection over union. The TransExION repository packages this in ms_compute_structural_similarity.py, which takes an MGF reference file, an MGF query file, and writes a structural similarity matrix (.ref) usable directly by training/testing scripts. The resulting Tanimoto scores serve two roles: as regression targets when training TransExION (via --db-ref/--query-ref files) and as ground truth for benchmark evaluation, where they are split into B=10 equal-width bins between 0.0 and 1.0 with at most k=3 pairs sampled per bin per test spectrum, or thresholded at Tanimoto cut-offs of 0.6–0.9 to define structural analogues for precision-recall analysis.

## Related tools

- **RDKit** (Generates Daylight fingerprints from compound structures and computes the Tanimoto score for each query-reference pair)
- **TransExION** (Repository script ms_compute_structural_similarity.py wraps the Tanimoto computation and emits .ref similarity matrices consumed by training and testing scripts) — https://github.com/banhdzui/TransExION
- **pyteomics** (Listed as a repository dependency supporting MGF spectrum parsing) — https://github.com/banhdzui/TransExION

## Examples

```
python ms_compute_structural_similarity.py --db GNPS_MassBank_train.mgf --query GNPS_MassBank_val.mgf --out GNPS_MassBank_val.ref
```

## Evaluation signals

- The .ref output contains a finite Tanimoto score in [0.0, 1.0] for every requested query-reference pair
- When used for binned evaluation, 11,425 mergedGNPS testing pairs are obtained after equal-width binning into 10 bins with at most k=3 pairs per bin per test spectrum
- Downstream validation should reproduce reported correlations against the Tanimoto ground truth: TransExION r = 0.811, Cosine 0.489, Modified Cosine 0.538, Spec2Vec 0.557, MS2DeepScore 0.755 on mergedGNPS
- Precision-recall validation: TransExION curves should show a better precision/recall combination than baselines at each Tanimoto cut-off (0.6, 0.7, 0.8, 0.9)
- Squared-error box-plots per Tanimoto bin should show TransExION with low SE and minimal variation across bins (Fig. 6 pattern)

## Limitations

- Tanimoto on Daylight fingerprints is a proxy for structural similarity; spectral similarity methods are evaluated against it, so errors in fingerprint choice propagate to all downstream benchmarks
- The balanced binned pairing protocol (B=10 bins, k<=3 pairs per bin) determines which pairs enter evaluation; deviating binning changes the test-pair count and comparability with reported results

## Evidence

- [methods] Compute the Tanimoto score on Daylight fingerprints for every query-reference pair as ground-truth structural similarity using RDKit via ms_compute_structural_similarity.py: [evidence span withheld — anchor status: not_found]
- [readme] README usage of the script: "To compute the structral similarity matrix between MS/MS spectra, the script ```ms_compute_structural_similarity.py``` can be used."
- [methods] Divide the Tanimoto scores into B=10 equal-width bins between 0.0 and 1.0, compute the squared error (SE = squared distance between predicted and Tanimoto score) per pair per method: [evidence span withheld — anchor status: not_found]
- [methods] Define high structural similarity using four Tanimoto cut-offs (0.6, 0.7, 0.8, 0.9) on Daylight-fingerprint Tanimoto scores: [evidence span withheld — anchor status: not_found]
- [discussion] Tanimoto cut-off range claim: "It also outperforms the deep-learning model MS2DeepScore in both test sets, especially when nearly-identical structure matching is desired"

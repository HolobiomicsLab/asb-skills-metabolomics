---
name: spectral-cosine-similarity-ranking
description: Use when an experimental MS/MS spectrum is unannotated but a candidate
  compound list exists (e.g. from precursor mass) and in silico predicted spectra
  are available to score each candidate for identification.
when_to_use_negative:
- Candidate spectra come from libraries that only provide one collision energy per
  compound (e.g. GNPS, MSDial, MassBank) — the paper notes these are often under-curated
  and merged-CE comparison is not possible
- Predicting or reconstructing the actual spectrum intensities is the goal — this
  skill only produces a binary peak-presence-based ranking score
- Input spectra are in negative mode or contain non-supported atom types or precursor
  types — outside SingleFrag's validated scope
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SingleFrag
  role: Generates the per-molecule in silico predicted spectra used as the ranking
    reference against the experimental spectrum
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds candidate SMILES into the 300-dimensional vector that is the input
    to SingleFrag's ANN peak-prediction models
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:4cc9084fdd2e8312aa6a79fe55f523496a66b9b5247bf6f4550cbf7aade4f5ab
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# spectral-cosine-similarity-ranking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Ranks candidate compounds by cosine similarity between an unannotated empirical MS/MS spectrum and in silico predicted spectra from a ~1.9M-molecule SingleFrag library, evaluating Top-1/Top-5/Top-10 annotation accuracy. The paper's own skill: predicted spectra are used purely as a ranking score for metabolite annotation, not for spectrum reconstruction.

## When to use

Use when an experimental MS/MS spectrum is unannotated but a candidate compound list exists (e.g. from precursor mass) and in silico predicted spectra are available to score each candidate for identification.

## When NOT to use

- Candidate spectra come from libraries that only provide one collision energy per compound (e.g. GNPS, MSDial, MassBank) — the paper notes these are often under-curated and merged-CE comparison is not possible
- Predicting or reconstructing the actual spectrum intensities is the goal — this skill only produces a binary peak-presence-based ranking score
- Input spectra are in negative mode or contain non-supported atom types or precursor types — outside SingleFrag's validated scope

## Inputs

- Unannotated experimental MS/MS spectrum (HMDB-derived, positive mode, 2-decimal m/z)
- Candidate compound list with SMILES (filtered by precursor mass at 1 or 10 ppm tolerance)
- In silico predicted spectra for candidates from SingleFrag ANN models
- Known exact molecular formula (formula-known condition) or precursor m/z only

## Outputs

- Ranked candidate list per test spectrum sorted by cosine similarity
- Top-1 / Top-5 / Top-10 accuracy table per collision-energy condition and ppm tolerance
- Verified benchmark values: 37.6% Top-1, 71.9% Top-5, 82.7% Top-10 (merged CE, 1 ppm)

## How to apply

Compute the cosine similarity between the experimental test spectrum and each candidate's predicted spectrum produced by SingleFrag's per-MZ-position ANN models (ModelVal*.pickle, via spectraPrediction.ipynb). Restrict candidates using a precursor mass filter — the paper evaluates both 1 ppm and 10 ppm tolerances. The paper computes similarity separately for four collision-energy conditions (low: 29.9/59.3/72.6%, medium: 32.2/66.5/79.9%, high: 36.6/69.0/81.2%, and merged spectra across all energies) and finds that merged spectra give the strongest ranking. Sort candidates by cosine score and record where the true compound falls; on the 2,401-molecule test set the paper reports merged-spectrum, formula-known accuracies of 37.6% Top-1, 71.9% Top-5, and 82.7% Top-10. Use Top-1/Top-5/Top-10 hit rates across CE conditions as the decision metric for whether the annotation is reliable.

## Related tools

- **SingleFrag** (Generates the per-molecule in silico predicted spectra used as the ranking reference against the experimental spectrum) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds candidate SMILES into the 300-dimensional vector that is the input to SingleFrag's ANN peak-prediction models)

## Examples

```
cosine_similarity = np.dot(exp_spec, pred_spec) / (np.linalg.norm(exp_spec) * np.linalg.norm(pred_spec)); rank = candidates_by_cosine(cosine_similarity, candidates)
```

## Evaluation signals

- Top-1 accuracy ~37.6%, Top-5 ~71.9%, Top-10 ~82.7% under formula-known, merged-CE condition (published values to compare against)
- Cosine similarity computed on predicted-vs-experimental spectra should be deterministic and reproducible given the same candidate list and CE condition
- Ranking results should be reported for both 1 ppm and 10 ppm precursor mass filters — stricter tolerance should not degrade Top-1 accuracy for the true compound
- Per-CE-condition results (low/medium/high/merged) should show merged spectra achieving the highest Top-1/Top-5/Top-10 rates, as reported in the paper
- Number of test spectra evaluated should be 2,401 per condition

## Limitations

- Ranking accuracy depends on the candidate list quality — exact formula assumption inflates Top-1 hit rate relative to formula-unknown scenarios
- Precursor mass tolerance is a hard filter; candidates outside 1 or 10 ppm are excluded and cannot be recovered by cosine score
- Merged-CE condition requires the empirical library to have spectra at multiple collision energies; this is rarely available outside NIST
- The ~1.9M in silico library has no coverage guarantees for arbitrary molecules — molecules missing from the training distribution may be poorly predicted
- Public libraries (GNPS, MSDial, MassBank) often lack sufficient curation and multi-collision-energy spectra, limiting reuse of this ranking workflow outside NIST

## Evidence

- [other] Ranking accuracy evaluation across CE conditions: [evidence span withheld — anchor status: not_found]

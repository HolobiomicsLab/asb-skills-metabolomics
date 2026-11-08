---
name: collision-energy-condition-comparison
description: Use when evaluating whether in silico predicted spectra can rank the
  true compound from a candidate list filtered by precursor mass, and when comparing
  ranking accuracy across single versus merged collision-energy conditions.
when_to_use_negative:
- When peak intensities must be modeled, since SingleFrag deliberately predicts only
  the presence or absence of each peak, not intensity
- When the goal is direct spectral library matching against curated empirical libraries
  (e.g., NIST MS/MS) rather than in silico candidate ranking
- When the molecular formula or precursor mass of the unknown is unavailable, since
  candidate filtering relies on mass accuracy at 1–10 ppm
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SingleFrag
  role: Generates the in silico predicted MS/MS spectra (per-MZ-position ANN models)
    used as the reference library for cosine-similarity candidate ranking
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds input SMILES into the 300-dimensional vector consumed by the ANN models
    during spectra prediction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:7a4a59de25466c490de2ec401023e4d78a6d20452aa1cb431f2a5e01fd194c24
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# collision-energy-condition-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Benchmark metabolite annotation by ranking candidate compounds via cosine similarity between an unannotated empirical MS/MS spectrum and SingleFrag's in silico predicted spectra, evaluated separately under low, medium, high, and merged collision-energy conditions.

## When to use

Use when evaluating whether in silico predicted spectra can rank the true compound from a candidate list filtered by precursor mass, and when comparing ranking accuracy across single versus merged collision-energy conditions.

## When NOT to use

- When peak intensities must be modeled, since SingleFrag deliberately predicts only the presence or absence of each peak, not intensity
- When the goal is direct spectral library matching against curated empirical libraries (e.g., NIST MS/MS) rather than in silico candidate ranking
- When the molecular formula or precursor mass of the unknown is unavailable, since candidate filtering relies on mass accuracy at 1–10 ppm

## Inputs

- Unannotated empirical MS/MS test spectra (e.g., 2,401 test spectra at multiple collision energies)
- SingleFrag in silico predicted spectral library for ~1.9M molecules (results/predSpectra.json reconstructed from results/dfPredictionPeak_*.pickle)
- Precursor molecular formulas for candidate filtering
- ANN model files (ModelVal*.pickle) and thresholdsANN.pickle from the Figshare SingleFrag dataset

## Outputs

- Ranked candidate lists per test spectrum by cosine similarity score
- Top-1 / Top-5 / Top-10 ranking accuracy table per collision-energy condition (low, medium, high, merged) at 1 ppm and 10 ppm mass tolerances

## How to apply

Load the ~1.9M-molecule in silico spectral library reconstructed from SingleFrag's per-MZ-position ANN models (ModelVal*.pickle, thresholdsANN.pickle) via spectraPrediction.ipynb, where each prediction is a peak presence/absence at 2-decimal MZ resolution rather than an intensity. Assemble candidate lists per test spectrum by filtering on precursor mass at 1 ppm and 10 ppm tolerances. Compute cosine similarity between each empirical test spectrum and all candidate predicted spectra separately under four collision-energy conditions — low, medium, high, and merged (spectra combined across CEs) — then rank candidates by score and report the fraction of test spectra where the true compound lands at Top-1, Top-5, or Top-10. The merged condition is expected to outperform any single-CE condition (e.g., 37.6% Top-1 / 71.9% Top-5 / 82.7% Top-10 for the ANN model), because SingleFrag predicts peak presence rather than intensity and merged spectra capture fragment diversity across collision energies.

## Related tools

- **SingleFrag** (Generates the in silico predicted MS/MS spectra (per-MZ-position ANN models) used as the reference library for cosine-similarity candidate ranking) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds input SMILES into the 300-dimensional vector consumed by the ANN models during spectra prediction)

## Examples

```
from scipy.spatial.distance import cosine; sim = 1 - cosine(exp_peaks, pred_peaks)  # rank candidates per test spectrum per CE condition
```

## Evaluation signals

- Top-1/Top-5/Top-10 accuracies for the merged condition should match the reported 37.6% / 71.9% / 82.7% within numeric agreement
- Merged collision-energy condition should outperform each single-CE condition (low, medium, high) on all three rank metrics
- Ranking accuracy should degrade slightly (or stay comparable) when moving from the 10 ppm to the stricter 1 ppm mass filter, reflecting a smaller candidate pool
- Predicted spectra should contain binary peak-presence calls at 2-decimal MZ positions, reconstructed from per-MZ-position model outputs
- ANN-based rankings should outperform GNN- or COM-based rankings, consistent with the paper's finding that ANN models got better results

## Limitations

- The GNN model shows slightly lower performance than the ANN model, attributed to unoptimized graph features and deliberately simple architectures
- Other spectral libraries (GNPS, MSDial, MassBank) often lack spectra at multiple collision energies, limiting equivalent comparisons outside NIST MS/MS
- Merged-condition gains depend on the availability of empirical spectra at multiple collision energies; single-CE acquisitions cannot exploit the merged condition

## Evidence

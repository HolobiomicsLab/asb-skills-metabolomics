---
name: candidate-spectrum-similarity-ranking
description: Use when you have an unidentified experimental MS/MS spectrum (e.g. an
  ARUS recurrent unidentified spectrum), a candidate molecule list with SMILES and
  assigned formulas, and need to rank candidates by spectral similarity for annotation.
when_to_use_negative:
- A matching empirical spectrum already exists in a curated library (NIST MS/MS) —
  direct library matching is preferable
- The candidate list is empty or no formula assignment (e.g. from BUDDY) is available,
  since ranking depends on enumerated candidate SMILES
- Peak-intensity prediction is required — SingleFrag predicts only presence/absence
  of peaks per m/z position
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: SingleFrag
  role: Deep learning tool that predicts per-peak presence via ANN models to generate
    in silico MS/MS spectra for each candidate molecule
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds candidate SMILES into the 300-dimensional vector used as ANN input
  repo: https://github.com/MaribelPR/SingleFrag
- name: 3DMolMS
  role: State-of-the-art baseline used to benchmark SingleFrag's spectral prediction
    quality before ranking
  repo: https://github.com/JosieHong/3DMolMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:3a8381a61eb1bcd67474e05d2696b482b1f2ebfc81b354ef4c1ce55f9f23520d
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# candidate-spectrum-similarity-ranking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Ranks candidate molecules for an unknown MS/MS spectrum by comparing their in silico predicted spectra (from SingleFrag ANN models) against the experimental spectrum using cosine similarity at 2-decimal m/z resolution, retaining the top candidates.

## When to use

Use when you have an unidentified experimental MS/MS spectrum (e.g. an ARUS recurrent unidentified spectrum), a candidate molecule list with SMILES and assigned formulas, and need to rank candidates by spectral similarity for annotation.

## When NOT to use

- A matching empirical spectrum already exists in a curated library (NIST MS/MS) — direct library matching is preferable
- The candidate list is empty or no formula assignment (e.g. from BUDDY) is available, since ranking depends on enumerated candidate SMILES
- Peak-intensity prediction is required — SingleFrag predicts only presence/absence of peaks per m/z position

## Inputs

- Candidate molecule SMILES (e.g. BUDDY-assigned formulas loaded from inputMols_git.xlsx, 'smiles' column)
- Experimental MS/MS spectrum of the unknown (e.g. ARUS database spectrum)
- SingleFrag Figshare dataset: trained ANN ModelVal*.pickle models, Mol2vec model, AllMostFreqMolGeneral_rep_dades1.pickle, thresholdsANN.pickle

## Outputs

- Per-MZ-position prediction files (results/dfPredictionPeak_*.pickle)
- Reconstructed predicted spectra per candidate (results/predSpectra.json)
- Ranked candidate list (top-10 per spectrum) with cosine similarity scores and Δcos gaps
- Graphical comparison of predicted vs experimental spectra

## How to apply

Embed each candidate SMILES into a 300-dimensional vector with the Mol2vec model (Mol2vecModelGeneral_rep_dades1.model), then run the trained per-MZ-position ANN models (ModelVal*.pickle, one per each of the 1,000 most frequented m/z positions, filename encoding mz*100), calling a peak present when the prediction meets the position-specific threshold in thresholdsANN.pickle. Reconstruct each candidate's predicted spectrum into results/predSpectra.json via spectraPrediction.ipynb (ANN models only, since they outperformed GNN and COM). Compare each predicted spectrum against the experimental spectrum at 2-decimal m/z resolution using cosine similarity, retain the top-10 candidates per spectrum, and look for a significant Δcos gap separating the leading candidate(s) from the rest; candidates with such a gap (e.g. glucuronyl-2-hydroxyhippurate, 8-hydroxyquinoline glucuronide, a truxilline isomer) then undergo expert manual structural and fragmentation elucidation for final confirmation.

## Related tools

- **SingleFrag** (Deep learning tool that predicts per-peak presence via ANN models to generate in silico MS/MS spectra for each candidate molecule) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds candidate SMILES into the 300-dimensional vector used as ANN input) — https://github.com/MaribelPR/SingleFrag
- **3DMolMS** (State-of-the-art baseline used to benchmark SingleFrag's spectral prediction quality before ranking) — https://github.com/JosieHong/3DMolMS

## Examples

```
jupyter nbconvert --to notebook --execute spectraPrediction.ipynb  # builds dfPredictionPeak_*.pickle and results/predSpectra.json, then rank candidates by cosine similarity vs the ARUS spectrum
```

## Evaluation signals

- The true annotation ranks among the top-10 candidates per ARUS spectrum with a significant cosine-similarity (Δcos) gap separating it from remaining candidates
- A graphical comparison of predicted vs experimental spectrum is produced for each confirmed candidate
- Spectral comparison is performed at 2-decimal m/z resolution as the models were trained
- Peak calls are only made at the 1,000 most frequented m/z positions, each using its position-specific threshold from thresholdsANN.pickle
- Confirmed annotations (glucuronyl-2-hydroxyhippurate, 8-hydroxyquinoline glucuronide, truxilline isomer) survive expert manual structural and fragmentation elucidation

## Limitations

- All SingleFrag models outperform CFM-ID and 3DMolMS, but predictions are peak presence/absence only, not intensities
- The GNN model shows slightly lower performance than the ANN model, attributed to deliberately simple, unoptimized architectures; spectraPrediction.ipynb therefore uses ANN models only
- Top-ranked candidates still require expert manual elucidation of molecular structures and fragmentation patterns before being reported as confirmed annotations

## Evidence

- [methods] Compare each predicted spectrum against the experimental ARUS spectrum at 2-decimal m/z resolution and rank the candidate molecules, retaining the top-10 per spectrum.: [evidence span withheld — anchor status: not_found]

---
name: arus-spectrum-annotation
description: Use when you need to annotate previously unidentified recurrent MS/MS
  spectra (e.g. ARUS plasma/urine spectra) by ranking candidate compounds via in silico
  predicted fragment spectra.
when_to_use_negative:
- Input is already a feature table with known compound identities or a spectral library
  match at high confidence — this skill targets unidentified recurrent spectra needing
  de novo candidate ranking.
- The experimental spectra were acquired at only one or a few collision energies (e.g.
  GNPS, MSDial, MassBank-style data) — the article notes such libraries 'often lack
  sufficient curation' and completeness for this workflow's ranking.
- Candidate molecules exceed the 1,000 most-frequented m/z positions in the training
  set or fall outside the m/z coverage of the released ModelVal*.pickle files — predicted
  spectra will be incomplete and ranking unreliable.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SingleFrag
  role: Deep learning tool that predicts peak presence per m/z position using trained
    ANN models, reconstructing full MS/MS spectra from candidate SMILES for comparison
    against experimental ARUS spectra
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds each candidate SMILES into a 300-dimensional vector that serves as
    the input to SingleFrag's ANN models
  repo: https://github.com/MaribelPR/SingleFrag
- name: 3DMolMS
  role: Alternative state-of-the-art in silico spectral predictor used as a performance
    benchmark; SingleFrag outperforms it, so it is not used in the annotation workflow
    itself
  repo: https://github.com/JosieHong/3DMolMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:38215cd9835f06ab9a804d39791c3c7ad1f59a68e698850e7cf1de2637c14e6f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# arus-spectrum-annotation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Annotates recurrent unidentified spectra from the ARUS database (NIST Mass Spectrometry Data Center) by predicting MS/MS spectra for BUDDY-assigned candidate molecules with SingleFrag ANN models and ranking candidates by spectral similarity, confirming annotations by expert manual elucidation.

## When to use

Use when you need to annotate previously unidentified recurrent MS/MS spectra (e.g. ARUS plasma/urine spectra) by ranking candidate compounds via in silico predicted fragment spectra.

## When NOT to use

- Input is already a feature table with known compound identities or a spectral library match at high confidence — this skill targets unidentified recurrent spectra needing de novo candidate ranking.
- The experimental spectra were acquired at only one or a few collision energies (e.g. GNPS, MSDial, MassBank-style data) — the article notes such libraries 'often lack sufficient curation' and completeness for this workflow's ranking.
- Candidate molecules exceed the 1,000 most-frequented m/z positions in the training set or fall outside the m/z coverage of the released ModelVal*.pickle files — predicted spectra will be incomplete and ranking unreliable.

## Inputs

- Recurrent unidentified MS/MS spectra from the ARUS database (plasma/urine, acquired at multiple collision energies)
- Candidate molecule SMILES list (e.g. inputMols_git.xlsx with smiles/name/spectrum/link columns; BUDDY-assigned molecular formulas used to enumerate PubChem candidates)
- SingleFrag dataset from Figshare (trained ANN/GNN/COM ModelVal*.pickle, Mol2vecModelGeneral_rep_dades1.model, AllMostFreqMolGeneral_rep_dades1.pickle, thresholdsANN/GNN/COM.pickle)
- Python 3.8.19 environment with packages in requirements.txt

## Outputs

- results/dfPredictionPeak_*.pickle — per-MZ-position peak-presence predictions for all candidates
- results/predSpectra.json — reconstructed in silico MS/MS spectrum per candidate molecule at 2-decimal m/z resolution
- Ranked top-10 candidate list per ARUS spectrum with cosine-similarity (Δcos) gap as ranking criterion
- Confirmed annotations (e.g. glucuronyl-2-hydroxyhippurate, 8-hydroxyquinoline glucuronide, truxilline isomer) with graphical experimental-vs-predicted spectrum comparisons

## How to apply

Set up a Python 3.8.19 environment with the SingleFrag requirements and download the SingleFrag dataset from Figshare, placing the trained ANN models (ANN/ModelVal*.pickle, filenames encoding m/z as mz*100), the Mol2vec model, and the modelData threshold files under data/. Embed each candidate SMILES (e.g. from a BUDDY-formula-derived PubChem-extended candidate list of 96,492,904 predicted spectra) into a 300-dimensional vector with Mol2vecModelGeneral_rep_dades1.model. For each of the 1,000 most-frequented m/z positions, run the corresponding ModelVal*.pickle ANN and call a peak when the prediction is ≥ the position-specific threshold in thresholdsANN.pickle. Aggregate per-MZ-position predictions into results/dfPredictionPeak_*.pickle, reconstruct the full predicted spectrum into results/predSpectra.json, compare each prediction against the experimental ARUS spectrum at 2-decimal m/z resolution, and retain the top-10 candidates per spectrum; confirmed annotations are those with a significant cosine-similarity (Δcos) gap separating one candidate from the rest. Validate by expert manual elucidation of molecular structures and fragmentation patterns — the paper confirms glucuronyl-2-hydroxyhippurate, 8-hydroxyquinoline glucuronide, and a truxilline isomer this way.

## Related tools

- **SingleFrag** (Deep learning tool that predicts peak presence per m/z position using trained ANN models, reconstructing full MS/MS spectra from candidate SMILES for comparison against experimental ARUS spectra) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds each candidate SMILES into a 300-dimensional vector that serves as the input to SingleFrag's ANN models) — https://github.com/MaribelPR/SingleFrag
- **3DMolMS** (Alternative state-of-the-art in silico spectral predictor used as a performance benchmark; SingleFrag outperforms it, so it is not used in the annotation workflow itself) — https://github.com/JosieHong/3DMolMS

## Examples

```
jupyter nbconvert --to notebook --execute spectraPrediction.ipynb  # run after placing ANN/ModelVal*.pickle, thresholdsANN.pickle, and inputMols_git.xlsx under data/ per the README
```

## Evaluation signals

- Each of the 1,000 most-frequented m/z positions has a peak called only when the ANN prediction is ≥ its position-specific threshold in thresholdsANN.pickle
- The top-ranked candidate per ARUS spectrum shows a significant cosine-similarity (Δcos) gap over the remaining candidates, not a marginal similarity advantage
- Confirmed annotations (glucuronyl-2-hydroxyhippurate, 8-hydroxyquinoline glucuronide, truxilline isomer) rank among the top-10 candidates and survive expert manual elucidation of structure and fragmentation
- A graphical comparison of each predicted spectrum against its experimental counterpart shows peak placement agreement at 2-decimal m/z resolution
- results/predSpectra.json contains a fully reconstructed spectrum per candidate, assembled from the per-MZ-position pickle files

## Limitations

- The GNN model shows slightly lower performance than the ANN model; its underperformance reflects unoptimized graph features and layer architectures rather than a fundamental limitation, so the released workflow uses ANN models only.
- Neural network architectures were deliberately kept simple and are not optimized — performance could improve with more expressive models.
- Public spectral libraries such as GNPS, MSDial, or MassBank often lack spectra at multiple collision energies and sufficient curation, producing incomplete or less informative reference spectra compared with NIST MS/MS.
- Annotation reliability remains 'the standing roadblock for the development of the full potential of metabolomics', so confirmed annotations require expert manual elucidation beyond automated ranking.

## Evidence

- [methods] Load candidate molecules from input file: "Inside the *data.zip* folder, we already find a toy example of an input file. This contains 4 columns: smiles, name, spectrum, link"
- [readme] Peak presence threshold logic: "thresholdsANN.pickle: threshold per each of the most frequented 1,000 MZ positions in the training set. If a prediction using an ANN model for a specific position is higher or equal to this value"
- [readme] Two-stage prediction output files: "we first create a file with all the predictions per MZ position (*results/dfPredictionPeak_***.pickle*), and then we reconstruct each spectra considering all the predictions from the different MZ"
- [results] Confirmed annotation outputs: [evidence span withheld — anchor status: not_found]
- [discussion] Library-curation rationale for choosing NIST MS/MS: "other spectral libraries such as GNPS, MSDial, or MassBank often lack sufficient curation and typically contain spectra acquired at only one or a few collision energies per compound."

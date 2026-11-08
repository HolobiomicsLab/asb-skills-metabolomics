---
name: mass-spectral-library-matching
description: Use when the goal is to identify an unknown small molecule from its EI
  mass spectrum by ranking candidates in a spectral reference library (e.g., NIST
  2017) and reporting recall@10.
when_to_use_negative:
- The molecule of interest is absent from every external reference library and no
  predicted spectra are available — without augmentation, correct identification is
  impossible (the paper's stated coverage problem).
- The spectra come from an ionization method other than electron ionization and the
  model/library was not retrained for that method (NEIMS was trained only on EI spectra
  from NIST 2017).
- Isotope intensity peaks in ion fragments matter for the match — NEIMS currently
  does not model isotope peaks, so augmented-library matching can be degraded for
  such cases.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NEIMS
  role: Bidirectional MLP that predicts m/z-intensity spectra for candidate molecules
    in ~5 ms each, enabling construction of an augmented reference library for matching
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Generates additive ECFP fingerprints (length 4096, radius 2) from molecular
    structures to feed the spectrum predictor
- name: Tensorflow
  role: Runs the pretrained NEIMS bidirectional MLP used to produce predicted spectra
    for the augmented library
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:720c42b2a46b2cdca27545679749ddb3b193ef1a1b349de48f36af2a700a3a33
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# mass-spectral-library-matching

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Identifies small molecules by comparing an observed query electron-ionization mass spectrum against a (possibly NEIMS-augmented) reference library using weighted cosine similarity with a molecular-mass filter. This is the evaluation task that the NEIMS paper uses to quantify how model-predicted spectra extend library coverage.

## When to use

Use when the goal is to identify an unknown small molecule from its EI mass spectrum by ranking candidates in a spectral reference library (e.g., NIST 2017) and reporting recall@10.

## When NOT to use

- The molecule of interest is absent from every external reference library and no predicted spectra are available — without augmentation, correct identification is impossible (the paper's stated coverage problem).
- The spectra come from an ionization method other than electron ionization and the model/library was not retrained for that method (NEIMS was trained only on EI spectra from NIST 2017).
- Isotope intensity peaks in ion fragments matter for the match — NEIMS currently does not model isotope peaks, so augmented-library matching can be degraded for such cases.

## Inputs

- Query spectra from the NIST 2017 replicates library (noisier spectra of molecules present in the NIST main library)
- NIST 2017 Mass Spectral Main Library spectra (240,942 spectra) as the reference library
- Optional: model-predicted spectra (e.g., NEIMS bidirectional MLP output as m/z-intensity vectors) for molecules absent from the library, to build the augmented reference library
- Molecular mass of each query molecule for the mass filter

## Outputs

- Ranked list of candidate spectra per query with weighted cosine similarity scores
- Rank of the correct spectrum for every query molecule
- recall@10 (fraction of queries with rank ≤ 10), e.g., 91.8% with the 5 Da mass filter
- Per-query filtered library sizes (median 6696 candidates at 5 Da tolerance)

## How to apply

Treat each query spectrum as a vector over integral m/z bins and compute the weighted cosine similarity (eq 1) between the query and every spectrum in the reference library. Optionally restrict the candidate set with the mass filter: keep only spectra whose molecular mass differs by no more than 5 Da from the query molecule's mass, which cuts the NIST main library to a median of 6696 spectra per query and adds roughly 6 percentage points of recall. To test an augmented library, first remove the query-set molecules' observed spectra from the NIST main library and replace them with model-predicted spectra (e.g., NEIMS bidirectional forward/reverse predictions), so the correct answer is only reachable via the prediction. Record the rank of the true spectrum for each query and report recall@10 (rank ≤ 10) as the accuracy metric. On the NIST 2017 replicates test split with the 5 Da mass filter, the expected performance for a NEIMS-augmented library is about 91.7–91.8% recall@10.

## Related tools

- **NEIMS** (Bidirectional MLP that predicts m/z-intensity spectra for candidate molecules in ~5 ms each, enabling construction of an augmented reference library for matching) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Generates additive ECFP fingerprints (length 4096, radius 2) from molecular structures to feed the spectrum predictor)
- **Tensorflow** (Runs the pretrained NEIMS bidirectional MLP used to produce predicted spectra for the augmented library)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- recall@10 on the held-out test half of the NIST 2017 replicates library reaches ~91.7–91.8% with the 5 Da mass filter and the NEIMS-augmented library
- Mass filter at 5 Da tolerance reduces the candidate library to a median of 6696 spectra per query molecule
- Augmented-library recall@10 must exceed the observed-spectra-only baseline (query molecules' spectra replaced by predictions)
- Each query spectrum is matched against every library spectrum via weighted cosine similarity per eq 1, and the correct hit's rank is recorded for 100% of queries
- Predicted spectra are produced at roughly 5 ms per molecule, so timing can be checked as a sanity signal

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments, which can limit match accuracy for isotope-heavy compounds.
- Library matching cannot identify a molecule whose spectrum is not in the library; the approach depends on augmentation with predicted spectra to fix this coverage gap.
- The 5 Da mass filter requires an accurate molecular mass of the sample, which is only possible in some experimental setups.
- No version numbers for the software are documented, so reproducing the exact recall@10 figure may require checking the training_splits directory in the archived repository.

## Evidence

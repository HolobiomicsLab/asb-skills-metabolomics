---
name: weighted-cosine-similarity-scoring
description: Use when matching an observed electron-ionization mass spectrum against
  a reference/augmented library to identify a molecule and you need ranked candidates
  with recall@k metrics.
when_to_use_negative:
- The sample molecule has no prior mass estimate and no mass filter can be applied
  — the full-library search loses the ~6-point recall gain from mass filtering
- The spectra are from an ionization method other than electron ionization and the
  library/predictions were not trained for that method
- The query molecule's true spectrum is in the library unaltered — this skill is designed
  to evaluate identification when library entries are replaced by NEIMS predictions
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: NEIMS
  role: Supplies predicted spectra used to construct the augmented reference library
    and the eq 1 similarity score used for library matching
  repo: github.com/brain-research/deep-molecular-massspec
- name: RDKit Cheminformatics package
  role: Generates the additive ECFP fingerprints (length 4096, radius 2) from SMILES/InChI
    to obtain molecular mass and molecular identity for the filter step
- name: Tensorflow
  role: Framework in which the trained NEIMS model produces the spectra that populate
    the augmented library being scored
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:950023f2179b047a3c8b136929d6ed0c028b61c7129d6cdcec68c379c21c8f6b
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# weighted-cosine-similarity-scoring

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Scores query mass spectra against a spectral reference library using the NEIMS weighted cosine similarity (eq 1), optionally restricting the library to candidate molecules within a mass tolerance, to identify unknown molecules by rank of the correct spectrum.

## When to use

Use when matching an observed electron-ionization mass spectrum against a reference/augmented library to identify a molecule and you need ranked candidates with recall@k metrics.

## When NOT to use

- The sample molecule has no prior mass estimate and no mass filter can be applied — the full-library search loses the ~6-point recall gain from mass filtering
- The spectra are from an ionization method other than electron ionization and the library/predictions were not trained for that method
- The query molecule's true spectrum is in the library unaltered — this skill is designed to evaluate identification when library entries are replaced by NEIMS predictions

## Inputs

- Query spectrum as an intensity vector over integral m/z bins (observed EI mass spectrum)
- Reference or augmented spectral library (e.g., NIST 2017 Mass Spectral Main Library with NEIMS-predicted spectra substituted for query molecules)
- Molecular mass of each query molecule (for the 5 Da mass filter)

## Outputs

- Ranked list of library spectra per query with weighted cosine similarity scores
- Rank of the correct spectrum for each query
- recall@10 (rank ≤ 10) summary metric over the held-out test half

## How to apply

Build the reference library (e.g., the NIST 2017 main library augmented by replacing query-set spectra with NEIMS predictions) and represent every spectrum as a vector of peak intensities over m/z bins. For each query spectrum, compute the weighted cosine similarity of eq 1 between the query vector and every library spectrum; optionally pre-filter the library with the mass filter set to a tolerance of 5 Da of the query molecule's molecular mass, which reduces the library to a median of 6696 spectra per query. Rank library entries by similarity score and record the rank of the correct molecule's spectrum; compute recall@10 (rank ≤ 10). Rationale: mass filtering exploits prior knowledge of molecular mass to shrink the search space and contributed an additional ~6 percentage points of recall in the article's evaluation.

## Related tools

- **NEIMS** (Supplies predicted spectra used to construct the augmented reference library and the eq 1 similarity score used for library matching) — github.com/brain-research/deep-molecular-massspec
- **RDKit Cheminformatics package** (Generates the additive ECFP fingerprints (length 4096, radius 2) from SMILES/InChI to obtain molecular mass and molecular identity for the filter step)
- **Tensorflow** (Framework in which the trained NEIMS model produces the spectra that populate the augmented library being scored)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Recall@10 on the held-out test half of the NIST 2017 replicates library should be near the reported 91.8% (with the 5 Da mass filter) on the augmented library
- The 5 Da mass filter should reduce the candidate library to a median of approximately 6696 spectra per query molecule
- Applying the mass filter should improve recall by roughly 6 percentage points relative to unfiltered full-library matching
- The rank of the correct query molecule's spectrum should be recorded for every query with no missing/unmatched entries

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments, which can affect similarity scoring of isotope-containing candidates
- Mass filtering requires that the molecular mass of the sample be known with some accuracy; without it, matching accuracy on the augmented library is lower
- Results are benchmarked on NIST 2017 libraries; performance on other libraries (e.g., NIST 14) or ionization methods may differ

## Evidence

- [readme] custom library matching task in README: [evidence span withheld — anchor status: external_doc]

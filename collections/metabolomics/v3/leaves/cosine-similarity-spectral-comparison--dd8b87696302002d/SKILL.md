---
name: cosine-similarity-spectral-comparison
description: Use when matching an observed EI mass spectrum against a library of candidate
  spectra and you need ranked similarity scores (e.g. recall@10 evaluation) under
  a molecular-mass filter.
when_to_use_negative:
- Query molecules are not yet converted to spectra (e.g. you only have SMILES/InChI)
  — spectral prediction, not similarity ranking, is needed first.
- Candidate library spectra have not been generated and mass-filtered, so similarity
  would be computed against an unfiltered full library, invalidating the NEIMS evaluation
  protocol.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NEIMS
  role: Generates the predicted spectra inserted into the augmented reference library
    that similarity is computed against
  repo: github.com/brain-research/deep-molecular-massspec
- name: Tensorflow
  role: Constructs and trains the MLP model producing the spectra used in library
    matching
- name: RDKit Cheminformatics package
  role: Generates the ECFP molecular fingerprints used to represent molecules for
    spectral prediction
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:0d7d539de58d168aecad0fe75eac9ba3d32ea09f1d14b22d3de7df2fe53c4c04
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cosine-similarity-spectral-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Rank mass-spectral candidates by computing the weighted cosine similarity (eq 1) between each query spectrum and every spectrum in an augmented reference library, then report recall@10. In NEIMS, library matching restricts candidates to spectra whose molecular mass is within 5 Da of the query before scoring.

## When to use

Use when matching an observed EI mass spectrum against a library of candidate spectra and you need ranked similarity scores (e.g. recall@10 evaluation) under a molecular-mass filter.

## When NOT to use

- Query molecules are not yet converted to spectra (e.g. you only have SMILES/InChI) — spectral prediction, not similarity ranking, is needed first.
- Candidate library spectra have not been generated and mass-filtered, so similarity would be computed against an unfiltered full library, invalidating the NEIMS evaluation protocol.

## Inputs

- NIST 2017 replicates library query spectra
- augmented reference library (NIST main library spectra + NEIMS-predicted spectra for query-set molecules)
- per-query molecular mass for mass filtering
- optionally per-query molecular formula (CFM-EI-style formula filter)

## Outputs

- similarity score matrix (query × filtered candidate spectra)
- rank of the correct spectrum for each query
- recall@10 statistic (percentage of query set with correct spectrum ranked ≤10)

## How to apply

Load the augmented reference library (NIST main library with NEIMS predictions substituted for query-set molecules) together with the query spectra (e.g. NIST 2017 replicates). For each query, apply the mass filter with a 5 Da tolerance of the query molecule's mass, which reduces the library to a median of 6696 spectra per query. Compute the weighted cosine similarity (eq 1) between the query spectrum and every remaining candidate spectrum, and record the rank of the correct spectrum. Aggregate ranks into recall@10 — the percentage of queries whose correct spectrum ranked ≤10; NEIMS reports 91.8% on this protocol. Note CFM-EI uses the molecular formula to filter the search library, an alternative filtering scheme.

## Related tools

- **NEIMS** (Generates the predicted spectra inserted into the augmented reference library that similarity is computed against) — github.com/brain-research/deep-molecular-massspec
- **Tensorflow** (Constructs and trains the MLP model producing the spectra used in library matching)
- **RDKit Cheminformatics package** (Generates the ECFP molecular fingerprints used to represent molecules for spectral prediction)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Correct-spectrum rank ≤10 for a majority of queries; paper reports recall-at-10 of 91.8%
- 5 Da mass filter reduces the library to a median of 6696 spectra per query molecule
- Weighted cosine similarity (eq 1) values lie in [0,1] across query–candidate pairs
- Ranks and recall statistics saved to mass_filter_results.csv
- Recall@10 with the mass filter should exceed the unfiltered augmented-library baseline

## Limitations

- NEIMS lacks a method to model intensity peaks for isotopes in ion fragments
- Weighted cosine similarity is computed over integral m/z bins; unresolved peaks may distort scores
- If the query's molecular mass is unknown or inaccurate, the 5 Da mass filter can exclude the true spectrum

## Evidence

- [methods] 5 Da mass filter tolerance: "During the library match search, we have a mass filtering option. This feature reduces the library size so it only includes spectra from molecular candidates that have a molecular mass that differ by"
- [results] median 6696 spectra after mass filter: "The mass filter was set to a tolerance of 5 Da of the query molecule’s mass; this reduces the size of the library to a median of 6696 spectra for each query molecule"

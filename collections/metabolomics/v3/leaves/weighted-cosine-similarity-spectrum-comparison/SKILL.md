---
name: weighted-cosine-similarity-spectrum-comparison
description: Use when a query EI mass spectrum must be identified by ranking it against
  a spectral reference library, especially when the library has been augmented with
  predicted spectra (e.g. NEIMS outputs) and recall@k must be measured.
when_to_use_negative:
- The sample molecule is already known to be absent from any library and no synthetic
  spectra have been added — library matching cannot identify an uncovered molecule
- The molecular formula is used as a hard pre-filter (as in the CFM-EI model) rather
  than the 5 Da mass-tolerance filter described here
- Spectra come from a non-EI ionization method for which the NEIMS model has not been
  retrained
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: NEIMS
  role: Generates the predicted spectra used to augment the reference library before
    weighted cosine similarity matching
  repo: github.com/brain-research/deep-molecular-massspec
- name: Tensorflow
  role: Constructs and trains the NEIMS model whose predictions populate the augmented
    library used in matching
- name: RDKit Cheminformatics package
  role: Generates the additive ECFP fingerprints (length 4096, radius 2) that are
    the model input producing predicted spectra for matching
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acscentsci.9b00085
    title: Rapid Prediction of Electron–Ionization Mass Spectrometry Using Neural
      Networks
  source_content_hash: sha256:60e151662562c5f06365f2f0489b7c139ed509beba49bd89df2b800e2a3a7c6a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Adam optimizer
  - NEIMS
  - RDKit Cheminformatics package
  - Tensorflow
  - Vizier
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# weighted-cosine-similarity-spectrum-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Ranks candidate molecules for a query spectrum by computing a weighted cosine similarity between the observed/query spectrum and every spectrum in an augmented (observed + NEIMS-predicted) reference library, with an optional molecular-mass filter to prune candidates before scoring.

## When to use

Use when a query EI mass spectrum must be identified by ranking it against a spectral reference library, especially when the library has been augmented with predicted spectra (e.g. NEIMS outputs) and recall@k must be measured.

## When NOT to use

- The sample molecule is already known to be absent from any library and no synthetic spectra have been added — library matching cannot identify an uncovered molecule
- The molecular formula is used as a hard pre-filter (as in the CFM-EI model) rather than the 5 Da mass-tolerance filter described here
- Spectra come from a non-EI ionization method for which the NEIMS model has not been retrained

## Inputs

- Query mass spectra from the NIST 2017 replicates library (held-out molecules)
- NIST main library spectra (with query-molecule spectra removed and replaced by NEIMS predictions to form the augmented library)
- Predicted spectra from NEIMS model variants (forward, reverse, or bidirectional prediction)
- Molecular masses of query molecules for the 5 Da mass filter

## Outputs

- Rank of the correct candidate spectrum for every query under each prediction mode
- recall@10 (fraction of queries whose correct spectrum ranks at or below rank 10) per prediction mode
- Comparison table of matching accuracy across modes (e.g. results_table.csv)

## How to apply

Preprocess the query spectra (e.g. the held-out half of the NIST 2017 replicates library) and build the reference library by removing query-molecule spectra from the NIST main library and replacing them with model-predicted spectra. Optionally apply the mass filter, keeping only candidates whose molecular mass differs by at most 5 Da from the query molecule's mass, which reduces the library to a median of 6696 spectra per query. Score every query against each remaining candidate with the weighted cosine similarity metric, then record the rank of the correct candidate. Evaluate by recall@10 (fraction of queries whose correct spectrum ranks at or above rank 10); the full bidirectional NEIMS reaches 91.8% recall@10 under this protocol.

## Related tools

- **NEIMS** (Generates the predicted spectra used to augment the reference library before weighted cosine similarity matching) — github.com/brain-research/deep-molecular-massspec
- **Tensorflow** (Constructs and trains the NEIMS model whose predictions populate the augmented library used in matching)
- **RDKit Cheminformatics package** (Generates the additive ECFP fingerprints (length 4096, radius 2) that are the model input producing predicted spectra for matching)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- Correct spectrum of each query should rank at or below rank 10 for 91.8% of queries under the full bidirectional NEIMS (recall@10 = 91.8%)
- Mass filter with 5 Da tolerance should reduce the library to a median of 6696 spectra per query molecule
- Bidirectional prediction should yield higher recall@10 than forward-only and reverse-only modes (reported ~30% improvement over linear-regression and MLP baselines)
- Each query must have a recorded rank; no query should be scored against its own observed spectrum in the library
- Bidirectional model should achieve the highest recall@10 of all tested modes before saving the results table

## Limitations

- NEIMS does not model intensity peaks corresponding to isotopes in ion fragments, requiring a small shift handling for isotope offsets in reverse prediction
- The mass filter can only be applied when the molecular mass of the sample is known with some accuracy
- Matching accuracy depends on the augmented library: query-molecule spectra must be removed from the NIST main library before replacing them with predictions, or results are invalid

## Evidence

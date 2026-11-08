---
name: weighted-cosine-similarity-scoring
description: Use when you have paired predicted and experimental EI mass spectra (e.g.,
  NEIMS predictions vs. NIST 2017 measured spectra) and need to quantify prediction
  quality or benchmark a spectral predictor.
when_to_use_negative:
- When the goal is fast library retrieval/identification at million-scale rather than
  prediction-quality scoring — FastEI (Word2vec embedding + HNSW) is designed for
  that and is two orders of magnitude faster than WCS
- When spectra are not paired predicted-vs-experimental vectors (e.g., only query
  spectra without a reference), since WCS requires two comparable spectra vectors
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NEIMS
  role: Supplies the predicted EI spectra (from ECFP inputs to a fully connected network)
    that are scored against experimental spectra via WCS; weights downloaded without
    re-training
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: RDKit
  role: Reads the SMILES strings of test molecules and generates the ECFP fingerprints
    used as NEIMS input
  repo: https://www.rdkit.org/docs/Install.html
- name: faiss.IndexFlatL2
  role: Related exact-similarity computation used in the FastEI comparison context,
    calculating Euclidean distance between two normalized spectrum vectors
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:065aa415d68b887f6bc8b0f9c2be4c025e64e750451581b06eb8434226bc2e6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/brain-research/deep-molecular-massspec
  tools:
  - FastEI
  - NEIMS
  - RDKit
  - faiss.IndexFlatL2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# weighted-cosine-similarity-scoring

## Summary

Compute weighted cosine similarity (WCS) between a predicted EI mass spectrum (e.g., from NEIMS) and an experimental spectrum to quantify spectral prediction quality, e.g., thresholding the test-set WCS distribution at 0.7.

## When to use

Use when you have paired predicted and experimental EI mass spectra (e.g., NEIMS predictions vs. NIST 2017 measured spectra) and need to quantify prediction quality or benchmark a spectral predictor.

## When NOT to use

- When the goal is fast library retrieval/identification at million-scale rather than prediction-quality scoring — FastEI (Word2vec embedding + HNSW) is designed for that and is two orders of magnitude faster than WCS
- When spectra are not paired predicted-vs-experimental vectors (e.g., only query spectra without a reference), since WCS requires two comparable spectra vectors

## Inputs

- SMILES strings of NIST 2017 test-split molecules (11,499 test molecules)
- Experimental EI mass spectra from the NIST 2017 library
- Pre-trained NEIMS weights (massspec_weights.zip, used without re-training)

## Outputs

- Per-molecule weighted cosine similarity scores (wcs_scores.csv)
- Fraction of test molecules with predicted-vs-experimental WCS > 0.7 (expected ~78%)
- Predicted EI mass spectrum for every test molecule

## How to apply

Obtain the NIST 2017 test-split molecules (SMILES and measured spectra) and the pre-trained NEIMS weights from the official deep-molecular-massspec repository without re-training. Read the SMILES strings of the 11,499 test molecules with RDKit (v2022.03.3), generate their ECFP fingerprints, and feed the ECFPs into the NEIMS fully connected network to obtain one predicted EI spectrum per molecule. For each molecule, compute the weighted cosine similarity between the predicted and experimental spectra (the weighted variant emphasizes high-mass/intensity-relevant peaks relative to plain cosine similarity) and write all per-molecule scores to a file such as wcs_scores.csv. Finally, threshold the WCS distribution at 0.7 and compute the fraction of molecules above threshold; the reference result is ~78% of test molecules with WCS > 0.7, matching Supplementary Fig. 2 of the NEIMS work, and this WCS quantity is the baseline against which FastEI's embedding-based matching (which computes Euclidean distance between normalized vectors via faiss.IndexFlatL2) was compared.

## Related tools

- **NEIMS** (Supplies the predicted EI spectra (from ECFP inputs to a fully connected network) that are scored against experimental spectra via WCS; weights downloaded without re-training) — https://github.com/brain-research/deep-molecular-massspec
- **RDKit** (Reads the SMILES strings of test molecules and generates the ECFP fingerprints used as NEIMS input) — https://www.rdkit.org/docs/Install.html
- **faiss.IndexFlatL2** (Related exact-similarity computation used in the FastEI comparison context, calculating Euclidean distance between two normalized spectrum vectors)

## Examples

```
python make_spectra_prediction.py --input_file=examples/pentachlorobenzene.sdf --output_file=/tmp/annotated.sdf --weights_dir=$MODEL_WEIGHTS_DIR/massspec_weights
```

## Evaluation signals

- The fraction of test molecules with predicted-vs-experimental WCS > 0.7 should reproduce the reported ~78% shown in Supplementary Fig. 2
- All 11,499 test molecules should yield exactly one WCS score in wcs_scores.csv with no missing values
- WCS values must fall in a valid cosine-similarity range (roughly [-1, 1]); negative or out-of-range values indicate a spectra-vectorization bug
- The NEIMS weights must be used as downloaded without re-training, matching the official storage.googleapis.com/massspec_weights.zip checkpoint

## Limitations

- Exact WCS matching is slow: it is the baseline against which FastEI achieved a speedup of two orders of magnitude, making it impractical for million-scale library search
- The NEIMS pre-trained weight version/build date is not stated in the article, which can affect reproducibility
- WCS is referenced for comparison in the FastEI paper but not formally defined there, so the exact peak-weighting scheme must be sourced from the original NEIMS paper (DOI: 10.1021/acscentsci.9b00085)

## Evidence

- [results] WCS threshold finding: [evidence span withheld — anchor status: not_found]
- [discussion] NEIMS weights provenance: "The NEIMS model was downloaded from its official repository (https://storage.googleapis.com/deep-molecular-massspec/massspec_weights/massspec_weights.zip) without re-training"
- [discussion] RDKit fingerprint generation: "The SMILES strings of molecules were read by RDKit (v2022.03.3)"
- [discussion] Vector distance computation baseline: "They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL2"
- [readme] NEIMS prediction from README: [evidence span withheld — anchor status: external_doc]

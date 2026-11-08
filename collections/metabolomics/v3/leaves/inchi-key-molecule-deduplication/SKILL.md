---
name: inchi-key-molecule-deduplication
description: 'Use when preparing query and reference libraries for spectral database
  lookup: overlapping molecules between sides would inflate recall, so InChI-key-based
  disjointness must be asserted before ranking.'
when_to_use_negative:
- When query and reference libraries are intentionally the same set (e.g. self-match
  sanity checks) — the strict non-overlap assertion would fail.
- When the candidate list has no InChI strings available to generate inchi_key hashes,
  since dedup is keyed on the InChI-derived hash.
- When deduplicating within a single library for training data cleaning, where cross-side
  overlap is not the concern being enforced.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: rassp-public
  role: Provides library_match_pipeline.py, whose inchi_key non-overlap assertion
    enforces the deduplication split between query and main libraries
  repo: https://github.com/thejonaslab/rassp-public
- name: RDKit
  role: Generates inchi_key from the inchi string via Chem.InchiToInchiKey, the key
    used for dedup and disjointness checking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c02093
    title: Rapid Approximate Subset-Based Spectra Prediction for Electron Ionization–Mass
      Spectrometry
  source_content_hash: sha256:5456a71cf4fbb78140f448450565d039bcf3e71d7170232000d5b954998b213f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thejonaslab/rassp-public
  tools:
  - FormulaNet
  - NEIMS
  - RDKit
  - SubsetNet
  - rassp-public
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# InChI-Key Molecule Deduplication

## Summary

Deduplicate candidate spectra/molecule lists by generating canonical InChI keys with RDKit (Chem.InchiToInchiKey) and partitioning the 100-molecule smallmols sample into two strictly non-overlapping 50-molecule query and main libraries, enforcing the disjointness with an assertion before library matching.

## When to use

Use when preparing query and reference libraries for spectral database lookup: overlapping molecules between sides would inflate recall, so InChI-key-based disjointness must be asserted before ranking.

## When NOT to use

- When query and reference libraries are intentionally the same set (e.g. self-match sanity checks) — the strict non-overlap assertion would fail.
- When the candidate list has no InChI strings available to generate inchi_key hashes, since dedup is keyed on the InChI-derived hash.
- When deduplicating within a single library for training data cleaning, where cross-side overlap is not the concern being enforced.

## Inputs

- Molecules with InChI strings (from parquet columns inchi/inchi_key, e.g. smallmols_cfm_pred_public_sample.0.parquet and .1.parquet)
- Predicted or experimental spectra paired to molecules (List[Tuple[Float, Float]] (mass, intensity) pairs)
- RDKit Chem module (Chem.InchiToInchiKey, Chem.Mol for deserializing rdmol binary blobs)

## Outputs

- Two strictly non-overlapping molecule/spectra parquet files (query library and main library, disjoint by inchi_key)
- Validated input for library_match_pipeline.py that passes its inchi_key non-overlap assertion

## How to apply

Generate the inchi_key column from each molecule's InChI string using RDKit's Chem.InchiToInchiKey, as documented for the rassp sample_data parquet files. Split the molecule set into two disjoint halves and designate one as the query library and the other as the main library; in the provided sample the 100-molecule smallmols_cfm_pred_public_sample set is saved as two non-overlapping 50-molecule parquet files (.0 and .1), which are meant to be used together in the library matching pipeline. Before running the match, assert that query and main library inchi_key sets share no molecules — the pipeline's own assert checks this strict non-overlap. Only after dedup passes should weighted Stein dot-product ranking and top-10 recall/error-rate metrics be computed, since any key overlap would leak identical structures across sides and inflate apparent match performance.

## Related tools

- **rassp-public** (Provides library_match_pipeline.py, whose inchi_key non-overlap assertion enforces the deduplication split between query and main libraries) — https://github.com/thejonaslab/rassp-public
- **RDKit** (Generates inchi_key from the inchi string via Chem.InchiToInchiKey, the key used for dedup and disjointness checking)

## Examples

```
from rdkit import Chem; key = Chem.InchiToInchiKey(inchi); assert not (set(query.inchi_key) & set(main.inchi_key))
```

## Evaluation signals

- Set intersection of query_library inchi_key values and main_library inchi_key values is empty (the pipeline's assert passes).
- Query and main library parquet files correspond to the 50-molecule halves of the 100-molecule smallmols sample (50 rows each, disjoint).
- Every inchi_key is a valid 27-character InChI key derived from its inchi string via Chem.InchiToInchiKey.
- Running library_match_pipeline.py writes library_match_results.metrics.mass_filter_<MASS_FILTER_DELTA>.<DP_NAME> without raising the non-overlap assertion.

## Limitations

- The library matching pipeline assumes both inchi_key columns exist; missing keys make the disjointness check impossible.
- inchi_key is a hash of the inchi string, so molecules lacking an InChI string cannot be keyed this way.
- Deduplication by inchi_key groups by structure identity, not by stereoisomer/tautomer variants that InChI keys may not distinguish.

## Evidence

- [readme] inchi_key generated via Chem.InchiToInchiKey: "- `inchi_key`: String
  - Hash of the `inchi` string.
  - Generated from `inchi` via `Chem.InchiToInchiKey(inchi)`."
- [readme] two non-overlapping 50-molecule datasets: [evidence span withheld — anchor status: not_found]
- [readme] strict non-overlap assertion in pipeline: "The main library and query library molecules must be strictly non-overlapping. There is an assert to check for this."
- [readme] assumes inchi_key column exists: "this pipeline assumes the existence of the `inchi_key` column in the `main_library` and `query_library` Parquet files."

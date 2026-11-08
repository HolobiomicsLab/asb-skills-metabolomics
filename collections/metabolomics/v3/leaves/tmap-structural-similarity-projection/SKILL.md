---
name: tmap-structural-similarity-projection
description: Use when you must quantify how many compounds in a spectral library are
  new relative to open (GNPS, MassBank EU, MoNA) and commercial (mzCloud, NIST23)
  libraries, and visualize the overlap pattern as an UpSet plot and TMAP projection.
when_to_use_negative:
- You need full stereochemical identity counts — first-InChIKey-block (14-character)
  truncation deliberately conflates stereoisomers, which is not the right granularity
  for stereo-specific questions.
- Your goal is spectral (peak-level) library matching rather than compound-list coverage
  comparison.
- The reference library structures lack InChIKeys or resolvable identifiers, so first-block
  truncation and set-difference cannot be applied consistently.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: corinnabrungs/msn_tree_library
  role: Source of the MSn Lib InChIKey compound list and the metadata/PubChem structure
    resolution used to build the comparison sets
  repo: corinnabrungs/msn_tree_library
- name: MassBank/MassBank-data
  role: Reference library providing MassBank EU InChIKey records for the set-difference
    comparison
  repo: MassBank/MassBank-data
- name: upsetplot
  role: Python package used to build the UpSet plot of the six-library first-InChIKey
    block intersection matrix
- name: tmap
  role: Python package used to construct the TMAP layout over first-InChIKey block
    classes for the chemical-space projection
- name: myopic-mces
  role: Optional computation of myopic MCES structural distances for structural-similarity
    based projections
  repo: AlBi-HHU/myopic-mces
- name: pip
  role: Installs the requirements for running the pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:3d2e313cf3a3c45c956e6e73abdfc9933c6376c027eaea2c51cd921b5989d633
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/corinnabrungs/msn_tree_library
  tools:
  - MassBank/MassBank-data
  - corinnabrungs/msn_tree_library
  - myopic-mces
  - pip
  - tmap
  - upsetplot
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# tmap-structural-similarity-projection

## Summary

Compares compound coverage of the MSn Lib spectral library against reference libraries (GNPS, MassBank EU, MoNA, mzCloud, NIST23) by truncating InChIKeys to their first 14-character block, computing set-difference overlap counts, and projecting the shared/unique chemical space as an UpSet intersection matrix and a TMAP layout over first-InChIKey-block classes.

## When to use

Use when you must quantify how many compounds in a spectral library are new relative to open (GNPS, MassBank EU, MoNA) and commercial (mzCloud, NIST23) libraries, and visualize the overlap pattern as an UpSet plot and TMAP projection.

## When NOT to use

- You need full stereochemical identity counts — first-InChIKey-block (14-character) truncation deliberately conflates stereoisomers, which is not the right granularity for stereo-specific questions.
- Your goal is spectral (peak-level) library matching rather than compound-list coverage comparison.
- The reference library structures lack InChIKeys or resolvable identifiers, so first-block truncation and set-difference cannot be applied consistently.

## Inputs

- MSn Lib InChIKey list from corinnabrungs/msn_tree_library
- InChIKey sets from GNPS (ALL_GNPS_NO_PROPAGATED), MassBank EU, MoNA, mzCloud, and NIST23
- SMILES/InChI structures for similarity computation (queried from PubChem by Name search when absent)

## Outputs

- Overlap counts table (per-library and union set-difference counts)
- UpSet plot of the six-library first-InChIKey block intersection matrix (reproduces Extended Data Fig. 4)
- TMAP layout over first-InChIKey block classes (reproduces Figure 2c)

## How to apply

Load the MSn Lib InChIKey list from the corinnabrungs/msn_tree_library repository and the InChIKey sets extracted from each reference library, truncating every InChIKey to its first block (14 characters) so that compounds are compared at the skeleton level rather than full stereochemistry. Compute the per-library set-difference against MSn Lib, then aggregate into a union-based tally; for this study the union set-difference reproduces ~22,700 first-InChIKey blocks unique to MSnLib. Build the intersection matrix of the six libraries with the upsetplot Python package (Extended Data Fig. 4 panel), and compute pairwise Tanimoto/structural-similarity projections to construct a TMAP layout over the first-InChIKey-block classes with the tmap package (Figure 2c panel). Rationale: first-block truncation makes cross-library identity comparison robust to differing stereochemical annotation across databases, and the TMAP over block classes exposes chemical-space coverage gaps that a flat count hides. The myopic-mces package can supply structural (MCES) distances for similarity work, with COIN_CMD as the default ILP solver and exact distances only trusted below the threshold.

## Related tools

- **corinnabrungs/msn_tree_library** (Source of the MSn Lib InChIKey compound list and the metadata/PubChem structure resolution used to build the comparison sets) — corinnabrungs/msn_tree_library
- **MassBank/MassBank-data** (Reference library providing MassBank EU InChIKey records for the set-difference comparison) — MassBank/MassBank-data
- **upsetplot** (Python package used to build the UpSet plot of the six-library first-InChIKey block intersection matrix)
- **tmap** (Python package used to construct the TMAP layout over first-InChIKey block classes for the chemical-space projection)
- **myopic-mces** (Optional computation of myopic MCES structural distances for structural-similarity based projections) — AlBi-HHU/myopic-mces
- **pip** (Installs the requirements for running the pipeline)

## Examples

```
pip install requirements
```

## Evaluation signals

- The union-based set-difference tally over first-InChIKey blocks reproduces the paper's reported ~22,700 compounds unique to MSn Lib.
- The UpSet plot's library-coverage intersection pattern matches Figure 2c / Extended Data Fig. 4.
- All InChIKeys are truncated to exactly 14 characters before comparison across all six libraries.
- The TMAP layout clusters first-InChIKey-block classes consistently with the pairwise Tanimoto/structural-similarity projection.
- Per-library set-difference counts are each non-negative and the union count is >= each individual per-library difference.

## Limitations

- Counting at the first-InChIKey block (14 characters) merges stereoisomers and salt forms, so 'new compound' counts are skeleton-level, not full-structure counts.
- Some external databases need a local file and/or special access and must otherwise be set to False in jobs.py, so commercial libraries (e.g. mzCloud, NIST23) may not be fully reproducible.
- If no structure information is provided, structures are queried from PubChem by Name search, which can fail or return wrong structures for ambiguous names.
- For myopic MCES distances, computation modes 5/6 (time-limit results) are only reliable for the COIN_CMD and CPLEX_PY solvers; exact distances below the threshold require an ILP solver to be configured since PuLP does not ship with solvers.

## Evidence

- [abstract] a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules: "a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
- [other] Build an UpSet plot of the six-library first-InChIKey block intersection matrix using the upsetplot Python package: [evidence span withheld — anchor status: not_found]
- [other] Compute pairwise Tanimoto/structural-similarity projections and construct a TMAP layout over the first-InChIKey block classes using the tmap package: [evidence span withheld — anchor status: not_found]
- [other] For each reference library, truncate InChIKeys to the first block (14 characters) and compute the set-difference against MSn Lib: [evidence span withheld — anchor status: not_found]
- [readme] If no structure information is provided, it is queried from PubChem by Name search.: "If no structure information is provided, it is queried from PubChem by Name search."

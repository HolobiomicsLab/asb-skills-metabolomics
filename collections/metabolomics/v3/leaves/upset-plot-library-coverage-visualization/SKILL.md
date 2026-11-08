---
name: upset-plot-library-coverage-visualization
description: Use when comparing compound/structure coverage across multiple spectral
  libraries via InChIKeys and you need intersection/union counts plus an UpSet-style
  visualization of set overlaps.
when_to_use_negative:
- Input is already a per-feature or per-spectrum table (mzML/MGF peak data) rather
  than compound-level InChIKey identifiers — data processing pipelines (mzmine, ThermoRawFileParser)
  are the wrong layer
- Only two libraries are being compared — a simple Venn/venn2 or direct set difference
  suffices and an UpSet intersection matrix adds overhead
- Full-stereoisomer-level (complete InChIKey) overlap is required instead of first-block
  (skeleton-level) comparison — truncating to 14 characters merges stereoisomers
edam_operation: http://edamontology.org/operation_0560
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0121
tools:
- name: upsetplot
  role: Python package used to build the UpSet plot of the six-library first-InChIKey
    block intersection matrix (Extended Data Fig. 4 panel)
- name: tmap
  role: Used to construct the TMAP layout over first-InChIKey block classes for the
    Figure 2c chemical-space panel
- name: MassBank-data
  role: Source of MassBank EU records from which InChIKey sets are extracted as one
    reference library
  repo: https://github.com/MassBank/MassBank-data
- name: msn_tree_library
  role: Repository providing the MSn Lib InChIKey list that is compared against the
    reference libraries
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: pip
  role: Used to install the analysis dependencies (requirements) for running the comparison
    scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:2cc656ebfaf448d82d76dbca6fb35978944c3b3af279a53e752ad7dcca12b0c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MassBank/MassBank-data
  tools:
  - MassBank-data
  - MassBank/MassBank-data
  - corinnabrungs/msn_tree_library
  - msn_tree_library
  - pip
  - tmap
  - upsetplot
  edam_operation: http://edamontology.org/operation_0560
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# upset-plot-library-coverage-visualization

## Summary

Quantify and visualize how many compounds a spectral library (MSn Lib) adds relative to reference libraries by computing first-InChIKey-block (14-character) set differences and rendering them as an UpSet plot, as done to reproduce Figure 2c / Extended Data Fig. 4 showing ~22,700 novel MSn Lib compounds.

## When to use

Use when comparing compound/structure coverage across multiple spectral libraries via InChIKeys and you need intersection/union counts plus an UpSet-style visualization of set overlaps.

## When NOT to use

- Input is already a per-feature or per-spectrum table (mzML/MGF peak data) rather than compound-level InChIKey identifiers — data processing pipelines (mzmine, ThermoRawFileParser) are the wrong layer
- Only two libraries are being compared — a simple Venn/venn2 or direct set difference suffices and an UpSet intersection matrix adds overhead
- Full-stereoisomer-level (complete InChIKey) overlap is required instead of first-block (skeleton-level) comparison — truncating to 14 characters merges stereoisomers

## Inputs

- InChIKey list of MSn Lib compounds from the corinnabrungs/msn_tree_library repository
- InChIKey sets extracted from reference libraries: GNPS (ALL_GNPS_NO_PROPAGATED), MassBank EU (MassBank/MassBank-data records), MoNA, mzCloud, NIST23
- Compound structure identifiers (InChIKeys/SMILES) for pairwise similarity projections and TMAP layout

## Outputs

- Per-library and union set-difference counts table (overlap counts) of first-InChIKey blocks unique to MSn Lib
- UpSet plot of the six-library first-InChIKey block intersection matrix (Extended Data Fig. 4 reproduction)
- TMAP layout of first-InChIKey block classes with pairwise similarity projections (Figure 2c reproduction)

## How to apply

Load the InChIKey list of the target library (MSn Lib from corinnabrungs/msn_tree_library) and the InChIKey sets extracted from each reference library (GNPS ALL_GNPS_NO_PROPAGATED, MassBank EU, MoNA, mzCloud, NIST23). Truncate every InChIKey to the first block (first 14 characters) so that comparison is at the parent-compound/skeleton level rather than full stereochemistry. Compute per-library set differences against the target library and aggregate them into a union-based tally (expected ~22,700 first-InChIKey blocks unique to MSnLib). Build the six-library first-InChIKey-block intersection matrix and render it with the upsetplot Python package; optionally compute pairwise structural-similarity projections and a TMAP layout over the first-InChIKey block classes with the tmap package to reproduce the Figure 2c panel. Write the overlap counts table and both figures to the outputs directory.

## Related tools

- **upsetplot** (Python package used to build the UpSet plot of the six-library first-InChIKey block intersection matrix (Extended Data Fig. 4 panel))
- **tmap** (Used to construct the TMAP layout over first-InChIKey block classes for the Figure 2c chemical-space panel)
- **MassBank-data** (Source of MassBank EU records from which InChIKey sets are extracted as one reference library) — https://github.com/MassBank/MassBank-data
- **msn_tree_library** (Repository providing the MSn Lib InChIKey list that is compared against the reference libraries) — https://github.com/corinnabrungs/msn_tree_library
- **pip** (Used to install the analysis dependencies (requirements) for running the comparison scripts)

## Examples

```
from upsetplot import UpSet; UpSet(intersection_matrix, show_counts=True).plot()  # six-library first-InChIKey[:14] block intersections
```

## Evaluation signals

- The union-based set-difference count must reproduce the paper's reported ~22,700 first-InChIKey blocks unique to MSnLib
- The UpSet plot must show the same library-coverage pattern as Figure 2c / Extended Data Fig. 4 (MSn Lib contributing ~22,700 compounds not covered by GNPS, MassBank EU, MoNA, mzCloud, NIST23)
- All InChIKeys must be truncated to exactly 14 characters (first block) before set operations — verify no full-length keys leak into the comparison
- Per-library set-difference counts, when aggregated to the union, must be internally consistent with the intersection matrix shown in the UpSet plot
- Overlap counts table and both figures are written to the outputs directory without missing reference libraries

## Limitations

- Comparison operates at first-InChIKey-block (14-char) level, so it does not distinguish stereoisomers or salt forms within the same skeleton
- Coverage claims depend on the exact reference-library exports (e.g. GNPS ALL_GNPS_NO_PROPAGATED); different snapshots of commercial libraries like NIST23 or mzCloud will change the count
- External databases used in the broader pipeline need a local file and/or special access; some must be set to False in jobs.py if unavailable

## Evidence

- [results] MSn Lib contributes ~22,700 compounds not covered by existing open (GNPS, MassBank EU, MoNA) and commercial (mzCloud, NIST23) libraries: [evidence span withheld — anchor status: not_found]
- [methods] For each reference library, truncate InChIKeys to the first block (14 characters) and compute the set-difference against MSn Lib: [evidence span withheld — anchor status: not_found]
- [methods] Build an UpSet plot of the six-library first-InChIKey block intersection matrix using the upsetplot Python package: [evidence span withheld — anchor status: not_found]
- [abstract] MS n Lib, a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules: "MS n Lib, a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
- [readme] This repo contains all MassBank records and uses GitHub Actions to validate the content of all records: [evidence span withheld — anchor status: external_doc]

---
name: spectral-library-overlap-quantification
description: Use when you must quantify compound-level overlap between a new spectral
  library and reference open/commercial libraries (GNPS, MassBank EU, MoNA, mzCloud,
  NIST23) using first-InChIKey-block set differences.
when_to_use_negative:
- Input is already a per-compound feature/annotation table rather than raw InChIKey
  identity lists from spectral library records
- Comparison should be at full-InChIKey (stereospecific, second-block) level rather
  than the first-14-character skeleton level used here
- The task is spectral peak/matching overlap rather than compound-identity coverage
  comparison
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: corinnabrungs/msn_tree_library
  role: Source of the MSn Lib InChIKey compound list being compared against reference
    libraries
  repo: corinnabrungs/msn_tree_library
- name: MassBank/MassBank-data
  role: Repository of MassBank records from which the MassBank EU reference InChIKey
    set is extracted
  repo: MassBank/MassBank-data
- name: upsetplot
  role: Python package used to build the six-library first-InChIKey block intersection
    UpSet plot
- name: tmap
  role: Python package used to construct the TMAP layout over first-InChIKey block
    classes for the Figure 2c panel
- name: pip
  role: Installs the Python requirements for the overlap analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:953ab1a7f1a6788cfea885da0fd272dda33f282e008037c69cf432a75eca8a35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/corinnabrungs/msn_tree_library
  tools:
  - MassBank/MassBank-data
  - corinnabrungs/msn_tree_library
  - pip
  - tmap
  - upsetplot
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# spectral-library-overlap-quantification

## Summary

Quantifies how many compounds in a spectral library (e.g. MSn Lib's 30,008 unique small molecules) are absent from reference libraries (GNPS, MassBank EU, MoNA, mzCloud, NIST23) by first-InChIKey-block set operations, and visualizes intersections with UpSet plots and TMAP chemical-space layouts.

## When to use

Use when you must quantify compound-level overlap between a new spectral library and reference open/commercial libraries (GNPS, MassBank EU, MoNA, mzCloud, NIST23) using first-InChIKey-block set differences.

## When NOT to use

- Input is already a per-compound feature/annotation table rather than raw InChIKey identity lists from spectral library records
- Comparison should be at full-InChIKey (stereospecific, second-block) level rather than the first-14-character skeleton level used here
- The task is spectral peak/matching overlap rather than compound-identity coverage comparison

## Inputs

- InChIKey list from MSn Lib (corinnabrungs/msn_tree_library)
- InChIKey sets from GNPS (ALL_GNPS_NO_PROPAGATED)
- InChIKey sets from MassBank EU (MassBank/MassBank-data)
- InChIKey sets from MoNA
- InChIKey sets from mzCloud
- InChIKey sets from NIST23

## Outputs

- Per-library and union-based set-difference counts of compounds unique to the target library (~22,700 first-InChIKey blocks for MSn Lib)
- UpSet plot of the six-library first-InChIKey block intersection matrix
- TMAP layout over first-InChIKey block classes based on pairwise Tanimoto/structural similarity
- Overlap counts table written to the outputs directory

## How to apply

Load the target library's InChIKey list (e.g. from corinnabrungs/msn_tree_library) and the InChIKey sets extracted from each reference library (GNPS as ALL_GNPS_NO_PROPAGATED, MassBank EU from MassBank/MassBank-data, MoNA, mzCloud, NIST23). Truncate every InChIKey to its first block (first 14 characters) to compare at the skeleton level, then compute the per-library set-difference against the target library and aggregate into a union-based tally of compounds absent from all references — MSn Lib yields ~22,700 such first-InChIKey blocks. Build an UpSet plot of the six-library first-InChIKey block intersection matrix with the upsetplot Python package (reproducing Extended Data Fig. 4) and compute pairwise Tanimoto/structural-similarity projections to build a TMAP layout over first-InChIKey block classes with the tmap package (reproducing Figure 2c). Write the overlap counts table and both figures to the outputs directory. Validate by checking the union set-difference count reproduces the reported ~22,700 new compounds and that the coverage pattern in the figures matches Figure 2c / Extended Data Fig. 4.

## Related tools

- **corinnabrungs/msn_tree_library** (Source of the MSn Lib InChIKey compound list being compared against reference libraries) — corinnabrungs/msn_tree_library
- **MassBank/MassBank-data** (Repository of MassBank records from which the MassBank EU reference InChIKey set is extracted) — MassBank/MassBank-data
- **upsetplot** (Python package used to build the six-library first-InChIKey block intersection UpSet plot)
- **tmap** (Python package used to construct the TMAP layout over first-InChIKey block classes for the Figure 2c panel)
- **pip** (Installs the Python requirements for the overlap analysis)

## Examples

```
msn_new = {k[:14] for k in msn_ik} - {k[:14] for lib in [gnps, massbank, mona, mzcloud, nist23] for k in lib}; print(len(msn_new))
```

## Evaluation signals

- Union-based set-difference count reproduces the paper's reported ~22,700 first-InChIKey blocks unique to MSn Lib
- UpSet plot shows the same library-coverage pattern as Figure 2c / Extended Data Fig. 4
- All InChIKeys truncated to exactly the first block (14 characters) before comparison
- TMAP layout clusters first-InChIKey block classes consistently with the pairwise Tanimoto/structural-similarity projections
- Overlap counts table and both figures present in the outputs directory

## Limitations

- First-InChIKey-block (14-character) comparison is at skeleton level and ignores stereochemical/isomeric differences in the second block
- Some reference libraries require local files and/or special access; mzCloud and NIST23 are commercial and their extraction depends on availability
- Commercial library sets (mzCloud, NIST23) may not be fully reproducible if source exports change

## Evidence

- [other] MSn Lib contributes ~22,700 compounds not covered by existing open (GNPS, MassBank EU, MoNA) and commercial (mzCloud, NIST23) libraries: [evidence span withheld — anchor status: not_found]
- [other] truncate InChIKeys to first block and set-difference: [evidence span withheld — anchor status: not_found]
- [other] upsetplot build step: [evidence span withheld — anchor status: not_found]
- [other] TMAP step: [evidence span withheld — anchor status: not_found]
- [abstract] MSn Lib scale: "a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"

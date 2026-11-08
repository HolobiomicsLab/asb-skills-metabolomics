---
name: inchikey-set-difference-analysis
description: Use when you have InChIKey or SMILES lists from a new library and several
  existing open/commercial spectral libraries and need to quantify which compounds
  are novel.
when_to_use_negative:
- Inputs are raw spectra (.raw, .mzML) rather than curated InChIKey/compound lists
- The goal is stereochemistry-level (full-InChIKey) comparison rather than first-block
  skeleton-level novelty
- Only one reference library is available, making an UpSet intersection matrix uninformative
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3314
tools:
- name: corinnabrungs/msn_tree_library
  role: Source of the MSnLib InChIKey list and metadata cleanup pipeline
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: MassBank/MassBank-data
  role: Reference library record source (MassBank EU) for overlap comparison
  repo: https://github.com/MassBank/MassBank-data
- name: Prefect
  role: Orchestrates the metadata cleanup flow that produces the structured compound/InChIKey
    lists
- name: pip
  role: Installs the Python environment with upsetplot and tmap packages
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:bbdc6f2322275869f0c9c1aedadb818a4b9f475940a73be7d6c2b30b52009c39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/corinnabrungs/msn_tree_library
  tools:
  - MassBank/MassBank-data
  - Prefect
  - corinnabrungs/msn_tree_library
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# inchikey-set-difference-analysis

## Summary

Compute how many MSnLib compounds are new by comparing first-InChIKey-block sets against reference spectral libraries (GNPS, MassBank EU, MoNA, mzCloud, NIST23), then visualize the coverage gaps as UpSet and TMAP figures.

## When to use

Use when you have InChIKey or SMILES lists from a new library and several existing open/commercial spectral libraries and need to quantify which compounds are novel.

## When NOT to use

- Inputs are raw spectra (.raw, .mzML) rather than curated InChIKey/compound lists
- The goal is stereochemistry-level (full-InChIKey) comparison rather than first-block skeleton-level novelty
- Only one reference library is available, making an UpSet intersection matrix uninformative

## Inputs

- InChIKey list from MSnLib (corinnabrungs/msn_tree_library)
- InChIKey or compound lists from GNPS (ALL_GNPS_NO_PROPAGATED), MassBank EU, MoNA, mzCloud, NIST23
- SMILES/structural information for Tanimoto/TMAP similarity computation

## Outputs

- Union set-difference count (~22,700 new first-InChIKey blocks)
- UpSet plot of six-library first-InChIKey-block intersection matrix (Extended Data Fig. 4)
- TMAP layout of first-InChIKey-block classes (Figure 2c)
- Per-library overlap count table

## How to apply

Load the MSnLib InChIKey list from the corinnabrungs/msn_tree_library repository alongside InChIKey sets extracted from GNPS (ALL_GNPS_NO_PROPAGATED), MassBank EU, MoNA, mzCloud and NIST23. Truncate every InChIKey to its first block (14 characters) so that stereoisomer-level entries collapse to the same standard-inchi skeleton class, which is the granularity the paper uses for library overlap. Compute the set-difference of MSnLib minus each reference library individually, then take the union across all references to obtain the ~22,700 first-InChIKey blocks that no existing open or commercial library covers. Build an UpSet intersection matrix over the six libraries with the upsetplot Python package (Extended Data Fig. 4) and construct a TMAP layout over first-InChIKey-block classes using pairwise Tanimoto/structural similarity (Figure 2c). Write the overlap count table and both figures to the outputs directory for reproducibility.

## Related tools

- **corinnabrungs/msn_tree_library** (Source of the MSnLib InChIKey list and metadata cleanup pipeline) — https://github.com/corinnabrungs/msn_tree_library
- **MassBank/MassBank-data** (Reference library record source (MassBank EU) for overlap comparison) — https://github.com/MassBank/MassBank-data
- **Prefect** (Orchestrates the metadata cleanup flow that produces the structured compound/InChIKey lists)
- **pip** (Installs the Python environment with upsetplot and tmap packages)

## Examples

```
python -c "from upsetplot import UpSet; UpSet.from_contents({'GNPS': set1, 'MassBankEU': set2, 'MoNA': set3, 'mzCloud': set4, 'NIST23': set5, 'MSnLib': set6}).plot(); print(sum(all_sets - union_refs))"
```

## Evaluation signals

- Union set-difference across all six reference libraries must reproduce the paper's reported ~22,700 new compounds
- UpSet intersection matrix must match the library-coverage pattern shown in Extended Data Fig. 4
- TMAP layout must visually match Figure 2c's chemical-space coverage projection
- Every InChIKey in the comparison is truncated to exactly the first block (14 characters)
- Per-library set-difference counts must be consistent with (never larger than) the union count

## Limitations

- mzCloud and NIST23 require special access or local files; if unavailable they must be set to False in jobs.py, which changes the union count
- First-block truncation collapses stereoisomers, so stereochemical novelty is not captured
- The ~22,700 figure depends on the exact snapshot versions of each reference library; different versions shift the overlap

## Evidence

- [full_text] Union of six first-InChIKey-block set differences: [evidence span withheld — anchor status: not_found]
- [readme] Library records are validated data: [evidence span withheld — anchor status: external_doc]
- [methods] First-block truncation rationale: "If no structure information is provided, it is queried from PubChem by Name search."
- [readme] Metadata template requirement: "Please use the [template](https://docs.google.com/spreadsheets/d/1v6_IlGS3VgycGc-mSSdNeocY-CFXpONVZbuh3XNLX2E/edit?usp=sharing) for your metadata for having same column names and minimum needed"

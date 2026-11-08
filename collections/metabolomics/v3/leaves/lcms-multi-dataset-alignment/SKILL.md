---
name: lcms-multi-dataset-alignment
description: Use when two or more nontargeted LC-MS feature tables (Feature Metadata
  + Feature Abundances) from independently processed but same-method datasets must
  be merged into one complete dataset, and existing pairwise alignment tools do not
  fully meet the need.
when_to_use_negative:
- The features have not yet been independently processed into Feature Metadata + Feature
  Abundances tables (Eclipse aligns already-extracted features, not raw data — use
  Chroma for reading .raw and .mzML files)
- The datasets were produced by different LC-MS methods rather than same-method nontargeted
  datasets
- 'The goal is clustering redundant features within a single dataset rather than aligning
  two or more datasets (that is Gravity''s role: cluster redundant LCMS features based
  on RT and Correlation)'
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Eclipse
  role: 'Core aligner: its MSAligner loads subalignment match tables into a directed
    graph and runs iterative group-ranking clustering to form cross-dataset feature
    groups'
  repo: https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- name: bmxp
  role: Parent package providing the shared schema (bmxp.FMDATA / bmxp.IMDATA) used
    to load and merge Feature Metadata and Feature Abundances before alignment
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Runtime language for Eclipse MSAligner invocation, cluster-statistic computation,
    and CSV output
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:f1b92ec042bb0c80b2ca0bd2a39dbfe03c99cf6998d1bc0bed6c04fdca7c1ad6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lcms-multi-dataset-alignment

## Summary

Align two or more independently processed, same-method nontargeted LC-MS metabolomics datasets into a Combined Dataset using the Eclipse MSAligner graph-based matching and clustering algorithm from the Broad Institute's bmxp package.

## When to use

Use when two or more nontargeted LC-MS feature tables (Feature Metadata + Feature Abundances) from independently processed but same-method datasets must be merged into one complete dataset, and existing pairwise alignment tools do not fully meet the need.

## When NOT to use

- The features have not yet been independently processed into Feature Metadata + Feature Abundances tables (Eclipse aligns already-extracted features, not raw data — use Chroma for reading .raw and .mzML files)
- The datasets were produced by different LC-MS methods rather than same-method nontargeted datasets
- The goal is clustering redundant features within a single dataset rather than aligning two or more datasets (that is Gravity's role: cluster redundant LCMS features based on RT and Correlation)

## Inputs

- Per-dataset merged Feature Metadata + Feature Abundances tables conforming to the shared bmxp schema (Compound_ID feature index, injection_id injection index)
- Alignment configuration: minimum group size, minimum clique size, diameter (1, 2, or 3), Intensity weighting on/off
- Alignment mode: one-by-all (reference dataset vs each other dataset) or all-by-all

## Outputs

- Aligned feature table with cluster assignments (e.g. eclipse_alignment_results.csv, eclipse_alignment_nonclique.csv)
- Cluster overlap statistics: features in all datasets, features in n>=k datasets, fragmented clusters of size n<4, full vs partial match counts (e.g. alignment_summary.csv)
- Feature group assignments across datasets keyed by feature ID per the shared schema

## How to apply

First, load each dataset's Feature Metadata (index `Compound_ID`) and Feature Abundances pivot table (`Compound_ID` x `injection_id`) using the shared bmxp schema (bmxp.FMDATA / bmxp.IMDATA), and merge Metadata + Abundances per dataset as Eclipse requires. Then import MSAligner from bmxp.eclipse and configure the alignment: choose a diameter setting (1 for strict clique-based clustering, 2 or 3 for nonclique), set minimum group size and minimum clique size, and optionally disable Intensity weighting. Internally, all subalignment match tables are loaded into a directed graph in which features are nodes, matches are edges, and match penalties are edge weights; only bidirectional edges are retained, with penalties summed from both directions, before an iterative group-ranking clustering algorithm forms feature groups across datasets. Finally, tally cluster overlap statistics (e.g. features present in all datasets, features in n>=4 or n>=6 datasets, fragmented clusters of size <4, full vs partial matches) and write the aligned feature table with cluster assignments. Rerun with a different dataset insertion order to confirm order-independent feature group assignments.

## Related tools

- **Eclipse** (Core aligner: its MSAligner loads subalignment match tables into a directed graph and runs iterative group-ranking clustering to form cross-dataset feature groups) — https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- **bmxp** (Parent package providing the shared schema (bmxp.FMDATA / bmxp.IMDATA) used to load and merge Feature Metadata and Feature Abundances before alignment) — https://github.com/broadinstitute/bmxp
- **Python** (Runtime language for Eclipse MSAligner invocation, cluster-statistic computation, and CSV output)

## Examples

```
pip install bmxp; from bmxp.eclipse import MSAligner
```

## Evaluation signals

- Reference-count reproduction: one-by-all DS5 vs DS6-DS9 alignment must yield 1600 of 12461 DS5 features matched in all datasets, 3681 partial matches, 7180 unmatched, and 130 of 140 all-dataset overlapping features fully matched with 10 partially matched
- Six-plasma all-by-all clique alignment (diameter 1, min group/clique size 1) must yield 1435 features common to all six datasets, 6376 in n>=4 datasets, and 47522 fragmented clusters of size <4 (reported runtime 52 s, hardware-dependent)
- All-eleven-dataset nonclique alignment (Intensity disabled, diameter 2) must reproduce 976 features in all datasets and 5334 in n>=6 datasets per Supplementary Table S2
- Order-independence invariant: rerun the alignment with a different dataset insertion order and confirm identical feature group assignments
- Schema check: input tables use Compound_ID as feature index and injection_id as injection index unless bmxp.FMDATA/IMDATA dictionaries were modified before running

## Limitations

- Reported runtimes (e.g. 52 s for the six-plasma alignment) are hardware-dependent and left to expert review
- Existing software for multi-dataset (n > 2) alignment scenarios was deemed insufficient, which motivated Eclipse's graph-based approach; strictly clique-based clustering with diameter 1 fragments clusters of size <4 (47522 such clusters observed)
- No version numbers for bmxp or its modules are documented; Gravity's XIC shape-based clustering is not yet implemented
- Schema column headers must match package defaults (Compound_ID, injection_id) unless the FMDATA/IMDATA dictionaries are modified directly prior to running code

## Evidence

- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n > 2 datasets.: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [methods] Eclipse loads all subalignment match tables into a directed graph in which features are nodes, matches are edges, and match penalties are edge weights.: [evidence span withheld — anchor status: not_found]
- [readme] Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances.: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [intro] Eclipse is open source (https://github.com/broadinstitute/bmxp) and can be installed via pip install bmxp.: "Eclipse is open source (https://github.com/broadinstitute/bmxp) and can be installed via “pip install bmxp.”"

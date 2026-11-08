---
name: graph-based-feature-matching
description: Use when two or more independently processed same-method nontargeted
  LC-MS feature tables (Feature Metadata + Feature Abundances) must be merged into
  one dataset and features matched across all of them.
when_to_use_negative:
- Only a single dataset needs processing — alignment requires two or more same-method
  nontargeted LC-MS datasets
- The goal is clustering redundant features within one dataset by retention time and
  correlation — use Gravity's cluster, not Eclipse
- Data files are raw instrument output (.raw/.mzml) — these must first be processed
  into feature tables (e.g., via Chroma) before alignment
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Eclipse
  role: Provides MSAligner, the graph-based aligner that matches features across two
    or more datasets in clique mode
  repo: https://github.com/broadinstitute/bmxp
- name: bmxp
  role: Package supplying the shared schema (FMDATA/IMDATA dictionaries) and merged
    Feature Metadata + Feature Abundances input format that MSAligner consumes
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Execution environment in which the aligner is instantiated, configured, and
    run in the reproduced workflows
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:a036505b412cc489d7fe5a06e95568afdcb0e404c83520c863560f9359091fdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/bmxp
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# graph-based-feature-matching

## Summary

Align two or more independently processed nontargeted LC-MS metabolomics feature tables using Eclipse's graph-based MSAligner, which resolves complex matching scenarios that arise when more than two datasets must be matched.

## When to use

Use when two or more independently processed same-method nontargeted LC-MS feature tables (Feature Metadata + Feature Abundances) must be merged into one dataset and features matched across all of them.

## When NOT to use

- Only a single dataset needs processing — alignment requires two or more same-method nontargeted LC-MS datasets
- The goal is clustering redundant features within one dataset by retention time and correlation — use Gravity's cluster, not Eclipse
- Data files are raw instrument output (.raw/.mzml) — these must first be processed into feature tables (e.g., via Chroma) before alignment

## Inputs

- Feature Metadata table per dataset (bmxp.FMDATA schema, Compound_ID index, RT/MZ/Intensity/annotation columns)
- Feature Abundances pivot table per dataset (Compound_ID x injection_id)
- Alignment configuration parameters (min group size, min clique size, diameter, intensity weighting)

## Outputs

- Aligned feature table with cross-dataset cluster/match assignments (e.g., eclipse_alignment_results.csv)
- Summary counts of full matches, partial matches, and unmatched features (e.g., alignment_summary.csv)

## How to apply

Load each dataset's Feature Metadata and Feature Abundances tables under the bmxp shared schema (Compound_ID x injection_id pivot) and merge them, because Eclipse requires the merged representation. Optionally remap column headers via the bmxp.FMDATA/IMDATA dictionary objects so all datasets use consistent terminology. Instantiate MSAligner from bmxp.eclipse and run the alignment in strict clique mode — the reproduced workflows used diameter=1 for all-by-all plasma alignments and min group/clique size 2 with diameter 2 and Intensity weighting disabled for the one-by-all plasma-vs-tissue alignment; set minimum group/clique size to 1 only when singletons should be retained as clusters. Eclipse represents matches as a graph rather than pairwise chaining, which handles the complex matching scenarios arising from n > 2 datasets. Finally, tally alignment outcomes (e.g., features present in all datasets, partial matches, unmatched/fragmented clusters) and write the aligned feature table with cluster assignments to CSV.

## Related tools

- **Eclipse** (Provides MSAligner, the graph-based aligner that matches features across two or more datasets in clique mode) — https://github.com/broadinstitute/bmxp
- **bmxp** (Package supplying the shared schema (FMDATA/IMDATA dictionaries) and merged Feature Metadata + Feature Abundances input format that MSAligner consumes) — https://github.com/broadinstitute/bmxp
- **Python** (Execution environment in which the aligner is instantiated, configured, and run in the reproduced workflows)

## Examples

```
import bmxp; from bmxp.eclipse import MSAligner; aligner = MSAligner(merged_fmdatas_abundances_DS1_4); results = aligner.align(min_group_size=1, min_clique_size=1, diameter=1)
```

## Evaluation signals

- Reproduced DS1-DS4 clique-mode alignment yields 3861 combined features with 362/381 (95%) correctly matched overlapping annotated features and 3 spurious annotation-mismatch rows
- One-by-all alignment of DS5 vs DS6-DS9 reproduces the breakdown of 12461 DS5 features into 1600 full, 3681 partial, and 7180 unmatched, with 130/140 all-dataset overlapping features fully matched
- Six-plasma all-by-all alignment (min group/clique size 1, diameter 1) yields 1435 features common to all six datasets, 6376 in >=4 datasets, and 47522 fragmented clusters of size <4
- Input tables conform to the bmxp shared schema: Compound_ID as feature index and injection_id as injection index before merging

## Limitations

- Reported runtimes (~16 s for DS1-DS4; 52 s for the six-plasma alignment) are hardware-dependent and treated as expert-review rather than pass/fail criteria
- BMXP module versions are not stated, and the modules are standalone with a shared schema but no end-to-end pipeline yet
- Alignment requires the merged Feature Metadata + Feature Abundances representation; feeding unmerged tables is unsupported by Eclipse

## Evidence

- [abstract] graph-based approach for n>2 datasets: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [methods] MSAligner import: "from bxmp.eclipse import MSAligner"
- [methods] merge requirement: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Eclipse purpose per README: "Align two or more same-method nontargeted LCMS datasets"

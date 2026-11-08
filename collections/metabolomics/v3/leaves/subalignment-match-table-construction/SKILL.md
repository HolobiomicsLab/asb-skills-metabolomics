---
name: subalignment-match-table-construction
description: Use when two or more same-method, independently processed nontargeted
  LC-MS feature tables (Feature Metadata + Feature Abundances) must be aligned, and
  pairwise Source-to-Target feature matches must be identified and ranked.
when_to_use_negative:
- Datasets were processed by different LC-MS methods — Eclipse explicitly aligns 'same-method
  nontargeted LCMS datasets'
- You need drift correction within a single batch — that is Blueshift DriftCorrection's
  role (pooled technical replicates and internal standards), not pairwise Eclipse
  alignment
- You only need to cluster redundant features within one dataset — Gravity cluster
  (RT and correlation based) is the appropriate module
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Eclipse
  role: 'Performs the pairwise subalignment: LOWESS RT/m/z scaling, RSE cutoffs, penalty-ranked
    matching, and graph-based aggregation of match tables'
  repo: https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- name: bmxp
  role: Provides the shared schema (bmxp.FMDATA, IMDATA) and column-header remapping
    used to write and consume the match tables
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Language in which Eclipse and bmxp are implemented and in which the match
    tables are generated and validated
  repo: https://github.com/broadinstitute/bmxp
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:5b9142ea0403ea320387bbebd7262d2810575963ed8ece72b018e3730f85bc29
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# subalignment-match-table-construction

## Summary

Constructs pairwise subalignment match tables between independently processed nontargeted LC-MS datasets using Eclipse (bmxp): survey-based LOWESS RT/m/z scalers, ±6 RSE candidate cutoffs, and penalty-ranked one-to-one best matches, written in the shared bmxp schema so matches can later be re-clustered without rerunning subalignments.

## When to use

Use when two or more same-method, independently processed nontargeted LC-MS feature tables (Feature Metadata + Feature Abundances) must be aligned, and pairwise Source-to-Target feature matches must be identified and ranked.

## When NOT to use

- Datasets were processed by different LC-MS methods — Eclipse explicitly aligns 'same-method nontargeted LCMS datasets'
- You need drift correction within a single batch — that is Blueshift DriftCorrection's role (pooled technical replicates and internal standards), not pairwise Eclipse alignment
- You only need to cluster redundant features within one dataset — Gravity cluster (RT and correlation based) is the appropriate module

## Inputs

- Merged Feature Metadata (bmxp.FMDATA) + Feature Abundances pivot (Compound_ID x injection_id) per dataset
- DS1–DS4 plasma dataset feature tables with Compound_ID feature index and injection_id injection index
- Source and Target dataset pair specification for each pairwise subalignment

## Outputs

- subalignment_matches_<source>_vs_<target>.csv per pair with source/target Compound_ID, RT, MZ, RSE-scaled distance, and penalty rank
- One-to-one best-match assignments between Source and Target Compound_IDs
- Match tables in the shared bmxp column format usable as graph edges for downstream clustering

## How to apply

First merge bmxp.FMDATA with the Feature Abundances pivot (Compound_ID x injection_id) for each dataset, since Eclipse requires merged Feature Metadata + Feature Abundances as input. For each Source→Target dataset pair, fit survey-based inter-dataset retention-time and m/z scalers via LOWESS on the shared features, then compute the Residual Standard Error (RSE) of each LOWESS fit and weight residual distances by the RSE. Retain candidate matches as Target features within ±6 RSE of the Source's RT, m/z, and Intensity. Rank candidates per feature with the best-match penalty score and assign one-to-one best matches between Source and Target Compound_IDs, decoupling pairwise matching from graph-based aggregation so match tables can be re-clustered later (e.g. diameter=2, min clique size=3) without rerunning subalignments. Write each subalignment_matches_<source>_vs_<target>.csv with required bmxp FMDATA fields (Compound_ID, RT, MZ) using shared column headers so headers can be remapped globally.

## Related tools

- **Eclipse** (Performs the pairwise subalignment: LOWESS RT/m/z scaling, RSE cutoffs, penalty-ranked matching, and graph-based aggregation of match tables) — https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- **bmxp** (Provides the shared schema (bmxp.FMDATA, IMDATA) and column-header remapping used to write and consume the match tables) — https://github.com/broadinstitute/bmxp
- **Python** (Language in which Eclipse and bmxp are implemented and in which the match tables are generated and validated) — https://github.com/broadinstitute/bmxp

## Examples

```
import bmxp; from bmxp.eclipse import MSAligner; aligner = MSAligner(); aligner.align(source_merged, target_merged)
```

## Evaluation signals

- Each subalignment_matches_<source>_vs_<target>.csv exists and contains the required bmxp FMDATA fields (Compound_ID, RT, MZ)
- Assigned best matches are one-to-one between Source and Target Compound_IDs within the ±6 RSE cutoff (expert review of one-to-one assignment)
- All match tables use the shared bmxp column headers so headers can be remapped globally via the bmxp.FMDATA/IMDATA dictionaries
- Re-clustering the match tables at diameter=2, min clique size=3 reproduces the reference counts (4530 matched features, 367/381 overlapping annotations, 13 spurious matches)

## Limitations

- The ±6 RSE cutoff and penalty ranking can admit spurious matches; strict-clique versus relaxed (diameter=2, min clique 3) clustering shifts spurious match counts (13 observed after relaxed re-clustering)
- Match tables and clustering are decoupled, so changing clustering parameters changes match/annotation counts without rerunning subalignments — counts are parameter-dependent
- No version numbers for bmxp or its modules are documented in the context

## Evidence

- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios arising from n > 2 datasets: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [methods] Merge FMDATA + Feature Abundances for Eclipse input: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Shared schema across bmxp modules: "All BMXP modules use a shared schema and file formats with our prefered columns headers."
- [intro] Eclipse scope: same-method nontargeted LCMS alignment: "Align two or more same-method nontargeted LCMS datasets"

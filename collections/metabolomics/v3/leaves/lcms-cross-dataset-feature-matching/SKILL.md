---
name: lcms-cross-dataset-feature-matching
description: Use when two or more independently processed same-method nontargeted
  LC-MS datasets must be aligned to form a complete dataset, and you need pairwise
  Source→Target subalignment match tables.
when_to_use_negative:
- You need redundant-feature clustering based on RT and correlation — that is Gravity's
  role, not Eclipse's subalignment matching
- You need drift correction of pooled technical replicates and internal standards
  — that is Blueshift's DriftCorrection step
- Datasets were produced by different LC-MS methods — Eclipse is scoped to align same-method
  nontargeted LCMS datasets
edam_topics:
- http://edamontology.org/topic_3369
- http://edamontology.org/topic_0091
tools:
- name: Eclipse
  role: Performs the pairwise subalignment matching via MSAligner, including LOWESS
    scaling, ±6 RSE cutoffs, and penalty-score best-match assignment
  repo: github.com/broadinstitute/bmxp
- name: bmxp
  role: Provides the shared schema (bmxp.FMDATA, bmxp.IMDATA) and global header remapping
    used to standardize match-table columns across modules
  repo: github.com/broadinstitute/bmxp
- name: Python
  role: Runtime language for driving the MSAligner and merging FMDATA with Feature
    Abundances
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:d7611de5c6b216c32f0d9fac70f7c32d6fdbf6f773d3a6f13d9584ec65fe58c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3369
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# lcms-cross-dataset-feature-matching

## Summary

Cross-dataset nontargeted LC-MS feature alignment via Eclipse (bmxp): scale the Source dataset to the Target with survey-based LOWESS RT/m-z scalers, retain candidate matches within ±6 RSE on RT, m/z and Intensity, and assign one-to-one best matches by penalty-score ranking.

## When to use

Use when two or more independently processed same-method nontargeted LC-MS datasets must be aligned to form a complete dataset, and you need pairwise Source→Target subalignment match tables.

## When NOT to use

- You need redundant-feature clustering based on RT and correlation — that is Gravity's role, not Eclipse's subalignment matching
- You need drift correction of pooled technical replicates and internal standards — that is Blueshift's DriftCorrection step
- Datasets were produced by different LC-MS methods — Eclipse is scoped to align same-method nontargeted LCMS datasets

## Inputs

- Feature Metadata table (bmxp.FMDATA) indexed by Compound_ID with RT, MZ, and Intensity fields
- Feature Abundances pivot table (Compound_ID x injection_id)
- Merged Feature Metadata + Feature Abundances tables for each of the DS1–4 plasma datasets (required by Eclipse)

## Outputs

- Pairwise subalignment match tables (subalignment_matches_<source>_vs_<target>.csv) with bmxp FMDATA fields (Compound_ID, RT, MZ), RSE-scaled distances, and penalty rank
- One-to-one best-match assignments between Source and Target Compound_IDs within the ±6 RSE cutoff

## How to apply

For each Source→Target dataset pair, first merge Feature Metadata (bmxp.FMDATA) with the Feature Abundances pivot (Compound_ID x injection_id), since Eclipse requires merged Feature Metadata + Feature Abundances as input. Fit survey-based inter-dataset retention-time and m/z scalers via LOWESS on the shared features between the two merged tables, then compute the Residual Standard Error (RSE) of each LOWESS fit. Weight residual distances by the RSE and apply the ±6 RSE cutoff on the Source's RT, m/z and Intensity to retain candidate Target feature matches. Rank candidates per feature using the best-match penalty ranking and assign one-to-one best matches between Source and Target Compound_IDs. Write each pairwise table to CSV using the bmxp shared column headers so headers can be remapped globally if needed.

## Related tools

- **Eclipse** (Performs the pairwise subalignment matching via MSAligner, including LOWESS scaling, ±6 RSE cutoffs, and penalty-score best-match assignment) — github.com/broadinstitute/bmxp
- **bmxp** (Provides the shared schema (bmxp.FMDATA, bmxp.IMDATA) and global header remapping used to standardize match-table columns across modules) — github.com/broadinstitute/bmxp
- **Python** (Runtime language for driving the MSAligner and merging FMDATA with Feature Abundances)

## Examples

```
from bmxp.eclipse import MSAligner
```

## Evaluation signals

- Verify each subalignment_matches_<source>_vs_<target>.csv exists for every Source→Target pair
- Check each match table contains the required bmxp FMDATA fields (Compound_ID, RT, MZ)
- Confirm assigned best matches are one-to-one between Source and Target Compound_IDs
- Confirm retained candidate matches fall within the ±6 RSE cutoff on RT, m/z, and Intensity
- Verify column headers match the bmxp shared schema after any global remapping (e.g. Compound_ID renamed to Feature_ID)

## Limitations

- The shared schema column labels (e.g. Compound_ID, injection_id) can be changed globally by modifying the module dictionaries before running, so tables written under remapped headers must be interpreted accordingly
- BMXP modules are standalone and pipeline linking is still in progress ('eventually may have a pipeline ability to run all steps'), so each step must be invoked and validated individually
- RT in bmxp.FMDATA is unitless and 'may or may not be scaled', so the LOWESS scaling step is essential before applying RSE-based cutoffs across datasets

## Evidence

- [abstract] Graph-based alignment scope: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [methods] Merging requirement: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [methods] Import path for the aligner: "from bxmp.eclipse import MSAligner"
- [intro] Eclipse scope: "Eclipse: align two or more same-method nontargeted LCMS datasets"
- [readme] Shared schema: "All BMXP modules use a shared schema and file formats with our prefered columns headers."
- [readme] FMDATA definition: "Feature Metadata `bmxp.FMDATA` - Describes the feature. Index default is `Compound_ID`"

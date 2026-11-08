---
name: lcms-feature-alignment-across-datasets
description: Use when two or more independently processed, same-method nontargeted
  LC-MS feature datasets — each a merged Feature Metadata + Feature Abundances table
  — must be combined into one aligned feature set with overlapping features matched.
when_to_use_negative:
- De-replicating redundant features within a single dataset — use Gravity cluster
  (RT and correlation) instead; Eclipse aligns features across datasets, not within
  one.
- Correcting abundance drift across batches — use Blueshift DriftCorrection with pooled
  technical replicates and internal standards.
- Datasets acquired with different LC-MS methods — Eclipse is specified for aligning
  same-method nontargeted LCMS datasets only.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Eclipse
  role: Provides MSAligner, which performs the graph-based strict clique-mode (diameter=1)
    alignment of the merged feature tables across all input datasets
  repo: https://github.com/broadinstitute/bmxp
- name: bmxp
  role: Parent package (pip install bmxp) supplying the shared-schema dictionaries
    bmxp.FMDATA and bmxp.IMDATA used to harmonize Feature Metadata and Injection Metadata
    column headers prior to alignment
  repo: https://github.com/broadinstitute/bmxp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:e164936129bd987fb855cb7410184754434ffb1deb3c59aefc1ceb1f4a0ba50e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/broadinstitute/bmxp
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# lcms-feature-alignment-across-datasets

## Summary

Aligns two or more independently processed, same-method nontargeted LC-MS metabolomics datasets into a single combined feature table using Eclipse's graph-based MSAligner in strict clique mode, with shared-schema header harmonization via bmxp. Validated on four human plasma datasets (DS1–DS4): 3861 combined features with 95% (362/381) of overlapping annotated features correctly matched.

## When to use

Use when two or more independently processed, same-method nontargeted LC-MS feature datasets — each a merged Feature Metadata + Feature Abundances table — must be combined into one aligned feature set with overlapping features matched.

## When NOT to use

- De-replicating redundant features within a single dataset — use Gravity cluster (RT and correlation) instead; Eclipse aligns features across datasets, not within one.
- Correcting abundance drift across batches — use Blueshift DriftCorrection with pooled technical replicates and internal standards.
- Datasets acquired with different LC-MS methods — Eclipse is specified for aligning same-method nontargeted LCMS datasets only.

## Inputs

- Merged Feature Metadata + Feature Abundances table per dataset (Feature Metadata indexed by Compound_ID; Feature Abundances as a Compound_ID x injection_id pivot)
- Two or more independently processed same-method nontargeted LC-MS feature datasets (e.g. the DS1–DS4 human plasma datasets)
- Annotation columns in Feature Metadata (Annotation_ID, Adduct, Metabolite) for post-alignment concordance checks

## Outputs

- Combined aligned feature table spanning all input datasets (3861 combined features for DS1–DS4 in strict clique mode)
- Alignment mapping assigning corresponding features (cliques) across the n input datasets
- Annotation concordance tallies distinguishing correctly matched rows from spurious annotation-mismatch rows (e.g. 362/381 matched, 3 spurious)

## How to apply

Merge each dataset's Feature Metadata with its Feature Abundances pivot (Compound_ID x injection_id), since Blueshift and Eclipse both require the merged representation. Harmonize column headers across datasets through the shared schema by modifying the bmxp.FMDATA / bmxp.IMDATA dictionary objects (e.g. bmxp.FMDATA['Compound_ID'] = 'Feature_ID') so all modules use identical terminology before running. Instantiate and run MSAligner from bmxp.eclipse across the datasets in default strict clique mode (diameter=1); the graph-based approach resolves complex matching scenarios that arise with n > 2 datasets, so no pairwise stitching is needed. Tabulate the combined aligned features and evaluate annotation concordance, counting correctly matched and spurious annotation-mismatch rows. In the DS1–DS4 human plasma benchmark this yielded 3861 features (29% of the smallest dataset, DS2) with 362/381 (95%) of overlapping annotated features correctly matched and 3 spurious rows; the reported ~16 s runtime is hardware-dependent expert review, not a pass/fail criterion.

## Related tools

- **Eclipse** (Provides MSAligner, which performs the graph-based strict clique-mode (diameter=1) alignment of the merged feature tables across all input datasets) — https://github.com/broadinstitute/bmxp
- **bmxp** (Parent package (pip install bmxp) supplying the shared-schema dictionaries bmxp.FMDATA and bmxp.IMDATA used to harmonize Feature Metadata and Injection Metadata column headers prior to alignment) — https://github.com/broadinstitute/bmxp

## Examples

```
import bmxp; from bmxp.eclipse import MSAligner; bmxp.FMDATA['Compound_ID'] = 'Feature_ID'
```

## Evaluation signals

- Combined feature count for DS1–DS4 in default strict clique mode is 3861 (29% of the smallest dataset, DS2); large deviations indicate misconfigured inputs or non-default parameters.
- Overlapping annotated features match at 95% (362/381) with exactly 3 spurious annotation-mismatch rows; a higher spurious rate points to bad header remapping or unmerged Feature Metadata + Feature Abundances.
- Schema check: column headers follow the shared schema after remapping the bmxp.FMDATA / bmxp.IMDATA dictionaries (e.g. Compound_ID as the feature index, injection_id as the injection index).
- Every combined feature traces back to member features in the contributing datasets, pairwise consistent within each clique (diameter=1).
- Pre-alignment validation confirms each dataset was loaded as merged Feature Metadata + Feature Abundances, the format Blueshift and Eclipse require.

## Limitations

- Strict clique mode (diameter=1) requires pairwise agreement among clique members, so borderline feature pairs may be left unmatched rather than forced into the alignment.
- Annotation mismatches can produce spurious aligned rows (3 in the DS1–DS4 run), so annotation-based review of the output remains necessary.
- bmxp provides no version numbers for its modules, complicating reproducibility pinning, and the tools are standalone with no end-to-end pipeline yet.
- Users are expected to already be familiar with Python and LCMS metabolomics data processing; the README states this prerequisite explicitly.
- Reported runtime (~16 s) is hardware-dependent and should be treated as expert review context, not a correctness criterion.

## Evidence

- [abstract] Motivation: combining independently processed datasets requires alignment: "Often, two or more independently processed datasets must be aligned to form a complete dataset, but existing software does not fully meet our needs."
- [abstract] Graph-based matching handles multi-dataset complexity: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [intro] Scope: same-method nontargeted LCMS alignment: "Align two or more same-method nontargeted LCMS datasets"
- [methods] Input format requirement for alignment: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [results] Benchmark result with accuracy quantification: [evidence span withheld — anchor status: not_found]
- [readme] Shared-schema header harmonization across datasets: "All BMXP modules use a shared schema and file formats with our prefered columns headers."

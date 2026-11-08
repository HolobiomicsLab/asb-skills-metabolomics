---
name: annotation-match-concordance-evaluation
description: Use when two or more independently processed, annotated, same-method
  LC-MS datasets have been aligned (e.g. with Eclipse) and you must report how accurately
  overlapping annotated features were matched, including a spurious-mismatch count.
when_to_use_negative:
- The input datasets carry no annotations (no Annotation_ID/Metabolite columns) —
  concordance cannot be computed and alignment must be judged by other criteria
- The datasets were generated with different LC-MS methods — Eclipse is scoped to
  'Align two or more same-method nontargeted LCMS datasets', so concordance measured
  on such a mix conflates method differences with alignment error
- You are evaluating redundant-feature clustering within one dataset (Gravity's role)
  rather than cross-dataset alignment of independently processed runs
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Eclipse
  role: MSAligner produces the multi-dataset aligned feature table whose annotation
    concordance this skill evaluates
  repo: https://github.com/broadinstitute/bmxp
- name: bmxp
  role: Provides the shared schema (bmxp.FMDATA/IMDATA dictionaries, Feature Metadata
    + Feature Abundances merge) that harmonizes column headers so annotations are
    comparable across datasets
  repo: https://github.com/broadinstitute/bmxp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:56dbf0da3762b6a52356a9dfaa1e3787fa3a221672447c0a6a7962d15bb3fa82
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
yes_rate: 0.46153846153846156
n_claims: 13
---

# annotation-match-concordance-evaluation

## Summary

Quantifies the accuracy of an n-dataset LC-MS feature alignment by counting how many features carrying the same annotation (Annotation_ID/Metabolite) across datasets were placed in the same aligned row, versus rows with conflicting annotations ('spurious' mismatches). Used to benchmark Eclipse clique-mode alignment on annotated human plasma datasets.

## When to use

Use when two or more independently processed, annotated, same-method LC-MS datasets have been aligned (e.g. with Eclipse) and you must report how accurately overlapping annotated features were matched, including a spurious-mismatch count.

## When NOT to use

- The input datasets carry no annotations (no Annotation_ID/Metabolite columns) — concordance cannot be computed and alignment must be judged by other criteria
- The datasets were generated with different LC-MS methods — Eclipse is scoped to 'Align two or more same-method nontargeted LCMS datasets', so concordance measured on such a mix conflates method differences with alignment error
- You are evaluating redundant-feature clustering within one dataset (Gravity's role) rather than cross-dataset alignment of independently processed runs

## Inputs

- Aligned combined feature table produced by Eclipse MSAligner over n datasets (default clique mode, diameter=1)
- Per-dataset Feature Metadata tables (bmxp.FMDATA schema: Compound_ID, RT, MZ, Annotation_ID, Adduct, __annotation_id, Metabolite)
- Per-dataset Feature Abundances pivot tables (Compound_ID x injection_id) merged with Feature Metadata
- BMXP shared-schema column-header dictionary objects (bmxp.FMDATA / bmxp.IMDATA) used to harmonize terminology across datasets

## Outputs

- Concordance summary: number of correctly matched overlapping annotated features out of total overlapping annotated features (e.g. 362/381 = 95%)
- Count of spurious annotation-mismatch rows in the aligned table (e.g. 3)
- Tabulated list of combined/aligned features with per-row annotation agreement status

## How to apply

After aligning the datasets, merge each dataset's Feature Metadata (bmxp.FMDATA, indexed on Compound_ID with Annotation_ID, Adduct, Metabolite columns) with its Feature Abundances pivot (Compound_ID x injection_id), since Eclipse requires merged Feature Metadata + Feature Abundances. Run the aligner (MSAligner, default strict clique mode, diameter=1) across all datasets, then take the resulting combined feature table and group rows by annotation identity: features sharing a globally unique annotation identifier across datasets count as correctly matched, while aligned rows whose annotation labels conflict are counted as spurious annotation mismatches. Report the concordance as correctly-matched / total overlapping annotated features (in the DS1–DS4 benchmark this was 362/381 = 95%, with 3 spurious rows and 3861 combined features, i.e. 29% of the smallest dataset DS2). Runtime (reported ~16 s) is hardware-dependent and should be recorded as expert-review context, not as a pass/fail criterion. Treat annotation concordance as an independent validation of the graph-based alignment rather than as a tuning parameter.

## Related tools

- **Eclipse** (MSAligner produces the multi-dataset aligned feature table whose annotation concordance this skill evaluates) — https://github.com/broadinstitute/bmxp
- **bmxp** (Provides the shared schema (bmxp.FMDATA/IMDATA dictionaries, Feature Metadata + Feature Abundances merge) that harmonizes column headers so annotations are comparable across datasets) — https://github.com/broadinstitute/bmxp

## Examples

```
import bmxp; from bxmp.eclipse import MSAligner; aligner = MSAligner(); aligned = aligner.align([ds1, ds2, ds3, ds4])  # default strict clique mode (diameter=1), then count matching/conflicting Annotation_ID per aligned row
```

## Evaluation signals

- Concordance fraction on overlapping annotated features falls in the high-90s range for same-method plasma datasets (benchmark: 95%, 362 of 381 correctly matched)
- Spurious annotation-mismatch rows are counted separately and remain a small absolute number relative to correct matches (benchmark: 3 vs 362)
- Combined aligned feature count is consistent with dataset sizes, expressible as a fraction of the smallest dataset (benchmark: 3861 features = 29% of DS2)
- Every aligned row carries a definite match/mismatch status derived from the globally unique __annotation_id / Annotation_ID fields, not from mz/RT proximity alone
- Runtime is logged but explicitly excluded from pass/fail validation since it is hardware-dependent

## Limitations

- Concordance can only be computed where datasets share annotations; the 381 overlapping annotated features are the denominator, not all 3861 aligned features
- Spurious rows indicate annotation conflicts in the aligned output but the metric does not localize whether the fault is in alignment or in upstream annotation
- Reported runtime (~16 s for DS1–DS4) is hardware-dependent and was treated as expert review context, not a reproducibility criterion
- No version numbers for bmxp or its modules are recorded, complicating exact reproduction

## Evidence

- [abstract] Eclipse graph-based handling of multi-dataset matches (abstract): "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [methods] Requirement to merge Feature Metadata + Feature Abundances before alignment: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Shared schema ensures annotation columns are comparable across datasets: "All BMXP modules use a shared schema and file formats with our prefered columns headers."
- [readme] Annotation fields available in Feature Metadata for concordance matching: "`Annotation_ID` - Method-unique annotation label"

---
name: cross-dataset-match-count-reporting
description: Use when an Eclipse MSAligner one-by-all alignment has been run (e.g.,
  a reference dataset like rat plasma DS5 against comparison datasets DS6–DS9) and
  you must report how many of the reference features matched in all datasets, partially
  matched, or had no match.
when_to_use_negative:
- Datasets were produced by different LC-MS methods — Eclipse aligns 'same-method
  nontargeted LCMS datasets' only
- You need per-feature correspondence tables or annotated feature identities rather
  than aggregate match-status counts
- The alignment has not yet been run — this skill only tallies results; it does not
  perform the alignment itself
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Eclipse
  role: MSAligner performs the one-by-all alignment whose per-feature match outcomes
    are tallied into full/partial/unmatched counts
  repo: https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- name: bmxp
  role: Provides the shared schema (bmxp.FMDATA, Feature Metadata + Feature Abundances
    merge) required as Eclipse input and the Python environment for tallying
  repo: https://github.com/broadinstitute/bmxp
- name: Python
  role: Language used to merge the schema tables, run the aligner, tally match statuses,
    and write alignment_summary.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf290
    title: '<i>Eclipse</i> : a Python package for alignment of two or more nontargeted
      LC-MS metabolomics datasets'
  source_content_hash: sha256:9bc09a62c93bd8efd73623912e470839eb2be7b8a790f438b235c60f905370df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Eclipse
  - Python
  - bmxp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-dataset-match-count-reporting

## Summary

Tally features from an Eclipse one-by-all LC-MS alignment into full matches (present in all datasets), partial matches, and unmatched features, and write the breakdown to a summary CSV. This quantifies cross-dataset feature overlap after alignment of independently processed nontargeted metabolomics datasets.

## When to use

Use when an Eclipse MSAligner one-by-all alignment has been run (e.g., a reference dataset like rat plasma DS5 against comparison datasets DS6–DS9) and you must report how many of the reference features matched in all datasets, partially matched, or had no match.

## When NOT to use

- Datasets were produced by different LC-MS methods — Eclipse aligns 'same-method nontargeted LCMS datasets' only
- You need per-feature correspondence tables or annotated feature identities rather than aggregate match-status counts
- The alignment has not yet been run — this skill only tallies results; it does not perform the alignment itself

## Inputs

- Merged Feature Metadata + Feature Abundances for each dataset (bmxp shared schema: Compound_ID x injection_id pivot)
- Eclipse MSAligner one-by-all alignment results (per-feature match status against each comparison dataset)
- Reference dataset feature count (e.g., 12461 DS5 features)

## Outputs

- alignment_summary.csv with counts of full matches, partial matches, and unmatched reference features
- Count of all-dataset overlapping features by match status (fully vs partially matched)

## How to apply

After executing the one-by-all alignment with bmxp.eclipse.MSAligner — configured per Supplementary Code S3 with min group/clique size 2, diameter 2, and Intensity weighting disabled — tally the reference dataset's features by match status: features present in every comparison dataset are full matches, features matching at least one but not all datasets are partial matches, and features with no match anywhere are unmatched. Separately count the all-dataset overlapping features by match status (e.g., of 140 features overlapping all datasets, how many fully vs partially matched). Write the summary counts (e.g., 1600 full / 3681 partial / 7180 unmatched out of 12461 DS5 features) to alignment_summary.csv. The rationale is that Eclipse's graph-based approach handles complex matching scenarios for n > 2 datasets, so a simple present-in-all tally is insufficient and the full/partial/unmatched trichotomy is the informative decomposition.

## Related tools

- **Eclipse** (MSAligner performs the one-by-all alignment whose per-feature match outcomes are tallied into full/partial/unmatched counts) — https://github.com/broadinstitute/bmxp/blob/main/bmxp/eclipse/readme.md
- **bmxp** (Provides the shared schema (bmxp.FMDATA, Feature Metadata + Feature Abundances merge) required as Eclipse input and the Python environment for tallying) — https://github.com/broadinstitute/bmxp
- **Python** (Language used to merge the schema tables, run the aligner, tally match statuses, and write alignment_summary.csv)

## Examples

```
from bmxp.eclipse import MSAligner; aligner = MSAligner(ds5_merged, ds6_merged, ds7_merged, ds8_merged, ds9_merged, min_group_size=2, diameter=2, intensity_weighting=False)
```

## Evaluation signals

- Reproduced breakdown matches the reported result exactly: 1600 of 12461 DS5 features matched in all datasets, 3681 partial matches, 7180 unmatched
- Sum of full + partial + unmatched equals the total reference feature count (1600 + 3681 + 7180 = 12461)
- All-dataset overlapping-feature check: 130 of 140 overlapping features fully matched with 10 partially matched
- Input tables conform to the bmxp shared schema (Feature Metadata indexed on Compound_ID, Feature Abundances pivoted as Compound_ID x injection_id)

## Limitations

- Eclipse requires merging Feature Metadata + Feature Abundances per dataset before use, per the bmxp shared schema
- Eclipse aligns same-method nontargeted LC-MS datasets; it is not validated for cross-method or targeted/nontargeted mixed data beyond the __extraction_method flag
- Match classification is tripartite (full/partial/unmatched); features failing the configured clique parameters (min group/clique size 2, diameter 2) will be scored unmatched even if marginally similar

## Evidence

- [abstract] Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n > 2 datasets.: "Eclipse uses a novel graph-based approach to handle complex matching scenarios that arise from n  > 2 datasets."
- [readme] Some modules require merging Feature Metadata + Feature Abundances.: "Some modules (Blueshift, Eclipse) require merging Feature Metadata + Feature Abundances."
- [readme] Eclipse aligns same-method nontargeted LCMS datasets.: "Align two or more same-method nontargeted LCMS datasets"
- [methods] MSAligner import for alignment.: "from bxmp.eclipse import MSAligner"

---
name: expert-annotation-aggregation
description: Use when multiple human raters have independently labeled the same items
  (e.g., ion images tagged via TagOff) and you must quantify inter-rater reliability
  before merging tags into a curated gold standard.
when_to_use_negative:
- You only have a single rater's labels and no curator reference — reliability coefficients
  like Cohen's kappa require at least two raters on the same items
- Your goal is to train or evaluate the off-sample classifier (ResNet-50/fastai) rather
  than to validate the human labels themselves
- The items are continuous scores rather than categorical off-sample/on-sample/unknown
  tags — kappa-based agreement as applied here assumes categorical labels
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: sqlite3
  role: Exports the deposited expert tagging records from imageclassification.sqlite
    to CSV with 'sqlite3 -header -csv ... "select * from imageclassifications"'
- name: scikit-learn
  role: Provides cohen_kappa_score to compute pairwise tagger agreement over off-sample/on-sample/unknown
    labels
- name: TagOff
  role: Web app through which multiple independent taggers labeled each ion image
    as off-sample, on-sample, or unknown; produces the tagging records this skill
    aggregates
  repo: https://github.com/metaspace2020/offsample
- name: METASPACE
  role: Source platform whose public datasets provided the ion images tagged by experts;
    its codebase underlies TagOff
  repo: https://github.com/metaspace2020/metaspace/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:6afa99cec35b6988bd9a46be6f9001e62a48c68fdb9901fa1fdc2aea3f0b1f9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/offsample
  tools:
  - METASPACE
  - TagOff
  - scikit-learn
  - sqlite3
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# expert-annotation-aggregation

## Summary

Aggregates independent expert tags of ion images (off-sample/on-sample/unknown) by computing pairwise Cohen's kappa, tagger-vs-curator agreement, and multi-rater Krippendorff's alpha to validate that the aggregated gold standard labels are reliable.

## When to use

Use when multiple human raters have independently labeled the same items (e.g., ion images tagged via TagOff) and you must quantify inter-rater reliability before merging tags into a curated gold standard.

## When NOT to use

- You only have a single rater's labels and no curator reference — reliability coefficients like Cohen's kappa require at least two raters on the same items
- Your goal is to train or evaluate the off-sample classifier (ResNet-50/fastai) rather than to validate the human labels themselves
- The items are continuous scores rather than categorical off-sample/on-sample/unknown tags — kappa-based agreement as applied here assumes categorical labels

## Inputs

- imageclassification.sqlite database of expert tagging records deposited by the TagOff web app
- CSV export of the imageclassifications table (sqlite3 -header -csv output)
- Curated per-image labels from the curator (reference labels)
- Subset definition of the ion images to analyze (e.g., the five most difficult gold standard datasets)

## Outputs

- Pairwise Cohen's kappa matrix per dataset for all tagger pairs (Supplementary Table S1 equivalent)
- Tagger-vs-curator agreement rates and overall Krippendorff's alpha (Supplementary Table S2 equivalent)
- Summary CSV (e.g., tagger_agreement_tables.csv) of agreement statistics

## How to apply

Export the deposited tagging records from the TagOff imageclassification.sqlite database with sqlite3 (sqlite3 -header -csv ... 'select * from imageclassifications' > results.csv), then load the CSV with pandas and subset to the ion images of interest (e.g., the five hardest gold standard datasets). Compute pairwise Cohen's kappa between all tagger pairs per dataset using scikit-learn's cohen_kappa_score over the off-sample/on-sample/unknown labels, and compute tagger-vs-curator agreement rates using the curated labels as reference. Additionally compute Krippendorff's alpha across all raters to quantify multi-rater reliability. Assemble the per-dataset kappa pairs and agreement coefficients into summary tables mirroring Supplementary Tables S1 and S2 (e.g., tagger_agreement_tables.csv). Accept the result if average tagger-curator Cohen's kappa is ~0.97 (range 0.89-1.0) and Krippendorff's alpha is approximately 0.9, confirming high agreement that justifies merging tags into the gold standard.

## Related tools

- **sqlite3** (Exports the deposited expert tagging records from imageclassification.sqlite to CSV with 'sqlite3 -header -csv ... "select * from imageclassifications"')
- **scikit-learn** (Provides cohen_kappa_score to compute pairwise tagger agreement over off-sample/on-sample/unknown labels)
- **TagOff** (Web app through which multiple independent taggers labeled each ion image as off-sample, on-sample, or unknown; produces the tagging records this skill aggregates) — https://github.com/metaspace2020/offsample
- **METASPACE** (Source platform whose public datasets provided the ion images tagged by experts; its codebase underlies TagOff) — https://github.com/metaspace2020/metaspace/

## Examples

```
sqlite3 -header -csv ./metaspace/webapp/imageclassification.sqlite "select * from imageclassifications" > results.csv
```

## Evaluation signals

- Average tagger-curator Cohen's kappa falls in the reported 0.89-1.0 range with average ~0.97
- Average Krippendorff's alpha across all raters is approximately 0.9, consistent with the average Cohen's kappa of 0.89
- Pairwise kappa values are computed over all tagger pairs for each of the five hardest gold standard datasets, matching Supplementary Tables S1 and S2 structure
- The CSV export contains all records of the imageclassifications table (row count matches deposited tagging records) before subsetting

## Limitations

- Cohen's kappa penalizes disagreement asymmetrically depending on label prevalence; the three-class labels (off-sample/on-sample/unknown) include an 'unknown' class whose handling must be consistent across raters
- Krippendorff's alpha summarizing all raters can mask dataset-specific disagreement; per-dataset pairwise tables should be inspected (the five hardest datasets are the relevant stress test)
- Curator labels are treated as reference; this assumes the curator's imaging MS experience justifies the gold standard integration and does not itself validate curator error

## Evidence

- [discussion] Inter-tagger agreement assessment using Cohen's kappa and Krippendorff's alpha: [evidence span withheld — anchor status: not_found]
- [results] The average Krippendorff's alpha is 0.9, similar to the average Cohen's kappa of 0.89: [evidence span withheld — anchor status: not_found]
- [discussion] Tagging of ion images via TagOff: [evidence span withheld — anchor status: not_found]
- [readme] Export command for tagging records: "sqlite3 -header -csv ./metaspace/webapp/imageclassification.sqlite "select * from imageclassifications" > ./metaspace/webapp/dist/results.csv"
- [discussion] Curation of tags: [evidence span withheld — anchor status: not_found]

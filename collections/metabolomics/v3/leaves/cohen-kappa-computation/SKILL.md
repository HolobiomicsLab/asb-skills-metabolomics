---
name: cohen-kappa-computation
description: Use when multiple human raters have independently labelled the same items
  (e.g. ion images as off-sample/on-sample/unknown) and you must report pairwise or
  multi-rater agreement coefficients to justify a gold standard.
when_to_use_negative:
- When only a single rater's labels exist — kappa requires at least two raters over
  the same items
- When the goal is classifier accuracy evaluation against a single gold standard (use
  F1-score/precision-recall instead, as the paper does for the deep learning model)
- When raters labelled different (non-overlapping) sets of ion images — pairwise kappa
  requires shared items
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: sqlite3
  role: Exports the deposited expert tagging records from imageclassification.sqlite
    to CSV for agreement computation
  repo: https://github.com/metaspace2020/offsample
- name: scikit-learn
  role: Provides cohen_kappa_score for pairwise tagger agreement computation
- name: TagOff
  role: Web app through which experts tagged ion images into off-sample/on-sample/unknown
    classes; the tagged records being assessed originate from TagOff
  repo: https://github.com/metaspace2020/offsample
- name: METASPACE
  role: Source of the public datasets whose ion images form the gold standard being
    agreement-checked
  repo: https://github.com/metaspace2020/metaspace/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:71452160dfe9d5065debfa34d1802288cc6e6bb1f13e50abf86e644575fb1b1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/offsample
  tools:
  - METASPACE
  - TagOff
  - scikit-learn
  - sqlite3
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cohen-kappa-computation

## Summary

Quantifies inter-tagger and tagger-vs-curator agreement on off-sample ion image classifications using pairwise Cohen's kappa (scikit-learn cohen_kappa_score) over off-sample/on-sample/unknown labels, complemented by Krippendorff's alpha for multi-rater reliability. Used in OffsampleAI to validate that the 23,238-image gold standard reflects consistent expert judgements.

## When to use

Use when multiple human raters have independently labelled the same items (e.g. ion images as off-sample/on-sample/unknown) and you must report pairwise or multi-rater agreement coefficients to justify a gold standard.

## When NOT to use

- When only a single rater's labels exist — kappa requires at least two raters over the same items
- When the goal is classifier accuracy evaluation against a single gold standard (use F1-score/precision-recall instead, as the paper does for the deep learning model)
- When raters labelled different (non-overlapping) sets of ion images — pairwise kappa requires shared items

## Inputs

- imageclassifications table exported from imageclassification.sqlite (TagOff tagging records, CSV via sqlite3 -header -csv)
- Curator labels for each dataset (curated on-sample/off-sample judgements used as reference)
- Subset definition of datasets to assess (e.g. the five most difficult gold standard datasets)

## Outputs

- Per-dataset pairwise tagger Cohen's kappa table (mirroring Supplementary Table S1)
- Tagger-vs-curator agreement table and average Krippendorff's alpha (mirroring Supplementary Table S2)
- Written summary file tagger_agreement_tables.csv

## How to apply

Export the deposited tagging records from the TagOff-backed imageclassification.sqlite database with sqlite3 in header/CSV mode ('select * from imageclassifications'). Load the export with pandas and subset to the ion images of the datasets of interest (in the paper, the five hardest gold standard datasets). Compute pairwise Cohen's kappa between all tagger pairs per dataset using scikit-learn's cohen_kappa_score over the three labels (off-sample, on-sample, unknown). Separately compute tagger-vs-curator agreement with curated labels as reference and Krippendorff's alpha across all raters to quantify multi-rater reliability. Assemble per-dataset kappa pairs and agreement coefficients into summary tables mirroring Supplementary Tables S1 and S2 (tagger_agreement_tables.csv). Rationale: chance-corrected kappa is needed because raw percent agreement inflates reliability with imbalanced on/off-sample label distributions.

## Related tools

- **sqlite3** (Exports the deposited expert tagging records from imageclassification.sqlite to CSV for agreement computation) — https://github.com/metaspace2020/offsample
- **scikit-learn** (Provides cohen_kappa_score for pairwise tagger agreement computation)
- **TagOff** (Web app through which experts tagged ion images into off-sample/on-sample/unknown classes; the tagged records being assessed originate from TagOff) — https://github.com/metaspace2020/offsample
- **METASPACE** (Source of the public datasets whose ion images form the gold standard being agreement-checked) — https://github.com/metaspace2020/metaspace/

## Examples

```
from sklearn.metrics import cohen_kappa_score; kappa = cohen_kappa_score(tagger_a_labels, tagger_b_labels)
```

## Evaluation signals

- Average tagger-vs-curator Cohen's kappa falls in the reported 0.89–1.0 range with average ~0.97
- Average Krippendorff's alpha across all raters is approximately 0.9, consistent with the average pairwise kappa of ~0.89
- Pairwise kappa matrix covers all tagger pairs per dataset with no missing cells over the off-sample/on-sample/unknown labels
- Exported CSV contains one row per classification record and row counts match the number of tagged ion images in the selected datasets
- Summary tables structurally mirror Supplementary Tables S1 and S2

## Limitations

- Reported statistics were computed only for the five most difficult gold standard datasets, not the full 87-dataset gold standard
- Kappa treats labels as nominal across three classes (off-sample/on-sample/unknown); no ordinal or per-pixel agreement is captured — the approach does not solve segmentation of off-sample pixels
- The GraphQL API reference for downloading ion images to TagOff was left as a placeholder '(link)' in the paper

## Evidence

- [discussion] Inter-tagger agreement via Cohen's kappa and Krippendorff's alpha: [evidence span withheld — anchor status: not_found]
- [results] High inter-tagger and tagger-curator agreement statistics: [evidence span withheld — anchor status: not_found]
- [methods] TagOff export command for tagging records: "sqlite3 -header -csv ./metaspace/webapp/imageclassification.sqlite "select * from imageclassifications""
- [discussion] TagOff tagging classes: [evidence span withheld — anchor status: not_found]
- [readme] README export instructions: "sqlite3 -header -csv ./metaspace/webapp/imageclassification.sqlite "select * from imageclassifications" > ./metaspace/webapp/dist/results.csv"

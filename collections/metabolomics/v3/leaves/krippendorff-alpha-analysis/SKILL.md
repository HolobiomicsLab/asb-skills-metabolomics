---
name: krippendorff-alpha-analysis
description: Use when multiple human raters have independently labeled the same items
  (e.g., ion images as off-sample/on-sample/unknown) and you must report inter-rater
  reliability before curating a gold standard.
when_to_use_negative:
- You need to evaluate classifier performance against the curated gold standard (F1-score)
  rather than human inter-rater reliability.
- There is only a single rater and no curator reference labels, so no agreement coefficient
  can be computed.
- The labels are continuous rather than categorical off-sample/on-sample/unknown tags.
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: sqlite3
  role: 'Exports the deposited expert tagging records from imageclassification.sqlite
    to CSV: sqlite3 -header -csv ... "select * from imageclassifications"'
  repo: https://github.com/metaspace2020/metaspace/
- name: scikit-learn
  role: Provides cohen_kappa_score for computing pairwise tagger agreement over the
    off-sample/on-sample/unknown labels
- name: TagOff
  role: Web app through which experts tagged ion images into off-sample/on-sample/unknown
    classes, producing the tagging records analyzed here
  repo: https://github.com/metaspace2020/offsample
- name: METASPACE
  role: Source of the public datasets from which the gold standard ion images to be
    assessed were drawn
  repo: https://github.com/metaspace2020/metaspace/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-3425-x
    title: 'OffsampleAI: artificial intelligence approach to recognize off-sample
      mass spectrometry images'
  source_content_hash: sha256:c450bd12da1c32930169eaaf0b919abc82ff61a5c738f70bf54e3559a327007a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - TagOff
  - scikit-learn
  - sqlite3
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# krippendorff-alpha-analysis

## Summary

Quantifies multi-rater and pairwise agreement among expert taggers of ion images (off-sample/on-sample/unknown) using Krippendorff's alpha and Cohen's kappa, as done to validate the OffsampleAI gold standard's tagging reliability.

## When to use

Use when multiple human raters have independently labeled the same items (e.g., ion images as off-sample/on-sample/unknown) and you must report inter-rater reliability before curating a gold standard.

## When NOT to use

- You need to evaluate classifier performance against the curated gold standard (F1-score) rather than human inter-rater reliability.
- There is only a single rater and no curator reference labels, so no agreement coefficient can be computed.
- The labels are continuous rather than categorical off-sample/on-sample/unknown tags.

## Inputs

- imageclassification.sqlite database of expert tagging records (exportable via sqlite3 to results.csv)
- Subset definition of the five most difficult gold standard datasets (from the 87-dataset, 23,238-image gold standard)
- Curator labels per ion image (reference annotations)

## Outputs

- Per-dataset pairwise Cohen's kappa matrix between taggers
- Tagger-vs-curator agreement rates (average Cohen's kappa ~0.97)
- Krippendorff's alpha across all raters (~0.9)
- tagger_agreement_tables.csv mirroring Supplementary Tables S1 and S2

## How to apply

Export the deposited tagging records from the imageclassification.sqlite database with sqlite3 (sqlite3 -header -csv 'select * from imageclassifications' > results.csv) and load the CSV with pandas. Subset the records to the ion images of the five hardest gold standard datasets, then compute pairwise Cohen's kappa between all tagger pairs per dataset using scikit-learn's cohen_kappa_score over the off-sample/on-sample/unknown labels. Compute tagger-vs-curator agreement rates (curated labels as reference) and Krippendorff's alpha across all raters to quantify multi-rater reliability. Assemble the per-dataset kappa pairs and agreement coefficients into two summary tables mirroring Supplementary Tables S1 and S2. Accept the reproduction if the average tagger-curator Cohen's kappa falls in the reported 0.89-1.0 range (~0.97) and Krippendorff's alpha is approximately 0.9.

## Related tools

- **sqlite3** (Exports the deposited expert tagging records from imageclassification.sqlite to CSV: sqlite3 -header -csv ... "select * from imageclassifications") — https://github.com/metaspace2020/metaspace/
- **scikit-learn** (Provides cohen_kappa_score for computing pairwise tagger agreement over the off-sample/on-sample/unknown labels)
- **TagOff** (Web app through which experts tagged ion images into off-sample/on-sample/unknown classes, producing the tagging records analyzed here) — https://github.com/metaspace2020/offsample
- **METASPACE** (Source of the public datasets from which the gold standard ion images to be assessed were drawn) — https://github.com/metaspace2020/metaspace/

## Examples

```
sqlite3 -header -csv ./metaspace/webapp/imageclassification.sqlite "select * from imageclassifications" > results.csv
```

## Evaluation signals

- Average tagger-curator Cohen's kappa falls in the reported range 0.89-1.0 with average ~0.97.
- Average Krippendorff's alpha across all raters is approximately 0.9.
- Reproduced tables structurally mirror Supplementary Tables S1 (pairwise kappas) and S2 (agreement coefficients).
- The SQLite export contains all taggers' records for the five hardest gold standard datasets before subsetting.
- All kappa computations use the three-class off-sample/on-sample/unknown label space via cohen_kappa_score.

## Limitations

- Agreement statistics only cover the five most difficult gold standard datasets, so values may not generalize to all 87 datasets.
- The 'unknown' class is included in the label space; treating it as a valid category vs. excluding it changes kappa values.
- High inter-tagger agreement does not resolve pixel-level off-sample segmentation; the approach does not find off-sample pixels.

## Evidence

- [discussion] Inter-tagger agreement via Cohen's kappa and Krippendorff's alpha: [evidence span withheld — anchor status: not_found]
- [discussion] Tagging via TagOff web app into three classes: [evidence span withheld — anchor status: not_found]
- [discussion] Curation of tags by an experienced curator: [evidence span withheld — anchor status: not_found]
- [methods] Export of tagging records from the SQLite database: "sqlite3 -header -csv ./metaspace/webapp/imageclassification.sqlite "select * from imageclassifications""
- [abstract] Gold standard size and provenance: "we created a high-quality gold standard of 23,238 expert-tagged ion images from 87 public datasets from the METASPACE knowledge base"

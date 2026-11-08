---
name: classifier-performance-comparison
description: Use when two MS/MS spectra classification approaches must be compared
  on the same ground-truth test set, e.g. reproducing the AnnoMe-vs-SIRIUS prenylated
  flavonoid benchmark from Section 3.2/Table ST2.
when_to_use_negative:
- The test spectra have no ground-truth labels for the compound class of interest,
  so 'correct classification' fractions cannot be computed.
- The goal is de novo molecular formula or structure elucidation rather than comparing
  compound-class assignment performance.
- The training dataset is highly imbalanced, a failure mode AnnoMe explicitly cannot
  resolve, making a headline accuracy comparison misleading.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS
  role: Baseline compound-class predictor (version 6.2.2, ClassyFire 'most specific
    class') run on every test spectrum for the comparison
- name: AnnoMe
  role: Provides the custom-trained classifiers (LDA, NN, SVM majority vote over MS2DeepScore
    embeddings) and the demonstration test dataset being benchmarked
  repo: https://github.com/chrboku/AnnoMe
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:cef97787534e91eb882ffcc7ba139a8b2cb5fa2ba42f454c32687f69312dbe42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - SIRIUS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# classifier-performance-comparison

## Summary

Benchmark the classification performance of a custom-trained spectral classifier (e.g. AnnoMe) against an established tool (e.g. SIRIUS compound-class prediction) on an identical ground-truth HR-MS/MS test dataset, reporting per-tool fractions of correctly classified spectra.

## When to use

Use when two MS/MS spectra classification approaches must be compared on the same ground-truth test set, e.g. reproducing the AnnoMe-vs-SIRIUS prenylated flavonoid benchmark from Section 3.2/Table ST2.

## When NOT to use

- The test spectra have no ground-truth labels for the compound class of interest, so 'correct classification' fractions cannot be computed.
- The goal is de novo molecular formula or structure elucidation rather than comparing compound-class assignment performance.
- The training dataset is highly imbalanced, a failure mode AnnoMe explicitly cannot resolve, making a headline accuracy comparison misleading.

## Inputs

- Prenylated-flavonoid demonstration test dataset of HR-MS/MS spectra (MGF) from the AnnoMe repository
- SIRIUS 6.2.2 ClassyFire compound-class predictions per spectrum ('most specific class')
- AnnoMe majority-vote classification results on the same spectra (average correct classification rate)

## Outputs

- annome_vs_sirius_comparison.csv: two-column table of prenylated-flavonoid classification fractions (AnnoMe vs SIRIUS)
- Per-spectrum class-assignment tabulation from parsed SIRIUS output

## How to apply

Obtain the prenylated-flavonoid demonstration test dataset (MS/MS spectra) from the AnnoMe repository per Instructions_DemoPrenylatedFlavonoids.md. Run SIRIUS 6.2.2 on every test spectrum with ClassyFire compound-class prediction enabled and take the 'most specific class' annotation per spectrum; parse the output and compute the fraction of spectra whose most specific class corresponds to prenylated flavonoids. Retrieve the AnnoMe classifiers' average correct-classification rate on the identical dataset (trained via LDA/NN/SVM with cross-validation, different random seeds, and majority vote over MS2DeepScore embeddings). Assemble a two-column comparison (AnnoMe vs SIRIUS) of prenylated-flavonoid classification fractions and write it to annome_vs_sirius_comparison.csv.

## Related tools

- **SIRIUS** (Baseline compound-class predictor (version 6.2.2, ClassyFire 'most specific class') run on every test spectrum for the comparison)
- **AnnoMe** (Provides the custom-trained classifiers (LDA, NN, SVM majority vote over MS2DeepScore embeddings) and the demonstration test dataset being benchmarked) — https://github.com/chrboku/AnnoMe

## Examples

```
sirius --input test_spectra.mgf --project annome_benchmark --canopus  # then tabulate 'most specific class' per spectrum vs AnnoMe rates into annome_vs_sirius_comparison.csv
```

## Evaluation signals

- Reproduced SIRIUS prenylated-flavonoid fraction matches the reported ~41% of spectra predicted as prenylated flavonoids.
- AnnoMe reference rate on the identical dataset matches the reported average of 77% correct classifications across the two test sets.
- Both fractions are computed on the identical test spectra (same dataset, per-spectrum assignments tabulated), so the comparison is like-for-like.
- Large deviation from the reported values (Section 3.2, Table ST2) flags a QC failure requiring expert review.
- Output file annome_vs_sirius_comparison.csv contains exactly two comparable fraction columns (AnnoMe vs SIRIUS).

## Limitations

- Classification of MS/MS spectra into substance classes most often will not be successful, especially with highly imbalanced training datasets; AnnoMe cannot resolve this issue.
- Users must exercise caution with assignments of MS/MS spectra to specific compound classes of interest.
- The AnnoMe 77% figure is an average across two test sets, not a single-set value; per-set variation is not captured by the summary comparison.

## Evidence

- [methods] AnnoMe vs SIRIUS benchmark: [evidence span withheld — anchor status: not_found]
- [results] reported performance gap: [evidence span withheld — anchor status: not_found]
- [readme] classifier methodology: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds. The results are then aggregated and a majority-vote is derived"
- [discussion] class-assignment caution: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"

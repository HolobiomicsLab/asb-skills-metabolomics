---
name: prenylated-flavonoid-spectral-benchmarking
description: Use when a ground-truth prenylated flavonoid MS/MS test set is available
  and you need a head-to-head comparison of AnnoMe's custom classifiers versus SIRIUS
  compound-class predictions.
when_to_use_negative:
- The goal is discovering putative prenylated flavonoids in new plant extracts (e.g.
  Paulownia tomentosa fruits or Glycyrrhizza uralensis roots) rather than benchmarking
  classification performance against a ground-truth test set
- Classifiers have not yet been trained on curated 'relevant'/'other' reference spectra
  — train them first via the AnnoMe classification workflow before any benchmark
- The comparison target is not compound-class assignment but, e.g., molecular formula
  or de novo structure prediction by SIRIUS
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIRIUS
  role: Baseline tool run on every test spectrum with ClassyFire compound-class prediction
    enabled; the 'most specific class' annotation defines the comparison benchmark
- name: AnnoMe
  role: Provides the demonstration test dataset, the custom-trained classifiers (LDA,
    NN, SVM with majority vote on MS2DeepScore embeddings), and the reference 77%
    classification rate
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates the structure-informed embeddings that feed AnnoMe's classifiers
    used in the benchmark comparison
- name: uv
  role: Python toolkit used to install and run AnnoMe commands for obtaining the demo
    dataset and executing the classification workflow
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:c7579664027f37a4db17e1c3a096676d45417cb26ea050ee92029212e38cc077
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - SIRIUS
  - uv
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# prenylated-flavonoid-spectral-benchmarking

## Summary

Benchmarks AnnoMe's user-trained HR-MS/MS classifiers against SIRIUS 6.2.2 ClassyFire 'most specific class' predictions on ground-truth prenylated flavonoid test spectra, quantifying the fraction each tool assigns to prenylated flavonoids.

## When to use

Use when a ground-truth prenylated flavonoid MS/MS test set is available and you need a head-to-head comparison of AnnoMe's custom classifiers versus SIRIUS compound-class predictions.

## When NOT to use

- The goal is discovering putative prenylated flavonoids in new plant extracts (e.g. Paulownia tomentosa fruits or Glycyrrhizza uralensis roots) rather than benchmarking classification performance against a ground-truth test set
- Classifiers have not yet been trained on curated 'relevant'/'other' reference spectra — train them first via the AnnoMe classification workflow before any benchmark
- The comparison target is not compound-class assignment but, e.g., molecular formula or de novo structure prediction by SIRIUS

## Inputs

- Prenylated-flavonoid demonstration test dataset of HR-MS/MS spectra from the AnnoMe repository (per Instructions_DemoPrenylatedFlavonoids.md)
- SIRIUS 6.2.2 ClassyFire output with 'most specific class' annotation per spectrum
- AnnoMe classifiers' average correct classification rate on the same test sets (reference ~77%, Section 3.2, Table ST2)

## Outputs

- annome_vs_sirius_comparison.csv with two columns of prenylated-flavonoid classification fractions (AnnoMe vs SIRIUS)
- Per-spectrum SIRIUS ClassyFire 'most specific class' assignments tabulated with fraction predicted as prenylated flavonoids

## How to apply

Obtain the prenylated-flavonoid demonstration test dataset (MS/MS spectra) from the AnnoMe repository following Instructions_DemoPrenylatedFlavonoids.md, then run SIRIUS 6.2.2 on every test spectrum with ClassyFire compound-class prediction enabled, taking the 'most specific class' annotation per spectrum. Parse the SIRIUS output, tabulate per-spectrum class assignments, and compute the fraction of spectra whose most specific class corresponds to prenylated flavonoids. Retrieve the AnnoMe classifiers' average correct prenylated-flavonoid classification rate on the identical dataset — the AnnoMe pipeline trains LDA, NN, and SVM classifiers with cross-validation and different random seeds and aggregates them by majority vote on MS2DeepScore embeddings. Assemble a two-column comparison (AnnoMe vs SIRIUS) of prenylated-flavonoid classification fractions and write it to annome_vs_sirius_comparison.csv.

## Related tools

- **SIRIUS** (Baseline tool run on every test spectrum with ClassyFire compound-class prediction enabled; the 'most specific class' annotation defines the comparison benchmark)
- **AnnoMe** (Provides the demonstration test dataset, the custom-trained classifiers (LDA, NN, SVM with majority vote on MS2DeepScore embeddings), and the reference 77% classification rate) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates the structure-informed embeddings that feed AnnoMe's classifiers used in the benchmark comparison)
- **uv** (Python toolkit used to install and run AnnoMe commands for obtaining the demo dataset and executing the classification workflow) — https://github.com/astral-sh/uv

## Examples

```
git clone https://github.com/chrboku/AnnoMe && cd AnnoMe && uv run annome_classificationgui  # then run SIRIUS 6.2.2 per test spectrum and write annome_vs_sirius_comparison.csv
```

## Evaluation signals

- Reproduced SIRIUS prenylated-flavonoid fraction should match the reported ~41% of spectra predicted as prenylated flavonoids
- AnnoMe reference rate on the identical dataset should be ~77% average across the two test sets (Section 3.2, Table ST2)
- Comparison CSV annome_vs_sirius_comparison.csv must contain exactly two columns (AnnoMe vs SIRIUS) of classification fractions computed on the same spectra
- Per-spectrum SIRIUS table must cover every test spectrum, with each row carrying exactly one 'most specific class' annotation
- Any large deviation from the reported 41% (SIRIUS) or 77% (AnnoMe) values flags a QC failure requiring expert review

## Limitations

- SIRIUS predicted only 41% of the ground-truth prenylated flavonoid spectra as prenylated flavonoids, substantially underperforming AnnoMe's ~77% average
- Classification of MS/MS spectra into substance classes most often fails, especially with highly imbalanced training datasets, and AnnoMe is not capable of resolving this failure mode
- Users must exercise caution with the generated results (assignment of MS/MS spectra to specific compound classes)
- MS2DeepScore embedding generation is the most time-consuming step (~4 hours on a standard laptop; not feasible on a passively cooled MacBook Air with 16GB RAM), so benchmark reproduction depends on cached embeddings

## Evidence

- [results] AnnoMe classifiers outperform SIRIUS on ground-truth test spectra: [evidence span withheld — anchor status: not_found]
- [methods] AnnoMe classifier architecture: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [readme] Majority-vote aggregation defines final classification: "The results are then aggregated and a majority-vote is derived which indicates the final classification of the input MS/MS spectra to either "relevant" or "other"."
- [readme] MS2DeepScore embeddings are the computational bottleneck: "The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop"
- [discussion] Caution with compound-class assignments: "the users are herewith informed to exercise caution with the generated results (i.e., the assignment of MS/MS spectra to specific compound classes of interest)."

---
name: sirius-classyfire-class-annotation
description: Use when ground-truth-labeled MS/MS spectra (e.g., prenylated flavonoid
  test sets from the AnnoMe repo) need an independent compound-class baseline from
  SIRIUS ClassyFire for comparison against user-defined classifiers.
when_to_use_negative:
- When the goal is to run AnnoMe's own classification (via `uv run annome_classificationgui`)
  rather than to produce an independent SIRIUS baseline for benchmarking
- When spectra already carry trusted compound-class annotations, making an independent
  ClassyFire prediction redundant
- When a SIRIUS version other than 6.2.2 must be used, since the reported 41% baseline
  is tied to that specific version
tools:
- name: SIRIUS
  role: Executed (version 6.2.2) on every test spectrum with ClassyFire compound-class
    prediction enabled; the 'most specific class' output is the annotation baseline
- name: AnnoMe
  role: Supplies the prenylated-flavonoid demonstration test dataset (via Instructions_DemoPrenylatedFlavonoids.md)
    and the reference 77% classification rate used for comparison
  repo: https://github.com/chrboku/AnnoMe
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:c0aa3dab09881c00e70dbd5ce3fe719b51873660e0de6be104f2af71da2963ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - SIRIUS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# sirius-classyfire-class-annotation

## Summary

Annotates each HR-MS/MS spectrum in a test dataset with a compound class by running SIRIUS 6.2.2 with ClassyFire prediction enabled and taking the 'most specific class' per spectrum, so the resulting fraction of class-correct spectra can be benchmarked against AnnoMe classifiers (e.g., 41% vs 77% for prenylated flavonoids).

## When to use

Use when ground-truth-labeled MS/MS spectra (e.g., prenylated flavonoid test sets from the AnnoMe repo) need an independent compound-class baseline from SIRIUS ClassyFire for comparison against user-defined classifiers.

## When NOT to use

- When the goal is to run AnnoMe's own classification (via `uv run annome_classificationgui`) rather than to produce an independent SIRIUS baseline for benchmarking
- When spectra already carry trusted compound-class annotations, making an independent ClassyFire prediction redundant
- When a SIRIUS version other than 6.2.2 must be used, since the reported 41% baseline is tied to that specific version

## Inputs

- Prenylated-flavonoid demonstration test dataset of HR-MS/MS spectra from the AnnoMe repository (per Instructions_DemoPrenylatedFlavonoids.md)
- SIRIUS 6.2.2 with ClassyFire compound-class prediction enabled
- AnnoMe classifiers' reference correct-classification rate on the same dataset (77%, Section 3.2, Table ST2)

## Outputs

- Per-spectrum SIRIUS 'most specific class' assignments (tabulated fraction classified as prenylated flavonoids)
- annome_vs_sirius_comparison.csv with two columns comparing AnnoMe vs SIRIUS prenylated-flavonoid classification fractions

## How to apply

Obtain the demonstration test dataset (MS/MS spectra) from the AnnoMe repository per Instructions_DemoPrenylatedFlavonoids.md, then run SIRIUS version 6.2.2 on every test spectrum with ClassyFire compound-class prediction enabled. For each spectrum, parse the SIRIUS output and record only the 'most specific class' annotation. Tabulate the per-spectrum class assignments and compute the fraction whose most specific class corresponds to prenylated flavonoids. Retrieve the AnnoMe classifiers' average correct classification rate (77% across the two test sets) on the identical dataset and assemble a two-column comparison (AnnoMe vs SIRIUS) written to annome_vs_sirius_comparison.csv.

## Related tools

- **SIRIUS** (Executed (version 6.2.2) on every test spectrum with ClassyFire compound-class prediction enabled; the 'most specific class' output is the annotation baseline)
- **AnnoMe** (Supplies the prenylated-flavonoid demonstration test dataset (via Instructions_DemoPrenylatedFlavonoids.md) and the reference 77% classification rate used for comparison) — https://github.com/chrboku/AnnoMe

## Evaluation signals

- The reproduced SIRIUS prenylated-flavonoid fraction matches the reported ~41% (Section 3.2, Table ST2)
- The AnnoMe reference rate on the identical dataset is ~77% average across the two test sets
- annome_vs_sirius_comparison.csv contains exactly two columns (AnnoMe vs SIRIUS) of prenylated-flavonoid classification fractions
- Any large deviation between reproduced and reported fractions flags a QC failure requiring expert review
- Every test spectrum has exactly one 'most specific class' annotation parsed from the SIRIUS output

## Limitations

- The result is version-sensitive: the 41% baseline was produced with SIRIUS 6.2.2; other versions may not reproduce it
- Classification of MS/MS spectra into substance classes is non-trivial and often unsuccessful; the paper warns users to exercise caution with compound-class assignments
- Only the 'most specific class' ClassyFire level is used per spectrum; higher-level or multi-level class information is not considered

## Evidence

- [other] SIRIUS most-specific-class run over test spectra: [evidence span withheld — anchor status: not_found]
- [results] Reported benchmark result: [evidence span withheld — anchor status: not_found]
- [other] Validation threshold: [evidence span withheld — anchor status: not_found]
- [readme] General caution on class assignment: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"

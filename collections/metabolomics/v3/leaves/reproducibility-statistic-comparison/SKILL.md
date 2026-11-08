---
name: reproducibility-statistic-comparison
description: Use when reproducing cross-laboratory lipid identification overlap statistics
  from a lab-by-lipid identification matrix, e.g. counting how many of 31 laboratories
  reported each SRM 1950 lipid and validating the ≥5-lab subset against a published
  value.
when_to_use_negative:
- Input is raw MS/MS data or individual spectra requiring lipid identification (use
  a spectrum-to-lipid identification method such as LipiDetective instead of overlap
  statistics)
- The identifications come from a single laboratory or single software run, so no
  cross-laboratory overlap statistic can be computed
- The lipid names are already harmonized and only downstream statistics other than
  lab-coverage overlap (e.g. quantification agreement) are needed
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Tabular computation of per-lipid reporting-laboratory counts from the lab-by-lipid
    identification matrix and filtering of lipids reported by ≥5 laboratories
- name: LipiDetective
  role: Source of the harmonized lipid identification results from the SRM 1950 multi-laboratory
    study that feed the overlap computation
  repo: https://github.com/LipiTUM/lipidetective
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:85647414562a20fff65fd296ad147318c99ec3c6aadc5542aaf4ab2404d0b2fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# reproducibility-statistic-comparison

## Summary

Reproduces cross-laboratory reproducibility statistics for lipid identifications by computing per-lipid reporting-laboratory counts from a lab-by-lipid identification matrix and validating the overlap against a published reference figure (e.g. 339 of 1527 SRM 1950 sum-composition lipids reported by ≥5 of 31 laboratories).

## When to use

Use when reproducing cross-laboratory lipid identification overlap statistics from a lab-by-lipid identification matrix, e.g. counting how many of 31 laboratories reported each SRM 1950 lipid and validating the ≥5-lab subset against a published value.

## When NOT to use

- Input is raw MS/MS data or individual spectra requiring lipid identification (use a spectrum-to-lipid identification method such as LipiDetective instead of overlap statistics)
- The identifications come from a single laboratory or single software run, so no cross-laboratory overlap statistic can be computed
- The lipid names are already harmonized and only downstream statistics other than lab-coverage overlap (e.g. quantification agreement) are needed

## Inputs

- Published multi-laboratory lipid identification results (lab-by-lipid identification matrix, e.g. SRM 1950 measurements from 31 laboratories)
- Sum-composition lipid species labels from heterogeneous data sources with varying nomenclatures
- Published reference statistic to validate against (e.g. 339 lipids reported by ≥5 laboratories)

## Outputs

- Per-lipid count of reporting laboratories (lab-coverage vector)
- Filtered cross-lab overlap set of lipids reported by ≥5 laboratories (expected 339 for SRM 1950)
- Lab-coverage distribution summary for reproducibility reporting

## How to apply

Load the published multi-laboratory identification results — in the SRM 1950 case, 1527 unique lipids on a sum composition level reported by 31 diverse laboratories. Harmonize lipid names by manually adjusting the varying nomenclatures from the different data sources to match the updated lipid shorthand notation, since sum-composition labels differ across labs and software. Build or load the lab-by-lipid identification matrix and use pandas to compute the per-lipid count of reporting laboratories (e.g. via groupby/nunique over the lab column). Filter lipids reported by at least 5 laboratories to obtain the cross-lab overlap set and record the full lab-coverage distribution. Validate by comparing the reproduced overlap count against the published reference figure of 339; any deviation indicates a harmonization error in name normalization rather than a computational mistake.

## Related tools

- **pandas** (Tabular computation of per-lipid reporting-laboratory counts from the lab-by-lipid identification matrix and filtering of lipids reported by ≥5 laboratories)
- **LipiDetective** (Source of the harmonized lipid identification results from the SRM 1950 multi-laboratory study that feed the overlap computation) — https://github.com/LipiTUM/lipidetective

## Examples

```
import pandas as pd; counts = df.groupby('lipid')['lab'].nunique(); overlap = counts[counts >= 5]; assert len(overlap) == 339
```

## Evaluation signals

- The reproduced count of lipids reported by ≥5 laboratories must exactly match the published value of 339 for SRM 1950; any deviation indicates a harmonization error in name normalization
- The total set of unique sum-composition lipids across all laboratories must equal 1527 before filtering
- Every lipid name in the matrix must conform to the updated lipid shorthand notation after harmonization (no legacy or source-specific nomenclature strings remain)
- Per-lipid lab counts must be integers in the range 1–31, consistent with 31 participating laboratories
- The recorded lab-coverage distribution must be monotone-consistent: the ≥5-lab subset is a subset of the full 1527-lipid set

## Limitations

- Nomenclature harmonization across data sources was performed manually, so name normalization is the dominant error source when reproducing the overlap statistic
- It is not yet common practice in lipidomics to supply raw data with a publication, and identifications are often hard to match back to exact spectra because spectrum IDs are rarely provided and metadata may differ due to preprocessing
- The threshold of ≥5 reporting laboratories is a fixed convention from the published study; changing it changes the reported overlap count and invalidates direct comparison to the reference figure

## Evidence

- [intro] the human plasma standard reference material (SRM) 1950 was measured by 31 diverse laboratories: "the human plasma standard reference material (SRM) 1950 was measured by 31 diverse laboratories"
- [methods] manual nomenclature harmonization: "The varying nomenclatures in the different data sources were manually adjusted to match the updated lipid shorthand notation"
- [intro] raw data availability caveat: "it is not yet common practice in lipidomics to supply the raw data with a publication. Even if the raw data is available, it is often not possible to match the identifications back to the exact"
- [readme] LipiDetective scope: "LipiDetective is a deep learning framework designed for the identification of molecular lipid species from tandem mass spectra."

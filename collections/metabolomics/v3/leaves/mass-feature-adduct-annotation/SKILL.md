---
name: mass-feature-adduct-annotation
description: Use when a metabolomics dataset contains unlabeled m/z mass features
  that must be assigned candidate metabolite structures before reaction-rule-based
  pathway prediction with MEANtools.
when_to_use_negative:
- The user already has MS/MS data and wants spectral-library or molecular-networking-based
  annotation — the article notes MEANtools does not currently support loading MS/MS
  data or library/networking annotation approaches
- Compounds are not represented in LOTUS and no user-defined metabolite structure
  database is available as an alternative
- Stereoisomer discrimination is required — mass spectrometric data does not capture
  stereoisomers, so adduct-based mass matching cannot resolve them
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MEANtools
  role: Pipeline performing adduct enumeration and 20 ppm candidate neutral mass computation
    for each mass feature
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: LOTUS
  role: Structure database queried by candidate neutral mass to assign molecular structures
    to mass features
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:64872c900199d0a9961cfdd54f0ec3171aaef559e4b57798d22bfd4ff4fa55b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - ClusterONE
  - LOTUS
  - MEANtools
  - RetroRules
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# mass-feature-adduct-annotation

## Summary

Assign putative molecular structures to metabolome mass features by enumerating candidate adducts (~40 by default) and querying the LOTUS structure database (SQLite release of 10/10/2023) within a default 20 ppm mass accuracy tolerance, producing an adduct-to-structure lookup table used downstream by MEANtools.

## When to use

Use when a metabolomics dataset contains unlabeled m/z mass features that must be assigned candidate metabolite structures before reaction-rule-based pathway prediction with MEANtools.

## When NOT to use

- The user already has MS/MS data and wants spectral-library or molecular-networking-based annotation — the article notes MEANtools does not currently support loading MS/MS data or library/networking annotation approaches
- Compounds are not represented in LOTUS and no user-defined metabolite structure database is available as an alternative
- Stereoisomer discrimination is required — mass spectrometric data does not capture stereoisomers, so adduct-based mass matching cannot resolve them

## Inputs

- Untargeted metabolome mass features (m/z values) from a paired transcriptomic-metabolomic dataset (e.g., the Jeon 2020 tomato dataset)
- LOTUS structure database converted to SQLite format (10/10/2023 release, distributed in the MEANtools GitHub repository)

## Outputs

- adduct_structure_lookup.csv mapping each mass feature to candidate adducts and LOTUS structures
- Structure-annotated mass features ready for MAD rescaling and mutual-rank correlation with transcripts

## How to apply

For each mass feature in the metabolome, enumerate the default adduct list of roughly 40 adducts and compute candidate neutral masses at the default 20 ppm mass accuracy tolerance, which the authors chose to balance sensitivity and specificity. Query the LOTUS structure database — converted to SQLite format (10/10/2023 release) for MEANtools compatibility — with the candidate neutral masses to assign molecular structures to each mass feature. Write the resulting assignments to an adduct-to-structure lookup table (adduct_structure_lookup.csv) that later MEANtools steps use to connect annotated metabolites to correlated transcripts. Note that the mass spectrometer's resolution directly impacts the precision of detected m/z values and thus the confidence of adduct-based matching, so confirm instrument resolution is adequate before trusting borderline matches.

## Related tools

- **MEANtools** (Pipeline performing adduct enumeration and 20 ppm candidate neutral mass computation for each mass feature) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **LOTUS** (Structure database queried by candidate neutral mass to assign molecular structures to mass features) — https://github.com/IntegrativeBioinformaticsLab/meantools

## Evaluation signals

- Every mass feature annotated within tolerance should yield a non-empty adduct-to-structure assignment; verify lookups return structures for features matched within 20 ppm
- The SQLite LOTUS database must open and return structure records for the candidate neutral masses computed from the ~40-adduct list
- Mass differences between observed m/z and assigned candidate neutral masses must fall within the 20 ppm tolerance (approximately ±0.002% of m/z)
- Downstream reaction anticipation (e.g., Solyc12g100250.1 at correlation 0.744, p = 1.106e-10 for crepenynic acid to dehydrocrepenynic acid) should only run on features that received structure assignments

## Limitations

- Mass spectrometer resolution directly impacts the precision of detected m/z values and the confidence of adduct-based matching
- Mass spectrometric data does not capture stereoisomers, so structures assigned by mass matching may be stereoisomer-ambiguous
- Annotating mass signatures can be improved by loading MS/MS data and using mass spectral library and networking-based annotation approaches, which MEANtools does not currently support
- A user-defined metabolite database can substitute for LOTUS, but only if it is mass-searchable in the same manner

## Evidence

- [results] Enumerate adducts and query LOTUS at 20 ppm tolerance: [evidence span withheld — anchor status: not_found]
- [discussion] 20 ppm balances sensitivity and specificity: "By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity."
- [discussion] Adduct identification and LOTUS query assign structures to mass features: "it predicts possible metabolites and their corresponding molecular structures for each mass feature by identifying possible adducts and querying the LOTUS database"
- [discussion] Mass spectrometer resolution impacts adduct matching confidence: "the resolution of the mass spectrometer directly impacts the precision of detected m/z values"
- [discussion] LOTUS SQLite made available in the MEANtools GitHub repository: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."

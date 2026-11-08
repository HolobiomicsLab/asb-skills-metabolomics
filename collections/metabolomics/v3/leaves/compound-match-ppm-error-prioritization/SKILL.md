---
name: compound-match-ppm-error-prioritization
description: Use when you have a short list of statistically significant m/z values
  (e.g. t-test hits at 264, 308, 441, 561 m/z) and must rank candidate compound matches
  from a built database such as HMDB by isotope/adduct status and ppm error.
when_to_use_negative:
- You only have raw peak data — MetaboShiny does not accept raw peaks; pre-processed
  m/z peaklists (XCMS/MetaboAnalyst export, MSnbase, MetaboLights format) are required.
- You want to annotate every m/z in the dataset — MetaboShiny performs annotation
  only on significant hits resulting from the statistical analysis, not on all m/z
  ratios.
- You need fully confirmed MSI level 1 identifications from MS/MS spectra alone —
  this skill yields putative annotations whose level 1 status depends on the reference
  study, not on the matching procedure itself.
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MetaboShiny
  role: Performs the m/z-vs-database matching, prioritizes compounds by isotope and
    adduct status, and sorts matches by ppm error within the user-set ppm error margin
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Builds the local compound database (e.g. HMDB), pre-generating m/z values
    for common isotopes and adducts of every compound that matching queries hit
  repo: https://github.com/joannawolthuis/MetaDBparse
- name: R
  role: Language in which MetaboShiny and MetaDBparse are implemented and run (R >=
    3.5 via install.metshi.R or the Docker image)
  repo: https://github.com/joannawolthuis/MetaboShiny
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:a4f44bee649a2e86f4bee993323b3dee8c1d36883f9ae770b91a8703b2ad9866
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Compound-match ppm-error prioritization

## Summary

Prioritize and rank putative compound matches for significant m/z hits by first filtering on main-peak isotopes and then sorting by ppm error within a user-defined mass-spectrometer error margin, collapsing compounds with identical SMILES into single annotation rows.

## When to use

Use when you have a short list of statistically significant m/z values (e.g. t-test hits at 264, 308, 441, 561 m/z) and must rank candidate compound matches from a built database such as HMDB by isotope/adduct status and ppm error.

## When NOT to use

- You only have raw peak data — MetaboShiny does not accept raw peaks; pre-processed m/z peaklists (XCMS/MetaboAnalyst export, MSnbase, MetaboLights format) are required.
- You want to annotate every m/z in the dataset — MetaboShiny performs annotation only on significant hits resulting from the statistical analysis, not on all m/z ratios.
- You need fully confirmed MSI level 1 identifications from MS/MS spectra alone — this skill yields putative annotations whose level 1 status depends on the reference study, not on the matching procedure itself.

## Inputs

- List of significant m/z values from a statistical analysis (e.g. t-test hits at 264, 308, 441, 561 m/z)
- Locally built compound database (e.g. HMDB) pre-generated with isotope and adduct m/z values via MetaDBparse
- User-defined mass-spectrometer error margin in parts per million (ppm)
- Adduct rule table and isotope scoring settings (e.g. M-score with default 2% intensity imprecision)

## Outputs

- Ranked compound match list per m/z value, filtered by main-peak isotopes and sorted by ppm error
- Annotations collapsed so compounds with identical SMILES but different names occupy one row
- MSI confidence level per annotation (level 1 confirmed vs. level 2 putatively annotated)
- annotation_counts.csv with exact row counts per MSI level (4 at MSI level 1, 264 at the next level)

## How to apply

Match each significant m/z value against a locally built compound database (e.g. HMDB prepared via MetaDBparse, which pre-generates m/z values for common isotopes and over twenty adducts) using the error margin in parts per million set for your mass spectrometer. Filter the returned matches first by main-peak isotopes, then sort the survivors by ppm error so the closest-matching compounds rise to the top. Collapse compounds that share identical SMILES but differ only in name into one row before assigning Metabolomics Standards Initiative confidence levels (level 1 confirmed vs. level 2 putatively annotated). MetaboShiny guides this prioritization by scoring compounds on isotope and adduct status; in the reproduced lung cancer urine study this surfaced creatine riboside as the M + H adduct of 264 m/z.

## Related tools

- **MetaboShiny** (Performs the m/z-vs-database matching, prioritizes compounds by isotope and adduct status, and sorts matches by ppm error within the user-set ppm error margin) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Builds the local compound database (e.g. HMDB), pre-generating m/z values for common isotopes and adducts of every compound that matching queries hit) — https://github.com/joannawolthuis/MetaDBparse
- **R** (Language in which MetaboShiny and MetaDBparse are implemented and run (R >= 3.5 via install.metshi.R or the Docker image)) — https://github.com/joannawolthuis/MetaboShiny

## Examples

```
Rscript -e "library(MetaboShiny); start_metshi(inBrowser=T)"  # then set ppm error margin, build HMDB via MetaDBparse, and match significant m/z hits
```

## Evaluation signals

- annotation_counts.csv contains exactly the reported tallies: 4 m/z values at MSI level 1 and 264 at the next annotation level.
- All returned matches fall within the user-defined ppm error margin set at file-import time.
- Match lists are ordered by ppm error only after filtering to main-peak isotopes — check ordering invariants.
- Compounds sharing identical SMILES but different names appear as a single row, not duplicated entries.
- For the 264 m/z hit, the top-ranked match is creatine riboside as the M + H adduct, matching the HMDB search result.

## Limitations

- MetaboShiny does not automatically update databases; a database must be rebuilt via the 'build database' button to pick up new versions.
- Annotation is restricted to significant hits from the analysis, so m/z values not selected by the statistical test are never matched.
- Compound prioritization depends on isotope/adduct scoring (M-score with default 2% intensity imprecision); imprecise intensities can misrank candidates.

## Evidence

- [methods] filter-then-sort order: "Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error"
- [methods] isotope/adduct prioritization: "MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status"
- [intro] significant-hits-only annotation: "MetaboShiny performs downstream data analysis directly on the m/z ratios and metabolite annotation occurs only on e.g. significant hits that result from the analysis."
- [methods] creatinine riboside M+H hit at 264 m/z: "a search including the HMDB (Fig. 1c, d) uncovered creatine riboside (M + H adduct) as a putative hit"
- [readme] ppm error margin setup: "Set the error margin of your mass spectrometer in parts per million (ppm)."

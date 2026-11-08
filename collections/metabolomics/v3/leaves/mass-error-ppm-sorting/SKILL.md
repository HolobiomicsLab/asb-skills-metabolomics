---
name: mass-error-ppm-sorting
description: Use when MetaboShiny/MetaDBparse database-search results for queried
  m/z values (with adduct, isotope label, ppm error and source database columns) must
  be reduced to prioritized putative (MSI level 2) compound annotations.
when_to_use_negative:
- You still need to generate the initial matches — this skill prioritizes existing
  MetaboShiny/MetaDBparse search results, it does not perform the m/z-to-database
  search itself (see the 'Pre-matching all m/z' step)
- The analysis goal is data normalization, statistical testing, dimension reduction
  (PCA, PLS-DA, t-SNE) or machine learning of the peak table rather than compound
  annotation
- MSI level 1 (identified compounds with authentic standards) confirmation is required
  — this skill by definition yields only level 2 putative annotations
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3370
tools:
- name: MetaboShiny
  role: Performs the filtering-by-main-peak-isotopes, adduct/isotope scoring, ppm-error
    sorting and SMILES collapsing, and assigns MSI level 2 putative annotation labels
    to retained hits
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Companion package that produced the compound database search results (adduct/isotope
    variant m/z values across 30+ databases) that this skill prioritizes
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: R
  role: Language in which MetaboShiny and MetaDBparse are implemented and in which
    the annotation workflow is run
  repo: https://github.com/joannawolthuis/MetaboShiny
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:20ed89255e510d77d21eaf0b6e4b396a7b44db97bcbf0a6f708208b0a59feca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3370
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mass-error-ppm-sorting

## Summary

MetaboShiny's post-search prioritization of compound matches: filter database-search results down to main-peak (100%) isotopes, rank surviving matches by ppm error within the user-defined mass tolerance, and emit putative MSI level 2 annotations.

## When to use

Use when MetaboShiny/MetaDBparse database-search results for queried m/z values (with adduct, isotope label, ppm error and source database columns) must be reduced to prioritized putative (MSI level 2) compound annotations.

## When NOT to use

- You still need to generate the initial matches — this skill prioritizes existing MetaboShiny/MetaDBparse search results, it does not perform the m/z-to-database search itself (see the 'Pre-matching all m/z' step)
- The analysis goal is data normalization, statistical testing, dimension reduction (PCA, PLS-DA, t-SNE) or machine learning of the peak table rather than compound annotation
- MSI level 1 (identified compounds with authentic standards) confirmation is required — this skill by definition yields only level 2 putative annotations

## Inputs

- Compound match results table from the MetaboShiny/MetaDBparse m/z database search, with per-match metadata: adduct, isotope label, ppm error, SMILES, formula and source database
- User-defined mass spectrometer error margin in ppm (set during file import in MetaboShiny)
- Adduct table and adduct rules table configured in MetaboShiny's Search/Adducts settings

## Outputs

- filtered_annotations.csv containing prioritized compound hits filtered by adduct, isotope and database, sorted by ppm error
- MSI level 2 ('putatively annotated compound') labels assigned to each retained hit
- On-screen prioritized match list in MetaboShiny's compound matches filtering panel

## How to apply

Load the compound match results produced by the MetaboShiny/MetaDBparse database search for the queried m/z values; each match carries metadata for adduct, isotope label, ppm error and source database. First filter the matches to retain only main-peak isotopes (100% peak), as MetaboShiny does in its search results table. Next score and filter the survivors on adduct and isotope status together with source database, and sort the remaining matches by ppm error within the mass tolerance set from the instrument's error margin in ppm. Collapse rows sharing an identical SMILES into single annotation entries while keeping distinct structural isomers (same formula, different SMILES) as separate rows. Assign each retained hit the MSI level 2 'putatively annotated compound' label and write the result to filtered_annotations.csv. The rationale is that MetaboShiny guides users to putative identifications by prioritizing compounds on isotope and adduct status and then mass accuracy, since it supports over 30 databases and over twenty commonly observed adducts and isotopes.

## Related tools

- **MetaboShiny** (Performs the filtering-by-main-peak-isotopes, adduct/isotope scoring, ppm-error sorting and SMILES collapsing, and assigns MSI level 2 putative annotation labels to retained hits) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Companion package that produced the compound database search results (adduct/isotope variant m/z values across 30+ databases) that this skill prioritizes) — https://github.com/joannawolthuis/MetaboShiny
- **R** (Language in which MetaboShiny and MetaDBparse are implemented and in which the annotation workflow is run) — https://github.com/joannawolthuis/MetaboShiny

## Examples

```
docker run -p 8080:8080 -v ~/MetaboShiny/:/root/MetaboShiny/:cached --rm -it jcwolthuis/metaboshiny:latest Rscript -e "MetaboShiny::start_metshi(inBrowser=F)"
```

## Evaluation signals

- filtered_annotations.csv exists and contains adduct, isotope and database filtering fields
- Every retained hit carries an MSI level 2 'putatively annotated compound' label
- Rows in the output are sorted by ppm error within the user-defined mass tolerance
- No rows sharing an identical SMILES remain collapsed as duplicates, while same-formula/different-SMILES structural isomers are kept as separate rows
- No non-main-peak (non-100%) isotope entries survive the isotope filter

## Limitations

- The annotation is putative (MSI level 2) — it does not confer confirmed identities without authentic standards
- MetaboShiny does not automatically update compound databases; database versions and download dates must be checked manually via the 'check if database exists' button
- MetaboShiny performs annotation only on significant hits resulting from the statistical analysis, so matches outside those hits are not prioritized
- The mass tolerance depends on the user-supplied instrument error margin in ppm, so sorting quality is bounded by that setting

## Evidence

- [methods] Methods quote on filtering order: "Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error"
- [methods] Methods quote on prioritization rationale: "MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status"
- [methods] Methods quote on MSI level 2 output: "MetaboShiny provinces MSI level 2 annotation i.e. putatively annotated compounds"
- [intro] Intro quote on adduct/isotope search scope: "supports over 30 databases (Table S1) and is capable of searching through millions of m/z values, including metabolite variants with over twenty commonly observed adducts and isotopes"
- [readme] README quote on ppm error margin setting: "Set the error margin of your mass spectrometer in parts per million (ppm)."

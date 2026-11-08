---
name: chromatographic-feature-correlation-filtering
description: Use when you have a chromatographic peak table from LC-MS lipidomics
  (e.g. Compound Discoverer output) with identified lipid features and numerous unidentified
  features, and you want to filter out co-eluting spurious/redundant features before
  downstream statistics.
when_to_use_negative:
- Input is already a filtered feature table or a feature-abundance matrix without
  the raw unidentified-feature annotations (RT apex, FWHM, per-sample quantitation)
  needed to evaluate the RT-window and correlation criteria
- You want to remove low-abundance or noisy features generally — this filter only
  removes unidentified features that are tightly correlated with (r ≈ 0.99) a co-eluting
  identified lipid, not features failing intensity or identification thresholds
- Working with data formats other than Compound Discoverer (.cdResult/.csv) or mzMine
  2 peak tables, or without LipiDex/LipiDex 2's spectral matching results linking
  lipids to features
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: LipiDex 2
  role: Provides the Peak Finder module whose filtering step removes unidentified
    features correlated with co-eluting identified lipid features; peak tables are
    passed downstream to Degreaser QC
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex
  role: Earlier tool that annotates chromatographic peak tables with lipid identifications
    and automatically filters peak tables for adduct peaks, in-source fragments, and
    dimers using the same correlation-based logic
  repo: https://github.com/coongroup/LipiDex
- name: LipiDex 2 Spectrum Searcher
  role: Supplies the lipid spectral matches that define the identified features against
    which unidentified co-eluting features are correlated
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2 Peak Finder
  role: Executes the correlation-based filtering directly on Compound Discoverer .cdResult
    peak tables
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:eb4506a9ec3f3f33c760042555f45e5a846838f20ec95eaa918a68a510fae5ec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LipiDex
  - LipiDex 2
  - LipiDex 2 Peak Finder
  - LipiDex 2 Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# chromatographic-feature-correlation-filtering

## Summary

A post-identification filtering skill from the LipiDex/LipiDex 2 Peak Finder that removes unidentified chromatographic features judged to be spurious or redundant byproducts of co-eluting identified lipid features, using retention-time proximity, sample prevalence, and intensity-profile correlation.

## When to use

Use when you have a chromatographic peak table from LC-MS lipidomics (e.g. Compound Discoverer output) with identified lipid features and numerous unidentified features, and you want to filter out co-eluting spurious/redundant features before downstream statistics.

## When NOT to use

- Input is already a filtered feature table or a feature-abundance matrix without the raw unidentified-feature annotations (RT apex, FWHM, per-sample quantitation) needed to evaluate the RT-window and correlation criteria
- You want to remove low-abundance or noisy features generally — this filter only removes unidentified features that are tightly correlated with (r ≈ 0.99) a co-eluting identified lipid, not features failing intensity or identification thresholds
- Working with data formats other than Compound Discoverer (.cdResult/.csv) or mzMine 2 peak tables, or without LipiDex/LipiDex 2's spectral matching results linking lipids to features

## Inputs

- Chromatographic peak table with identified lipid features and unidentified features (Compound Discoverer 3.3 .cdResult file or intermediate .csv peak table)
- Peak-table annotations: apex retention times, FWHM values, per-sample quantitation values for QC, blank, and biological samples
- LipiDex 2 Spectrum Searcher results (.csv) linking lipid identifications to chromatographic features

## Outputs

- Filtered peak table in which unidentified features meeting RT-window, prevalence, and 0.99-correlation criteria against a co-eluting identified lipid are removed (adducts, dimers, in-source fragments, other spurious features)
- Retained unidentified features that fail at least one criterion, carried forward for Degreaser QC and downstream statistical analysis (e.g. Metaboanalyst-compatible output)

## How to apply

Run Peak Finder filtering after spectral matching and peak annotation, when the peak table contains both identified lipids and unidentified features. For each unidentified feature, evaluate it against each identified lipid feature: (1) retention-time proximity — the unidentified feature's apex must fall within a window of FWHM ± 0.5 × FWHM around the identified feature's apex; (2) prevalence — the feature must be quantified in at least 10 non-QC, non-blank samples; (3) correlation — Pearson correlation of intensity profiles across samples must meet a high default threshold of 0.99, consistent with being an artifact (e.g. adduct, in-source fragment, dimer) of the co-eluting identified feature. An unidentified feature is filtered only when all three criteria are satisfied; failing any criterion retains the feature. In LipiDex 2, this filtering is one of the criteria used to boost confidence in identifications and quantitation, and results can be passed to Degreaser QC for further automated validation. Because the correlation threshold is very strict (0.99), features that merely co-elute but are not tightly correlated survive the filter, so the filter targets high-confidence artifacts rather than co-eluting biology.

## Related tools

- **LipiDex 2** (Provides the Peak Finder module whose filtering step removes unidentified features correlated with co-eluting identified lipid features; peak tables are passed downstream to Degreaser QC) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex** (Earlier tool that annotates chromatographic peak tables with lipid identifications and automatically filters peak tables for adduct peaks, in-source fragments, and dimers using the same correlation-based logic) — https://github.com/coongroup/LipiDex
- **LipiDex 2 Spectrum Searcher** (Supplies the lipid spectral matches that define the identified features against which unidentified co-eluting features are correlated) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2 Peak Finder** (Executes the correlation-based filtering directly on Compound Discoverer .cdResult peak tables) — https://github.com/coongroup/LipiDex-2/releases/latest

## Examples

```
Download LipiDex2.exe from https://github.com/coongroup/LipiDex-2/releases/latest, extract, and run it — then in the Peak Finder module load your Compound Discoverer 3.3 .cdResult peak table to apply the RT-window (FWHM ± 0.5×FWHM), ≥10-sample, and r = 0.99 correlation filters.
```

## Evaluation signals

- All unidentified features retained after filtering either fail at least one of the three criteria (RT window FWHM ± 0.5×FWHM, ≥10 non-QC/non-blank samples, Pearson r < 0.99 with the identified feature)
- Every removed feature is co-eluting with an identified lipid feature (apex within FWHM ± 0.5×FWHM) and has r ≥ 0.99 across the ≥10 qualifying samples — no removals should occur for features outside the RT window or below the correlation threshold
- Identified lipid features must all survive the filter; only unidentified features are eligible for removal
- Filtered peak table remains parseable by downstream LipiDex 2 modules (Degreaser QC) and Metaboanalyst without schema errors
- Spot-check a known spurious feature (e.g. an adduct of an identified lipid): its intensity profile across biological samples should track the parent lipid at r ≈ 0.99 and it should be absent from the filtered output

## Limitations

- The default correlation threshold is extremely strict (0.99); co-eluting features with slightly lower correlation will be retained, so the filter is conservative and may leave some redundant features in the table
- A feature is filtered only if it appears in at least 10 non-QC, non-blank samples; rare or low-prevalence artifacts below this count will not be removed
- The filter can only remove unidentified features correlated with *identified* lipids — unannotated spurious features with no co-eluting identification are not caught by this skill
- LipiDex 2 Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input for the peak table (LipiDex 1 accepts Compound Discoverer or mzMine 2 peak tables)

## Evidence

- [results] An unidentified feature is filtered only if it satisfies three criteria: it lies within the FWHM ± 0.5*FWHM window of the identified feature, appears in at least 10 non-QC, non-blank samples, and has a high default correlation value of 0.99.: [evidence span withheld — anchor status: not_found]
- [intro] Finally, LipiDex applied quality control (QC) measures after peak identification to filter lipid identifications and unidentified peaks to reduce spurious and redundant features.: "LipiDex applied quality control (QC) measures after peak identification to filter lipid identifications and unidentified peaks to reduce spurious and redundant features."
- [results] the filtered results after Peak Finder filtering to remove adducts, dimers, and other spurious features: "the filtered results after Peak Finder filtering to remove adducts, dimers, and other spurious features"
- [readme] All LipiDex 2 modules are stand-alone and pass .csv results files through the pipeline. However Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input for the peak table.: "All LipiDex 2 modules are stand-alone and pass .csv results files through the pipeline. However Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input"
- [readme] LipiDex 2 Results are filtered on 7 criteria to boost confidence in your identifications and quantitation: "IMPROVED: Results are filtered on 7 criteria to boost confidence in your identifications and quantitation"

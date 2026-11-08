---
name: benchmark-accuracy-statistics-computation
description: Use when evaluating modification-site localization accuracy of variable-mode
  spectral search hits, or when reproducing benchmark fractions (42% within distance
  2, 68% within distance 3) reported for 4,966 modified GNPSLIB structures.
when_to_use_negative:
- The goal is exact-mode identification of known molecules with matched precursor
  m/z rather than localization of modification sites under mass-shift variants
- Input spectra have been pre-processed externally and default peak merging (Peak
  Merge Threshold, Minimum Peak Count) or peak filtering (Peaks Per Window, Peak Filtering
  Window) settings no longer apply
- The candidate database is not the modified GNPSLIB set, so there is no ground-truth
  modification site against which to compute atom distances
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: VInSMoC
  role: Run in 'variable' search mode with default fragmentation settings to produce
    hits whose predicted modification sites are extracted and scored against correct
    sites
  repo: https://github.com/mohimanilab/VInSMoC
- name: npanalysis.org server (run.npanalysis.org)
  role: Hosts the VInSMoC task form used to configure search mode, fragmentation mode,
    ion mode, charge, tolerances, and minimum score, and to collect finished task
    results
  repo: https://run.npanalysis.org
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s43588-025-00923-5
    title: Identifying variants of molecules through database search of mass spectra
  source_content_hash: sha256:3b9f891c445f8377d6f41d830e5693bf39b9b924282821c8b3f77626248a9521
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/VInSMoC
  tools:
  - VInSMoC
  - npanalysis.org server (run.npanalysis.org)
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# benchmark-accuracy-statistics-computation

## Summary

Compute and reproduce benchmark accuracy metrics from VInSMoC variable-mode search results — specifically the fraction of modified GNPSLIB structures whose predicted modification site falls within average atom distance 2 or 3 of the correct site, reproducing Extended Data Fig. 2.

## When to use

Use when evaluating modification-site localization accuracy of variable-mode spectral search hits, or when reproducing benchmark fractions (42% within distance 2, 68% within distance 3) reported for 4,966 modified GNPSLIB structures.

## When NOT to use

- The goal is exact-mode identification of known molecules with matched precursor m/z rather than localization of modification sites under mass-shift variants
- Input spectra have been pre-processed externally and default peak merging (Peak Merge Threshold, Minimum Peak Count) or peak filtering (Peaks Per Window, Peak Filtering Window) settings no longer apply
- The candidate database is not the modified GNPSLIB set, so there is no ground-truth modification site against which to compute atom distances

## Inputs

- Modified molecule structure database built by applying 107 modification motifs to GNPSLIB base compounds (4,966 modified structures)
- Query mass spectra in MGF, mzXML, or mzML format (e.g. from MSV000086287)
- VInSMoC variable-mode task results reporting the best hit per input scan exceeding the minimum score threshold

## Outputs

- Tab-delimited table of modification-site localization accuracy, e.g. vinsmoc_site_localization.tsv, with fractions of entries at average atom distance ≤2 and ≤3
- Benchmark fractions comparable to Extended Data Fig. 2 (42% within distance 2, 68% within distance 3)

## How to apply

First build the benchmark database by applying the 107 frequent small-molecule modification motifs to GNPSLIB base compounds, yielding 4,966 modified structures. Run VInSMoC on the query spectra (MGF, mzXML, or mzML, e.g. from MSV000086287) in 'variable' search mode via run.npanalysis.org with default peak merging/filtering settings, fragmentation mode matching the molecule class, ion mode matching the input spectrum, and maximum absolute charge of 1 ([M+H]+/[M-H]-) or 2 ([M+2H]++/[M-2H]--). Collect the best hit per input scan that exceeded the minimum score threshold, each linked to its source database molecule. Extract the predicted modification site from each variable-mode hit, compute its average atom distance to the correct site across all entries, then tabulate the fraction of entries within distance 2 and distance 3 and write the results (e.g. to vinsmoc_site_localization.tsv). Validation: the computed fractions must reproduce the reported 42% (2,088 of 4,966) within distance 2 and 68% (3,377 of 4,966) within distance 3.

## Related tools

- **VInSMoC** (Run in 'variable' search mode with default fragmentation settings to produce hits whose predicted modification sites are extracted and scored against correct sites) — https://github.com/mohimanilab/VInSMoC
- **npanalysis.org server (run.npanalysis.org)** (Hosts the VInSMoC task form used to configure search mode, fragmentation mode, ion mode, charge, tolerances, and minimum score, and to collect finished task results) — https://run.npanalysis.org

## Evaluation signals

- Reproduced fraction within average atom distance ≤2 equals 42% (2,088 of 4,966 modified structures)
- Reproduced fraction within average atom distance ≤3 equals 68% (3,377 of 4,966 modified structures)
- Every reported hit corresponds to the best hit per input scan that exceeded the minimum score threshold and links back to its source database molecule
- Peak merging and peak filtering advanced settings were left at defaults (only changed if spectra were pre-processed before upload)
- Output table vinsmoc_site_localization.tsv contains fractions computed over the full 4,966-entry set

## Limitations

- Advanced peak pre-processing settings control how spectral input is pre-processed before scoring and 'should be left unchanged unless you have already down some preprocessing on your input spectra'
- Data inputs must be parsed by the server before use; malformed inputs show an error status and require checking that the format matches the declared extension
- Accuracy depends on default precursor/product ion tolerances; if unsure these 'can be safely left to the default'
- Results depend on maximum absolute charge choice (1: [M+H]+/[M-H]-; 2: adds [M+2H]++/[M-2H]--) matched to the ion mode of the input spectrum

## Evidence

- [results] Using default VInSMoC fragmentation settings, the predicted modification site is within average distance ≤2 of the correct site for 42% of entries: [evidence span withheld — anchor status: not_found]
- [methods] Generate the candidate structure database by applying the 107 modification motifs: [evidence span withheld — anchor status: not_found]
- [readme] Search mode can be either exact or variable.: "Search mode can be either exact or variable.
Fragmentation mode can work with generic molecules (Metaoblite mode) or specifically generalized peptides (Peptide mode)."
- [readme] Peak merging combines peaks that are within "Peak Merge Threshold": "Peak merging combines peaks that are within "Peak Merge Threshold" of one another, making sure to leave at least "Minimum Peak Count" peaks."

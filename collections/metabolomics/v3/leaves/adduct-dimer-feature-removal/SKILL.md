---
name: adduct-dimer-feature-removal
description: Use when a Compound Discoverer 3.3 peak table (from .cdResult or .csv)
  contains redundant unidentified features that correlate with identified lipid features,
  and you need to filter adducts, dimers, and other spurious features before quantitation.
when_to_use_negative:
- Input is already a filtered feature table where adducts/dimers have been removed
- Working with raw MS2/MS3 spectra for spectral matching rather than a chromatographic
  peak table — Spectrum Searcher, not Peak Finder, is the right module
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipiDex 2 Peak Finder
  role: 'Primary module: loads Compound Discoverer peak table, runs pairwise correlation
    filtering to remove adducts, dimers, and spurious features'
  repo: https://github.com/coongroup/LipiDex-2
- name: Compound Discoverer
  role: Upstream tool that produces the .cdResult or .csv peak table consumed by Peak
    Finder
- name: LipiDex 1
  role: Legacy version still available with similar adduct/dimer filtering logic
  repo: https://github.com/coongroup/LipiDex
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:5197ea10d8455ac466dea3a1d1a76b3653d224e36d60d0d1ac2f276bdc73a961
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coongroup/LipiDex-2
  tools:
  - Compound Discoverer
  - LipiDex 1
  - LipiDex 2
  - LipiDex 2 Peak Finder
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# adduct-dimer-feature-removal

## Summary

Applies LipiDex 2's Peak Finder pairwise correlation filtering to remove adduct peaks, in-source fragments, and dimers correlated with primary lipid features in a chromatographic peak table. On the Nile rat plasma lipidomics dataset (MSV000091033), this step reduced 14,826 unidentified features and 779 lipid identifications to 13,898 unidentified and 719 identified features.

## When to use

Use when a Compound Discoverer 3.3 peak table (from .cdResult or .csv) contains redundant unidentified features that correlate with identified lipid features, and you need to filter adducts, dimers, and other spurious features before quantitation.

## When NOT to use

- Input is already a filtered feature table where adducts/dimers have been removed
- Working with raw MS2/MS3 spectra for spectral matching rather than a chromatographic peak table — Spectrum Searcher, not Peak Finder, is the right module

## Inputs

- Compound Discoverer 3.3 .cdResult file or converted .csv peak table from LC-MS lipidomics data

## Outputs

- Filtered peak table (e.g. filtered_peak_features.csv) with remaining unidentified and identified features and their counts

## How to apply

Load the peak table produced by Compound Discoverer 3.3 into LipiDex 2 Peak Finder (either directly from .cdResult or via .csv). Run pairwise correlation filtering between unidentified and identified features using Pearson/Spearman correlation within a shared FWHM retention-time window, requiring at least 10 samples for correlation and a 0.99 correlation cutoff. Peak Finder then removes features identified as adducts, dimers, or other spurious features correlated with primary lipid features. The output is a filtered peak table with remaining features and their updated counts. Rationale: LipiDex applies QC measures after peak identification to reduce spurious and redundant features, so filtering these co-eluting isobaric or adduct-related species improves confidence in downstream lipid identifications and quantitation.

## Related tools

- **LipiDex 2 Peak Finder** (Primary module: loads Compound Discoverer peak table, runs pairwise correlation filtering to remove adducts, dimers, and spurious features) — https://github.com/coongroup/LipiDex-2
- **Compound Discoverer** (Upstream tool that produces the .cdResult or .csv peak table consumed by Peak Finder)
- **LipiDex 1** (Legacy version still available with similar adduct/dimer filtering logic) — https://github.com/coongroup/LipiDex

## Examples

```
Load filtered_peak_features.csv into LipiDex2.exe Peak Finder with Compound Discoverer 3.3 .cdResult input; run correlation filtering (≥10 samples, Pearson/Spearman r ≥ 0.99) to output the filtered peak table.
```

## Evaluation signals

- Feature counts after filtering should be lower than before (e.g. on MSV000091033: 14,826→13,898 unidentified; 779→719 identified)
- Filtered table must be non-empty and contain both unidentified and identified features
- Each removed feature should exhibit high pairwise correlation (≥0.99) with an identified feature within a shared FWHM retention-time window
- No identified features should be spuriously removed — only adducts/dimers/other spurious features correlated to primary lipid features

## Limitations

- Requires LipiDex 2 which needs .NET 7.0 and Thermo RawFileReader compatibility; legacy LipiDex 1 is no longer supported
- Peak Finder requires Compound Discoverer version 3.3 .cdResult files as input for the peak table — older versions or other peak-picking software output must first be converted
- Correlation cutoff (0.99) and minimum sample count (10) are tuned for this plasma lipidomics workflow; other datasets may need parameter adjustment
- Removal depends on identified features being present as anchors — very sparse identifications may reduce filtering efficacy

## Evidence

- [results] Peak Finder filters remove adducts, dimers, and other spurious features: "the filtered results after Peak Finder filtering to remove adducts, dimers, and other spurious features"
- [intro] LipiDex applies QC measures after peak identification to filter lipid identifications and unidentified peaks: "LipiDex applied quality control (QC) measures after peak identification to filter lipid identifications and unidentified peaks to reduce spurious and redundant features"
- [readme] Peak Finder direct reading of Compound Discoverer .cdResult files: "Direct reading of Compound Discoverer .cdResult files, avoiding intermediate conversion to `.csv` peak table"
- [readme] README filtering criteria: [evidence span withheld — anchor status: external_doc]

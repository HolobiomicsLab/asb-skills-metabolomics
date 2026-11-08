---
name: multi-software-lipid-annotation-corroboration-analysis
description: Use when a MZmine feature table has been annotated by multiple lipid
  identification tools (e.g., LipidMatch, MS-DIAL, Greazy) and you need to measure
  cross-software agreement of fatty acyl constituent assignments per polarity.
when_to_use_negative:
- You have only a single software's annotations — corroboration cannot be assessed
  without at least two independent identification sources.
- Feature annotations from different software have not been aligned to a common feature
  table (m/z/RT alignment must precede comparison).
- The goal is de novo lipid identification rather than benchmarking agreement between
  existing annotation sets.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: LipidMatch
  role: Source of the lipid identifications (fatty acyl constituent annotations) being
    corroborated; written in R and appended to feature tables.
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MS-DIAL
  role: Independent annotation software whose identifications are used to corroborate
    LipidMatch assignments after alignment to the shared feature table.
- name: Greazy
  role: Independent annotation software whose identifications are used to corroborate
    LipidMatch assignments after alignment to the shared feature table.
- name: MZmine
  role: Peak picking software producing the underlying feature table that identifications
    from all three tools are appended to.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:fc06cb3a9c196e32e51bc849b9dcab04d4fa97b9c3cf30da62fb8d43e6f87cc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - CombineSoftwareIDs.R
  - Greazy
  - LipidMatch
  - MS-DIAL
  - MZmine
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multi-software-lipid-annotation-corroboration-analysis

## Summary

Quantifies agreement between lipid annotations from independent software (LipidMatch, MS-DIAL, Greazy) by appending all identifications to a shared MZmine feature table via CombineSoftwareIDs.R and computing how often each LipidMatch fatty acyl assignment is corroborated by at least one other tool per ion polarity.

## When to use

Use when a MZmine feature table has been annotated by multiple lipid identification tools (e.g., LipidMatch, MS-DIAL, Greazy) and you need to measure cross-software agreement of fatty acyl constituent assignments per polarity.

## When NOT to use

- You have only a single software's annotations — corroboration cannot be assessed without at least two independent identification sources.
- Feature annotations from different software have not been aligned to a common feature table (m/z/RT alignment must precede comparison).
- The goal is de novo lipid identification rather than benchmarking agreement between existing annotation sets.

## Inputs

- MZmine feature table (per polarity) with m/z and retention time columns
- LipidMatch identifications appended to the feature table
- MS-DIAL identifications appended to the feature table
- Greazy identifications appended to the feature table
- CombineSoftwareIDs.R script for alignment (10 ppm m/z, 0.2 min RT windows)

## Outputs

- Per-polarity corroboration percentage (fraction of LipidMatch fatty acyl assignments corroborated by MS-DIAL or Greazy)
- Subset of features identified by all three software (e.g., 65 negative-mode, 68 positive-mode features)
- Per-software, per-polarity counts of total annotated features (e.g., LipidMatch 159 negative / 1401 positive; MS-DIAL 143 / 411; Greazy 94 / 180)

## How to apply

Build the aligned annotation workbook by appending LipidMatch, MS-DIAL, and Greazy identifications to the MZmine feature table using CombineSoftwareIDs.R with alignment windows of 10 ppm m/z and 0.2 min retention time. Split the table by polarity and retain only features carrying identifications from all three software (65 negative-mode, 68 positive-mode features in the Red Cross plasma benchmark). For each LipidMatch annotation in these intersection subsets, check whether the fatty acyl constituent composition matches an identification from MS-DIAL or Greazy, treating plasmenyl/plasmanyl pairs differing by one degree of unsaturation as the same identification. Compute the per-polarity corroboration percentage — the paper benchmarks 98% in negative mode and 92% in positive mode — and additionally count total annotated features per software per polarity to confirm which tool yields the most identifications.

## Related tools

- **LipidMatch** (Source of the lipid identifications (fatty acyl constituent annotations) being corroborated; written in R and appended to feature tables.) — https://github.com/GarrettLab-UF/LipidMatch
- **MS-DIAL** (Independent annotation software whose identifications are used to corroborate LipidMatch assignments after alignment to the shared feature table.)
- **Greazy** (Independent annotation software whose identifications are used to corroborate LipidMatch assignments after alignment to the shared feature table.)
- **MZmine** (Peak picking software producing the underlying feature table that identifications from all three tools are appended to.)

## Examples

```
Rscript CombineSoftwareIDs.R --feature-table mzmine_features.csv --lipidmatch LM_IDs.csv --msdial MSDIAL_IDs.csv --greazy Greazy_IDs.csv --ppm 10 --rt 0.2
```

## Evaluation signals

- Negative-mode corroboration percentage falls in the reported range (98% of LipidMatch fatty acyl assignments corroborated by at least one other software).
- Positive-mode corroboration percentage falls in the reported range (92%).
- Per-software counts show LipidMatch exceeding both MS-DIAL and Greazy in each polarity (e.g., 159 vs 143 vs 94 negative; 1401 vs 411 vs 180 positive).
- Only features with identifications from all three software are included in the corroboration subset (65 negative, 68 positive features).
- Plasmenyl/plasmanyl pairs differing by one degree of unsaturation are counted as matching identifications rather than discrepancies.

## Limitations

- Corroboration assesses agreement of fatty acyl constituents, not full structural identity; concordance does not guarantee identical headgroup or sn-position assignments.
- Comparison is restricted to the intersection of features identified by all software, so uniquely identified features (a key strength of LipidMatch) are excluded from the agreement metric.
- Alignment depends on the 10 ppm m/z and 0.2 min RT windows; misalignment can artificially inflate or deflate agreement.
- LipidMatch, MS-DIAL, and Greazy differ in lipid type coverage (56 vs 34 vs 24 lipid types), which affects how many features can be compared at all.

## Evidence

- [abstract] 92% corroboration in positive mode: "92% of LipidMatch identifications by fatty acyl constituents were corroborated by at least one other software in positive mode and 98% in negative ion mode."
- [methods] Combining software identifications: "Once lipid identifications are obtained using LipidMatch, identifications from any other software such as Greazy [17], LipidSearch (Thermo Scientific, San Jose, CA), and MS-DIAL can be appended in"
- [discussion] Most identifications via LipidMatch: "Compared to MS-DIAL and Greazy, LipidMatch was found to provide the most lipid identifications for Red Cross plasma"
- [methods] Feature table sources for alignment: [evidence span withheld — anchor status: not_found]
- [readme] Modular combination with other software (README): "LipidMatch is modular, allowing it to fit in various workflows you may have in your lab. For example LipidMatch can be used with various peak picking software"

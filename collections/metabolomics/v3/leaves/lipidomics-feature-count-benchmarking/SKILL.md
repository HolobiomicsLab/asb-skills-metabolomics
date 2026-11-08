---
name: lipidomics-feature-count-benchmarking
description: Use when you have identifications from ≥2 lipid annotation software appended
  to the same MZmine feature table and need to compare annotation coverage and cross-software
  corroboration per polarity.
when_to_use_negative:
- Input is a raw vendor file or an unannotated peak table — annotations from all tools
  must already be appended before benchmarking
- Comparing tools that annotate different lipid structural levels — corroboration
  is defined on fatty acyl constituent composition, not just class-level matches
- Waters vendor files, which LipidMatch does not currently support
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: LipidMatch
  role: R-based tool whose identifications (159 neg / 1401 pos features) are the benchmarked
    set being corroborated
  repo: https://github.com/GarrettLab-UF/LipidMatch
- name: MS-DIAL
  role: Comparison software whose annotations (143 neg / 411 pos features) corroborate
    or contradict LipidMatch assignments
- name: Greazy
  role: Comparison software whose annotations (94 neg / 180 pos features) corroborate
    or contradict LipidMatch assignments
- name: CombineSoftwareIDs.R
  role: R script used to append LipidMatch, MS-DIAL, and Greazy identifications onto
    the MZmine feature table (10 ppm m/z, 0.2 min RT windows)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1744-3
    title: 'LipidMatch: an automated workflow for rule-based lipid identification
      using untargeted high-resolution tandem mass spectrometry data'
  source_content_hash: sha256:09c9ce4e89839c7fcf5a1ab442e2042b8cdf061ae270a094ec4da93fdc93a457
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GarrettLab-UF/LipidMatch
  tools:
  - CombineSoftwareIDs.R
  - Greazy
  - LipidMatch
  - MS-DIAL
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# lipidomics-feature-count-benchmarking

## Summary

Benchmarks lipid identification software (e.g., LipidMatch, MS-DIAL, Greazy) by counting annotated features per tool per ion polarity and computing the percentage of one tool's fatty acyl constituent identifications corroborated by the others on shared features.

## When to use

Use when you have identifications from ≥2 lipid annotation software appended to the same MZmine feature table and need to compare annotation coverage and cross-software corroboration per polarity.

## When NOT to use

- Input is a raw vendor file or an unannotated peak table — annotations from all tools must already be appended before benchmarking
- Comparing tools that annotate different lipid structural levels — corroboration is defined on fatty acyl constituent composition, not just class-level matches
- Waters vendor files, which LipidMatch does not currently support

## Inputs

- Aligned MZmine feature table with appended lipid identifications from LipidMatch, MS-DIAL, and Greazy (e.g., Additional file 3: Table S4 workbook)
- Polarity-labeled annotation lists per software (feature m/z, retention time, lipid assignment, fatty acyl constituents)

## Outputs

- Per-polarity cross-software corroboration percentages (fraction of one tool's fatty acyl identifications matched by ≥1 other tool)
- Counts of annotated features per software per ion polarity
- Summary CSV of corroboration statistics and per-software feature counts

## How to apply

Append identifications from each tool (LipidMatch, MS-DIAL, Greazy) to the aligned feature table using CombineSoftwareIDs.R, aligning identifications across software with windows of 10 ppm m/z and 0.2 min retention time. Split the table by polarity (positive and negative) and filter to only features carrying identifications from all software being compared (in the paper this yielded 68 positive-mode and 65 negative-mode features). For each LipidMatch annotation in the intersection, check whether the fatty acyl constituent composition matches an MS-DIAL or Greazy identification, treating plasmenyl/plasmanyl pairs differing by one degree of unsaturation as the same identification. Compute the corroboration percentage per polarity (paper reported 92% positive, 98% negative) and also count total annotated features per software per polarity (LipidMatch 159 neg/1401 pos; MS-DIAL 143 neg/411 pos; Greazy 94 neg/180 pos) to rank tools by identification yield.

## Related tools

- **LipidMatch** (R-based tool whose identifications (159 neg / 1401 pos features) are the benchmarked set being corroborated) — https://github.com/GarrettLab-UF/LipidMatch
- **MS-DIAL** (Comparison software whose annotations (143 neg / 411 pos features) corroborate or contradict LipidMatch assignments)
- **Greazy** (Comparison software whose annotations (94 neg / 180 pos features) corroborate or contradict LipidMatch assignments)
- **CombineSoftwareIDs.R** (R script used to append LipidMatch, MS-DIAL, and Greazy identifications onto the MZmine feature table (10 ppm m/z, 0.2 min RT windows))

## Evaluation signals

- Per-polarity corroboration percentages fall in the reported ranges: 98% (negative mode) and 92% (positive mode) for LipidMatch fatty acyl constituents corroborated by ≥1 other tool
- Per-software counts match reported values: LipidMatch 159 neg/1401 pos, MS-DIAL 143 neg/411 pos, Greazy 94 neg/180 pos, with LipidMatch exceeding both alternatives in each polarity
- Intersection subsets contain exactly the all-three-software features (65 negative, 68 positive) after 10 ppm / 0.2 min alignment
- Plasmenyl/plasmanyl pairs differing by one degree of unsaturation are consistently counted as the same identification across comparisons
- Summary CSV is split by polarity and contains both corroboration percentages and per-software counts

## Limitations

- Corroboration depends on the cross-software alignment window (10 ppm m/z, 0.2 min retention time); tighter or looser windows change the overlap set
- Structural-equivalence decisions (plasmenyl/plasmanyl treated as identical) are made by the analyst and directly affect the corroboration percentage
- Key artifacts including the annotation workbook and CombineSoftwareIDs.R are provided only as additional files, not archived in a public repository
- Identifications were benchmarked on Red Cross plasma UHPLC-HRMS/MS data; corroboration rates may differ on other sample types or instruments

## Evidence

- [abstract] Cross-software corroboration reported in the abstract: "92% of LipidMatch identifications by fatty acyl constituents were corroborated by at least one other software in positive mode and 98% in negative ion mode."
- [discussion] Identification yield comparison: "Compared to MS-DIAL and Greazy, LipidMatch was found to provide the most lipid identifications for Red Cross plasma"
- [methods] Appending identifications from other tools: "Once lipid identifications are obtained using LipidMatch, identifications from any other software such as Greazy [17], LipidSearch (Thermo Scientific, San Jose, CA), and MS-DIAL can be appended in"
- [readme] README on combining results across software: "LipidMatch is modular, allowing it to fit in various workflows you may have in your lab. For example LipidMatch can be used with various peak picking software"

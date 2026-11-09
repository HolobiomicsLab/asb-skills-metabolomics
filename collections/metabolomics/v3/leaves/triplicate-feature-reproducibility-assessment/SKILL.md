---
name: triplicate-feature-reproducibility-assessment
description: Use when a label-free LC-MS dataset contains a spiked-in standard concentration
  series measured in triplicate (e.g. 11 levels of plasma spike-ins) and you need
  to verify quantification linearity and cross-run feature reproducibility.
when_to_use_negative:
- The data are already aggregated to a final feature table with no concentration series
  or technical replicates — there is nothing to correlate across levels or runs.
- Input is unprocessed mzML centroid data before FeatureFinderMetabo detection, RT
  alignment, and feature linking; reproducibility must be assessed on linked features,
  not raw traces.
- Samples are proteomics SILAC/iTRAQ experiments or other designs without a spike-in
  concentration series in a single matrix background.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FeatureFinderMetabo
  role: Detects mass traces and assembles them into features (m/z spacing, co-elution,
    SVM isotope-pattern classifier) whose intensities feed the reproducibility and
    correlation analysis
  repo: github.com/OpenMS/OpenMS
- name: MapAlignmentPoseClustering
  role: Corrects retention-time drift across the 33 triplicate measurements so features
    can be compared run-to-run
- name: FeatureLinkerUnlabeledQT
  role: Links coinciding features across all measurements, producing the unified feature
    table on which reproducibility (presence in ≥ 50% of runs) is evaluated
- name: TOPPAS
  role: Graphical workflow editor used to build the fully automated detection → alignment
    → linking → text-export pipeline that generates the quantification table
- name: KNIME
  role: Platform in which the OpenMS feature-detection/linking workflow was implemented
    for the spike-in series analysis
- name: OpenMS
  role: Framework providing the TOPP tools and TextExporter step used to output the
    tabular feature-intensity file
  repo: github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:346f034a69e5055351866935e8e74d2a5f0e22b0f2e009685060c97d10f3e8c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - KNIME
  - MapAlignmentPoseClustering
  - OpenMS
  - TOPPAS
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# triplicate-feature-reproducibility-assessment

## Summary

Assess the reproducibility and linear quantification of linked LC-MS feature intensities from a spike-in concentration series measured in randomized triplicate, using Pearson correlation (R ≥ 0.98) and a reproducible-feature definition (present in ≥ 50% of measurements).

## When to use

Use when a label-free LC-MS dataset contains a spiked-in standard concentration series measured in triplicate (e.g. 11 levels of plasma spike-ins) and you need to verify quantification linearity and cross-run feature reproducibility.

## When NOT to use

- The data are already aggregated to a final feature table with no concentration series or technical replicates — there is nothing to correlate across levels or runs.
- Input is unprocessed mzML centroid data before FeatureFinderMetabo detection, RT alignment, and feature linking; reproducibility must be assessed on linked features, not raw traces.
- Samples are proteomics SILAC/iTRAQ experiments or other designs without a spike-in concentration series in a single matrix background.

## Inputs

- Linked feature-intensity table exported from FeatureLinkerUnlabeledQT via OpenMS TextExporter (tabular text file)
- 33 mzML measurements of a human plasma spike-in series: 11 concentration levels (10 mg/L to 0.01 mg/L plus non-spiked control) × triplicate
- List of spiked standard compounds with their known concentrations

## Outputs

- Per-standard Pearson correlation coefficient (R) between concentration and feature intensity; acceptance threshold R ≥ 0.98
- Set of features reproducible in ≥ 50% of the 33 measurements (~1,168 features expected)
- Per-measurement reproducible-feature counts (expected 1,066 ± 36) and total detected feature counts (expected ~2,020 ± 236)
- Flagged concentration levels with no detectable signal (e.g. six lowest propionyl-l-carnitine-d3 concentrations)

## How to apply

After the OpenMS pipeline (FeatureFinderMetabo feature detection, MapAlignmentPoseClustering retention-time correction, FeatureLinkerUnlabeledQT linking, TextExporter export) has produced a linked feature table for the 33 mzML measurements, extract the intensities of each spiked standard's linked feature and compute the Pearson correlation between spike-in concentration (10 mg/L down to 0.01 mg/L plus the non-spiked control) and detected intensity across concentration levels. Accept linearity only when R ≥ 0.98 for every detectable standard compound. For reproducibility, count each linked feature as reproducible if it is present in at least 50% of the 33 measurements, then compare the resulting set size (~1,168 features) and per-measurement reproducible count (1,066 ± 36 of ~2,020 ± 236 detected) against expected ranges. Note compounds whose lowest concentrations yield no detectable signal (e.g. the six lowest propionyl-l-carnitine-d3 levels) and exclude those levels from the correlation rather than imputing.

## Related tools

- **FeatureFinderMetabo** (Detects mass traces and assembles them into features (m/z spacing, co-elution, SVM isotope-pattern classifier) whose intensities feed the reproducibility and correlation analysis) — github.com/OpenMS/OpenMS
- **MapAlignmentPoseClustering** (Corrects retention-time drift across the 33 triplicate measurements so features can be compared run-to-run)
- **FeatureLinkerUnlabeledQT** (Links coinciding features across all measurements, producing the unified feature table on which reproducibility (presence in ≥ 50% of runs) is evaluated)
- **TOPPAS** (Graphical workflow editor used to build the fully automated detection → alignment → linking → text-export pipeline that generates the quantification table)
- **KNIME** (Platform in which the OpenMS feature-detection/linking workflow was implemented for the spike-in series analysis)
- **OpenMS** (Framework providing the TOPP tools and TextExporter step used to output the tabular feature-intensity file) — github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in *.mzML && MapAlignmentPoseClustering && FeatureLinkerUnlabeledQT && TextExporter && python -c "import scipy.stats as s; r,p=s.pearsonr(conc,intens); assert r>=0.98"
```

## Evaluation signals

- Every detectable spiked standard compound must reach Pearson R ≥ 0.98 between concentration and feature intensity.
- The reproducible-feature set (present in ≥ 50% of 33 measurements) should be ≈ 1,168 features; per-run reproducible counts should fall near 1,066 ± 36.
- Total detected features per measurement should average ~2,020 ± 236, with ~351 singly, 12 doubly, 6 triply charged.
- No imputed values: compounds with undetectable levels (e.g. propionyl-l-carnitine-d3's six lowest concentrations) must have those levels flagged and excluded from the correlation.
- The linked feature table must originate from the full automated chain (detection, RT correction via MapAlignmentPoseClustering, linking via FeatureLinkerUnlabeledQT, TextExporter) rather than manual peak picking.

## Limitations

- Very low concentrations may yield no detectable signal at all (neither algorithm nor visual inspection) — in this study the first six concentrations of propionyl-l-carnitine-d3 could not be detected, so correlation is only valid over detectable levels.
- Reproducibility depends on the ~53% of detected features that are reproducible (1,066 ± 36 of ~2,020); roughly half of detected features do not pass the ≥ 50% presence threshold.
- Feature counts are sensitive to the OpenMS version and parameters (the study used OpenMS 1.10); changing these alters the reproducible set.
- The benchmark is specific to a complex human plasma background on a Waters Acquity/Synapt HDMS platform; transfer to other matrices or instruments is not validated here.

## Evidence

- [results] Triplicate intensity reproducibility and R ≥ 0.98 linearity: "The Pearson correlation coefficient between each compound's concentrations and corresponding feature intensities revealed an excellent linear relationship (R≥ 0.98 for all standard compounds)"
- [results] Reproducible-feature set size: "We found a set of 1,168 features to be reproducible in at least 50% of all measurements"
- [results] Per-measurement reproducible and total counts: "we found 1,066 ± 36 reproducible features in each of the 33 measurements—that is, about 53%"
- [results] Undetectable low-level compound: "In the case of the propionyl-l-carnitine-d3 standard, we could not detect any signals for the first six concentrations"
- [results] Export step feeding statistical analysis: "and the ability to export to a tabular text file for subsequent statistical analyses"
- [results] RT correction and linking tools named explicitly: "such as MapAlignmentPoseClustering for retention time correction"

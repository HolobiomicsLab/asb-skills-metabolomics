---
name: spike-in-concentration-linearity-analysis
description: Use when LC-MS mzML measurements of a metabolite spike-in dilution series
  (multiple concentration levels, technical replicates) must be checked for quantification
  linearity and replicate intensity reproducibility after feature detection/linking.
when_to_use_negative:
- The input is already a linked feature-intensity table — re-running FeatureFinderMetabo/FeatureLinkerUnlabeledQT
  is unnecessary; go directly to the correlation analysis
- The goal is compound identification or metabolite annotation — this skill validates
  quantification linearity and reproducibility, not identity assignment (e.g., Human
  Metabolome Database queries are a separate step)
- The study design has no concentration series or no technical replicates, so neither
  linearity nor replicate reproducibility can be estimated
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FeatureFinderMetabo
  role: Detects mass traces and assembles them into features (intensity source for
    the concentration-vs-intensity correlation) from each mzML run
  repo: https://github.com/OpenMS/OpenMS
- name: MapAlignmentPoseClustering
  role: Corrects retention-time drift across triplicate runs so corresponding features
    align before linking
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureLinkerUnlabeledQT
  role: Links coinciding features across all 33 measurements to produce a single intensity
    matrix for linearity analysis
  repo: https://github.com/OpenMS/OpenMS
- name: TOPPAS
  role: Graphical workflow editor in which the feature detection → RT correction →
    linking → export pipeline is assembled
- name: KNIME
  role: Alternative workflow platform for executing the OpenMS quantification pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:78abbfec43021c043de530efe2057b359b0320a2855703cd7572b85fc34c9777
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - KNIME
  - MapAlignmentPoseClustering
  - OpenMS
  - TOPPAS
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# spike-in-concentration-linearity-analysis

## Summary

Assesses label-free quantification reliability by computing Pearson correlation between spike-in compound concentrations and their linked feature intensities across a dilution series in a complex background (e.g., human plasma), plus technical-replicate reproducibility of the feature set.

## When to use

Use when LC-MS mzML measurements of a metabolite spike-in dilution series (multiple concentration levels, technical replicates) must be checked for quantification linearity and replicate intensity reproducibility after feature detection/linking.

## When NOT to use

- The input is already a linked feature-intensity table — re-running FeatureFinderMetabo/FeatureLinkerUnlabeledQT is unnecessary; go directly to the correlation analysis
- The goal is compound identification or metabolite annotation — this skill validates quantification linearity and reproducibility, not identity assignment (e.g., Human Metabolome Database queries are a separate step)
- The study design has no concentration series or no technical replicates, so neither linearity nor replicate reproducibility can be estimated

## Inputs

- 33 mzML files of a human plasma spike-in series (11 concentration levels 10 mg/L to 0.01 mg/L plus non-spiked control, triplicate randomized injections from a Waters Acquity/Synapt HDMS platform)
- Concentration table mapping each spike-in standard compound to its level per measurement

## Outputs

- Per-standard Pearson correlation coefficient (R) between concentration and linked feature intensity across detectable levels
- Tabular text file (TextExporter) of linked feature intensities across all measurements
- Reproducible-feature set: features present in ≥50% of measurements, with per-run reproducible counts (e.g., 1,066 ± 36 of 2,020 ± 236 total)

## How to apply

Load the full set of mzML measurements of the concentration series (e.g., 33 runs = 11 levels from 10 mg/L down to 0.01 mg/L plus non-spiked control, randomized triplicates) and run FeatureFinderMetabo (OpenMS) to detect mass traces and assemble features using m/z spacing, co-elution, and the SVM-based isotope-pattern classifier. Correct retention-time drift across replicates with MapAlignmentPoseClustering, then link coinciding features across all measurements with FeatureLinkerUnlabeledQT and export intensities to a tabular text file via TextExporter. For each spiked standard, extract its linked feature intensities across detectable concentration levels and compute the Pearson correlation between concentration and intensity; note and report levels with no detectable signal (e.g., the six lowest propionyl-l-carnitine-d3 concentrations) rather than forcing them into the fit. Quantify replicate reproducibility by counting features present in at least 50% of measurements and comparing per-run counts against the total detected features.

## Related tools

- **FeatureFinderMetabo** (Detects mass traces and assembles them into features (intensity source for the concentration-vs-intensity correlation) from each mzML run) — https://github.com/OpenMS/OpenMS
- **MapAlignmentPoseClustering** (Corrects retention-time drift across triplicate runs so corresponding features align before linking) — https://github.com/OpenMS/OpenMS
- **FeatureLinkerUnlabeledQT** (Links coinciding features across all 33 measurements to produce a single intensity matrix for linearity analysis) — https://github.com/OpenMS/OpenMS
- **TOPPAS** (Graphical workflow editor in which the feature detection → RT correction → linking → export pipeline is assembled)
- **KNIME** (Alternative workflow platform for executing the OpenMS quantification pipeline)

## Examples

```
FeatureFinderMetabo -in plasma_spikein_005mgL.mzML -out features_005mgL.featureXML
```

## Evaluation signals

- Pearson correlation R ≥ 0.98 between concentration and feature intensity for every detectable standard compound
- Reproducible-feature count in ≥50% of all 33 measurements is ~1,168, i.e., 1,066 ± 36 per run (~53% of the 2,020 ± 236 total features detected per measurement)
- Non-detected levels are explicitly flagged, e.g., no signals for the first six concentrations of propionyl-l-carnitine-d3 (confirmed by visual inspection), rather than silently imputed
- Feature charge composition is plausible for metabolites: ~351 singly, 12 doubly, 6 triply charged of the average 2,020 features per measurement
- Per-run feature counts remain stable across randomized triplicate injections, indicating platform and processing reproducibility

## Limitations

- Very low spike-in concentrations may yield no detectable signal (propionyl-l-carnitine-d3 showed no signals for its six lowest concentrations), so correlation must be computed only over detectable levels
- Linearity was demonstrated in a human plasma background on a Waters Acquity/Synapt HDMS platform; other matrices, instruments, or separation technologies may require revalidation
- The article reports sensitivity/specificity 'above 95% on typical data' without defining 'typical data' or explicit parameter thresholds, so parameter tuning may be needed on new datasets
- Benchmark comparisons with XCMS and CAMERA were run on a specific version (OpenMS 1.10); results may differ with newer releases

## Evidence

- [results] Excellent linear quantification of spiked standards: "The Pearson correlation coefficient between each compound's concentrations and corresponding feature intensities revealed an excellent linear relationship (R≥ 0.98 for all standard compounds)"
- [results] Reproducible feature set definition: "We found a set of 1,168 features to be reproducible in at least 50% of all measurements"
- [results] Missing low-concentration signals: "In the case of the propionyl-l-carnitine-d3 standard, we could not detect any signals for the first six concentrations"
- [abstract] Automated pipeline validation in complex background: "We obtained excellent results in a fully automated data-processing pipeline with respect to both accuracy and reproducibility."
- [readme] OpenMS as reusable open-source toolkit: [evidence span withheld — anchor status: not_found]

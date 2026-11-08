---
name: lc-ms-spike-in-reference-matching
description: Use when you have repeated spike-in LC-MS measurements (e.g. human plasma
  background) plus a ground-truth list of reference analytes, and need to quantify
  detection sensitivity/specificity or verify the >95% claim.
when_to_use_negative:
- No spike-in reference analytes or ground-truth feature list exists, so sensitivity/specificity
  cannot be computed against a truth set
- Input is already a validated feature table and the goal is downstream statistics
  (e.g. feature linking via FeatureLinkerUnlabeledQT or multivariate analysis), not
  detection-quality benchmarking
- Data is not small-molecule/metabolomics LC-MS feature detection (the algorithm targets
  metabolites and small molecules such as lipidomics, not protein identification)
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: FeatureFinderMetabo
  role: Detects mass traces and assembles them into metabolite features whose intensities
    are compared against the spike-in reference analytes
  repo: github.com/OpenMS/OpenMS
- name: OpenMS 1.10
  role: Framework in which FeatureFinderMetabo is implemented and from which features
    are exported to a tabular text file for comparison
  repo: github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:8f81a7a266a9e8b43a0c2a653c37239e11383d8a3cb15ceee1fa6a7bac15e7eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - OpenMS
  - OpenMS 1.10
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lc-ms-spike-in-reference-matching

## Summary

A benchmarking skill for verifying that FeatureFinderMetabo-derived metabolite feature intensities match spike-in reference analytes in LC-MS data, by computing sensitivity (recall) and specificity (precision) against the reported >95% threshold (97% precision, 96% recall on matched simulated data).

## When to use

Use when you have repeated spike-in LC-MS measurements (e.g. human plasma background) plus a ground-truth list of reference analytes, and need to quantify detection sensitivity/specificity or verify the >95% claim.

## When NOT to use

- No spike-in reference analytes or ground-truth feature list exists, so sensitivity/specificity cannot be computed against a truth set
- Input is already a validated feature table and the goal is downstream statistics (e.g. feature linking via FeatureLinkerUnlabeledQT or multivariate analysis), not detection-quality benchmarking
- Data is not small-molecule/metabolomics LC-MS feature detection (the algorithm targets metabolites and small molecules such as lipidomics, not protein identification)

## Inputs

- LC-MS measurement files of repeated spike-in experiments (standards in human plasma background), loadable into OpenMS
- Ground-truth/spike-in reference analyte list with known concentrations (e.g. propionyl-l-carnitine-d3 and other standard compounds)
- FeatureFinderMetabo output: feature intensities exported to a tabular text file

## Outputs

- Measured sensitivity (recall) and specificity (precision) values against the reference analytes
- Results table comparing measured vs. claimed >95% values (e.g. sensitivity_specificity_table.csv)
- Per-compound comparison of concentrations vs. feature intensities for linearity assessment

## How to apply

Load the repeated spike-in LC-MS measurement files (standards spiked into a human plasma background) into OpenMS and run FeatureFinderMetabo (OpenMS 1.10) to perform sensitive mass trace detection and assemble traces into features based on mass-to-charge spacing, co-elution information, and the SVM-based metabolite isotope pattern classifier. Score features with the hybrid mass deviation and isotope profile scoring model and export feature intensities to a tabular text file. Compare detected feature intensities against the spike-in reference analytes to compute sensitivity (fraction of true analyte signals detected with correct intensity) and specificity (fraction of reported features that are true). Verify both exceed 95%, noting the paper's finding of 97% precision and 96% recall on simulated data matched to experimental conditions; be aware the paper neither defines 'typical data' nor gives explicit cutoff or parameter values, so document your own data and parameter choices. Write a results table (e.g. sensitivity_specificity_table.csv) with rows for measured sensitivity, measured specificity, and the claimed >95% values.

## Related tools

- **FeatureFinderMetabo** (Detects mass traces and assembles them into metabolite features whose intensities are compared against the spike-in reference analytes) — github.com/OpenMS/OpenMS
- **OpenMS 1.10** (Framework in which FeatureFinderMetabo is implemented and from which features are exported to a tabular text file for comparison) — github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in plasma_spikein.mzML -out features.featureXML
```

## Evaluation signals

- Measured precision and recall both exceed 95% (paper reported 97% precision and 96% recall on simulated data matched to experimental conditions)
- Pearson correlation between compound concentrations and corresponding feature intensities shows R >= 0.98 for all standard compounds
- Known null result is reproduced: no signals detected for the first six concentrations of propionyl-l-carnitine-d3, either by the algorithm or visual inspection
- Per-measurement feature counts fall in the reported range (~2,020 ± 236 features per measurement) on comparable human plasma data
- Results table contains rows for measured sensitivity, measured specificity, and the claimed >95% values

## Limitations

- The paper does not explicitly define 'typical data' or give exact cutoff values for the >95% sensitivity/specificity claim, so reproduction requires the reproducer to choose and document data/parameters
- No explicit threshold or parameter values are reported for the sensitivity and specificity determination
- Very low-abundance analytes are a known failure mode: no signals could be detected for the first six concentrations of propionyl-l-carnitine-d3, either by the algorithm or visual inspection

## Evidence

- [discussion] The algorithm can determine feature intensities of metabolites with high sensitivity and specificity (above 95% on typical data).: "The algorithm can determine feature intensities of metabolites with high sensitivity and specificity (above 95% on typical data)."
- [results] revealed an excellent linear relationship (R≥ 0.98 for all standard compounds): "The Pearson correlation coefficient between each compound's concentrations and corresponding feature intensities revealed an excellent linear relationship (R≥ 0.98 for all standard compounds)"
- [results] In the case of the propionyl-l-carnitine-d3 standard, we could not detect any signals for the first six concentrations: "In the case of the propionyl-l-carnitine-d3 standard, we could not detect any signals for the first six concentrations"
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [readme] OpenMS is an open-source software C++ library for LC-MS data management and analyses.: [evidence span withheld — anchor status: not_found]

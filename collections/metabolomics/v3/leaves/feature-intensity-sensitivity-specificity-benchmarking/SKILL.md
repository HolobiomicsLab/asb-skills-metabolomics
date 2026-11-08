---
name: feature-intensity-sensitivity-specificity-benchmarking
description: Use when you must quantitatively validate that detected metabolite feature
  intensities are correct against known spike-in ground truth, e.g. precision/recall
  on repeated spike-in LC-MS measurements in a plasma background.
when_to_use_negative:
- Input is already a linked/quantified feature table — use feature linking (FeatureLinkerUnlabeledQT)
  and downstream statistics instead of re-running detection
- The goal is peak identification/annotation (e.g. database query against the Human
  Metabolome Database) rather than validating intensity-detection accuracy
- Data are not small-molecule LC-MS feature detection (the paper's method targets
  metabolites, lipids, peptides) — e.g. identification tasks like database search
  are out of scope
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FeatureFinderMetabo
  role: Performs sensitive mass trace detection and SVM-scored feature assembly whose
    precision/recall are being benchmarked
  repo: github.com/OpenMS/OpenMS
- name: OpenMS 1.10
  role: Framework in which FeatureFinderMetabo is implemented and executed on benchmark
    spike-in LC-MS data
  repo: github.com/OpenMS/OpenMS
- name: MapAlignmentPoseClustering
  role: Retention time correction step in the tool chain prior to comparing features
    across repeated spike-in measurements
  repo: github.com/OpenMS/OpenMS
- name: FeatureLinkerUnlabeledQT
  role: Links coinciding features from multiple experiments so intensities can be
    compared against spike-in references
  repo: github.com/OpenMS/OpenMS
- name: INIFileEditor
  role: Used to modify FeatureFinderMetabo parameters, e.g. to match experimental
    conditions when benchmarking
  repo: github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:2c94bfafa75361757c8dffc7af505eac86b74dae59372beccb2196cac36f259f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - INIFileEditor
  - MapAlignmentPoseClustering
  - OpenMS
  - OpenMS 1.10
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# feature-intensity-sensitivity-specificity-benchmarking

## Summary

Benchmark LC-MS metabolite feature-intensity determination by measuring sensitivity (recall) and specificity (precision) of FeatureFinderMetabo against ground-truth/spike-in reference analytes, verifying the paper's claim of >95% for both metrics.

## When to use

Use when you must quantitatively validate that detected metabolite feature intensities are correct against known spike-in ground truth, e.g. precision/recall on repeated spike-in LC-MS measurements in a plasma background.

## When NOT to use

- Input is already a linked/quantified feature table — use feature linking (FeatureLinkerUnlabeledQT) and downstream statistics instead of re-running detection
- The goal is peak identification/annotation (e.g. database query against the Human Metabolome Database) rather than validating intensity-detection accuracy
- Data are not small-molecule LC-MS feature detection (the paper's method targets metabolites, lipids, peptides) — e.g. identification tasks like database search are out of scope

## Inputs

- Benchmark LC-MS measurement files (repeated spike-in experiments in a human plasma background)
- Ground-truth/spike-in reference analyte list (identity and concentration of standard compounds)
- FeatureFinderMetabo INI parameter file (editable via INIFileEditor)

## Outputs

- Feature intensities exported to a tabular text file for statistical analysis
- Results table (e.g. sensitivity_specificity_table.csv) with rows for measured sensitivity, measured specificity, and the claimed >95% values

## How to apply

Run FeatureFinderMetabo (OpenMS 1.10) on the benchmark LC-MS measurements — repeated spike-in experiments in human plasma — to detect mass traces and assemble them into features using mass-to-charge spacing, co-elution information, and the SVM-based metabolite isotope-pattern classifier, scoring features with the hybrid mass deviation and isotope profile scoring model. Export the resulting feature intensities to a tabular text file via the OpenMS tool chain (feature detection, retention time correction with MapAlignmentPoseClustering, feature linking with FeatureLinkerUnlabeledQT). Match detected features to the spike-in reference analytes and compute precision (fraction of reported features that are true) and recall (fraction of true analyte signals detected with correct intensity). Judge success against the paper's benchmark: on simulated data with parameters matching experimental conditions, the algorithm reached 97% precision and 96% recall; the paper claims above 95% sensitivity and specificity on typical data but does not explicitly define 'typical data' or exact cutoffs, so report measured values alongside the claim.

## Related tools

- **FeatureFinderMetabo** (Performs sensitive mass trace detection and SVM-scored feature assembly whose precision/recall are being benchmarked) — github.com/OpenMS/OpenMS
- **OpenMS 1.10** (Framework in which FeatureFinderMetabo is implemented and executed on benchmark spike-in LC-MS data) — github.com/OpenMS/OpenMS
- **MapAlignmentPoseClustering** (Retention time correction step in the tool chain prior to comparing features across repeated spike-in measurements) — github.com/OpenMS/OpenMS
- **FeatureLinkerUnlabeledQT** (Links coinciding features from multiple experiments so intensities can be compared against spike-in references) — github.com/OpenMS/OpenMS
- **INIFileEditor** (Used to modify FeatureFinderMetabo parameters, e.g. to match experimental conditions when benchmarking) — github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in plasma_spikein.mzML -out features.featureXML
```

## Evaluation signals

- Measured precision and recall both exceed the 95% threshold reported by the paper (97% precision, 96% recall on simulated data matching experimental conditions)
- Detected feature intensities for standard compounds show linear relationship with concentration, Pearson R >= 0.98
- Average feature count per measurement falls near the reported 2,020 ± 236 features on comparable human plasma measurements
- Exported tabular output contains intensity values for each detected feature that match against spike-in reference analytes
- Known negative control: no signals detected for the first six concentrations of propionyl-l-carnitine-d3 should remain undetected (consistent with the paper)

## Limitations

- The paper does not explicitly define what constitutes 'typical data' for the >95% sensitivity/specificity claim
- No explicit threshold or exact parameter values are reported for the 95% sensitivity/specificity benchmark
- No signals could be detected for the first six concentrations of propionyl-l-carnitine-d3, either by the algorithm or visual inspection — low-abundance analytes may be missed
- The quantitative claim was demonstrated on simulated data and spike-in experiments; performance on other data types may differ

## Evidence

- [abstract] assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [discussion] The algorithm can determine feature intensities of metabolites with high sensitivity and specificity (above 95% on typical data).: "The algorithm can determine feature intensities of metabolites with high sensitivity and specificity (above 95% on typical data)."
- [discussion] Even in complex backgrounds (human plasma), we found excellent linearity of the quantification based on repeated spike-in experiments.: "Even in complex backgrounds (human plasma), we found excellent linearity of the quantification based on repeated spike-in experiments."
- [results] The Pearson correlation coefficient between each compound's concentrations and corresponding feature intensities revealed an excellent linear relationship (R>= 0.98 for all standard compounds): "The Pearson correlation coefficient between each compound's concentrations and corresponding feature intensities revealed an excellent linear relationship (R≥ 0.98 for all standard compounds)"
- [readme] It comes with a vast variety of pre-built and ready-to-use tools for proteomics and metabolomics data analysis (TOPPTools): [evidence span withheld — anchor status: external_doc]

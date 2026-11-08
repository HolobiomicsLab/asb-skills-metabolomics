---
name: svm-isotope-pattern-classification
description: Use when LC-MS feature hypotheses (groups of up to six mass traces) must
  be filtered and validated as plausible metabolite isotope patterns during feature
  assembly, e.g. when reproducing FeatureFinderMetabo's >95% sensitivity/specificity
  on human plasma data.
when_to_use_negative:
- Input is already an assembled feature table/feature map (e.g. feeding FeatureLinkerUnlabeledQT
  or MapAlignmentPoseClustering downstream) — classification happens during feature
  assembly, not after
- Data contain no expected isotope envelopes or are not small-molecule LC-MS profiles;
  the models cover CHNOPS formulas in the 1-1000 Da window
- Mass traces have not yet been detected — run sensitive mass trace detection first;
  the SVM needs trace-level m/z distances, FWHM overlap, and abundance ratios
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FeatureFinderMetabo
  role: Implements the hypothesis-driven feature assembly whose scoring function embeds
    the SVM-based metabolite isotope pattern classifier (OpenMS 1.10)
  repo: github.com/OpenMS/OpenMS
- name: OpenMS
  role: Framework providing the FeatureFinderMetabo tool, featureXML format, and INIFileEditor
    for parameter tuning
  repo: github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_005
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:bef0d3bc60c23384310e75f048cf49b8e948414cf33699605daaabfca73bec16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - OpenMS
  - OpenMS 1.10
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# svm-isotope-pattern-classification

## Summary

Classifies candidate isotope hypotheses from LC-MS mass traces as plausible metabolite isotope patterns using an SVM trained on simulated CHNOPS isotope distributions, so that FeatureFinderMetabo accepts only metabolite-like trace groups as features.

## When to use

Use when LC-MS feature hypotheses (groups of up to six mass traces) must be filtered and validated as plausible metabolite isotope patterns during feature assembly, e.g. when reproducing FeatureFinderMetabo's >95% sensitivity/specificity on human plasma data.

## When NOT to use

- Input is already an assembled feature table/feature map (e.g. feeding FeatureLinkerUnlabeledQT or MapAlignmentPoseClustering downstream) — classification happens during feature assembly, not after
- Data contain no expected isotope envelopes or are not small-molecule LC-MS profiles; the models cover CHNOPS formulas in the 1-1000 Da window
- Mass traces have not yet been detected — run sensitive mass trace detection first; the SVM needs trace-level m/z distances, FWHM overlap, and abundance ratios

## Inputs

- Mass-trace list from stage-1 mass trace detection (chromatographic apex positions, FWHM extents)
- Feature hypotheses of up to six isotopic mass traces per monoisotopic seed
- Trained SVM model built from 115,000 sampled CHNOPS sum formulas (1-1000 Da) with emass-computed isotope patterns
- Linear mu(j)/sigma(j) isotope-spacing models for j=1..5

## Outputs

- Set of accepted, non-overlapping features classified as plausible metabolite isotope patterns
- featureXML feature map with feature intensities and charge estimates (singly/doubly/triply charged)
- Tabular text export of feature intensities for downstream statistical analysis

## How to apply

Enumerate feature hypotheses H = (T0..Tk) of at most six isotopic mass traces from the mass-trace list, then score each with the N_Delta_mz error model on pairwise m/z spacings compared against linear mu(j)/sigma(j) spacing models fitted for isotope indices j=1..5. Compute co-elution between traces, requiring at least 70% FWHM overlap, and combine the m/z-spacing and elution-correlation scores into S_total using peak-area weighting. Build the SVM's four-instance feature vector from the monoisotopic mass plus the first three isotope abundance ratios; the SVM training set derives from ~115,000 randomly sampled CHNOPS sum formulas (1-1000 Da) whose theoretical isotope patterns were computed with emass, perturbed with Gaussian noise modeling 5% RMS error. Accept hypotheses greedily in descending S_total order among SVM-positive candidates, skipping any hypothesis whose traces overlap an already accepted feature, and store the resulting features (with intensities and charge estimates) in featureXML format. The paper reports this yields precision 97% and recall 96% on simulated data matched to experimental conditions.

## Related tools

- **FeatureFinderMetabo** (Implements the hypothesis-driven feature assembly whose scoring function embeds the SVM-based metabolite isotope pattern classifier (OpenMS 1.10)) — github.com/OpenMS/OpenMS
- **OpenMS** (Framework providing the FeatureFinderMetabo tool, featureXML format, and INIFileEditor for parameter tuning) — github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in sample.mzML -out features.featureXML
```

## Evaluation signals

- Every accepted feature stems from an SVM-accepted hypothesis whose traces do not overlap any other accepted feature (greedy descending S_total transfer guarantees one trace per feature)
- Output is a valid OpenMS featureXML feature map with intensities and charge estimates
- Measured precision ~97% and recall ~96% on simulated data matched to experimental conditions, meeting the reported >95% threshold
- Only hypotheses with SVM-positive four-feature vectors (monoisotopic mass + first three isotope abundance ratios) are accepted
- Co-eluting traces in each accepted hypothesis show at least 70% FWHM overlap

## Limitations

- The paper does not explicitly define 'typical data' or the exact cutoff for the above-95% sensitivity/specificity claim
- SVM training models only a 5% RMS error via Gaussian noise on abundance ratios and is restricted to CHNOPS compositions in the 1-1000 Da window sampled by HiRes-style enumeration
- Low-abundance analytes can be missed entirely: no signals were detected for the first six concentrations of propionyl-l-carnitine-d3, either by the algorithm or visual inspection

## Evidence

- [abstract] assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [discussion] An important component of the scoring function is the detection of likely metabolite profiles by means of statistical learning methods.: "An important component of the scoring function is the detection of likely metabolite profiles by means of statistical learning methods."
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"

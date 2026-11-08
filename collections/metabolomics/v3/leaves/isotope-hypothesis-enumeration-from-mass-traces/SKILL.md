---
name: isotope-hypothesis-enumeration-from-mass-traces
description: Use when you have a list of detected mass traces (with chromatographic
  apex positions and FWHM extents) from LC-MS data and need to assemble them into
  metabolite features with isotope groups and charge estimates.
when_to_use_negative:
- Input is already a feature table or linked feature map (use FeatureLinkerUnlabeledQT
  instead).
- Data are not LC-MS chromatograms of small molecules — the method was developed for
  metabolomics; the article notes it also covers other small molecules but not proteomics-scale
  peptide identification.
- No mass traces have been detected yet — run the mass trace detection stage first.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: Framework providing the FeatureFinderMetabo implementation of the isotope-hypothesis
    enumeration and feature-assembly algorithm
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: OpenMS/TOPP tool that executes the mass trace detection and isotope-hypothesis
    feature assembly pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:dfa5461d125309a0ab280dc37da54a4ffc30607cedfc075048e29dbffb673c38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - OpenMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# isotope-hypothesis-enumeration-from-mass-traces

## Summary

The feature-assembly stage of OpenMS FeatureFinderMetabo: enumerate isotope hypotheses of up to six mass traces from stage-1 mass trace detection, score them by m/z-spacing and co-elution, filter with an SVM isotope-abundance classifier, and greedily accept non-overlapping hypotheses as features.

## When to use

Use when you have a list of detected mass traces (with chromatographic apex positions and FWHM extents) from LC-MS data and need to assemble them into metabolite features with isotope groups and charge estimates.

## When NOT to use

- Input is already a feature table or linked feature map (use FeatureLinkerUnlabeledQT instead).
- Data are not LC-MS chromatograms of small molecules — the method was developed for metabolomics; the article notes it also covers other small molecules but not proteomics-scale peptide identification.
- No mass traces have been detected yet — run the mass trace detection stage first.

## Inputs

- List of mass traces from stage-1 mass trace detection (with chromatographic apex positions and FWHM extents)

## Outputs

- featureXML feature map of assembled metabolite features with intensities and charge estimates

## How to apply

Run the feature-assembly stage of FeatureFinderMetabo (OpenMS ≥ 1.10) on the mass-trace list produced by its stage-1 mass trace detection. First, enumerate isotope hypotheses H = (T0..Tk) with at most k = 5 additional mass traces per monoisotopic seed trace. Second, score each candidate hypothesis with the N_Delta_mz error model on pairwise m/z distances between traces and compute elution-correlation similarity between traces, requiring at least 70% FWHM overlap. Third, combine m/z-spacing and co-elution scores into a total score S_total using peak-area weighting, and apply the trained SVM to classify whether the hypothesis is a plausible metabolite isotope pattern. Fourth, transfer hypotheses to the accepted feature list greedily in descending S_total order, skipping any hypothesis whose mass traces overlap an already accepted feature — this guarantees no mass trace occurs in more than one feature. Finally, write the accepted features (with intensities and charge estimates) to a featureXML feature map.

## Related tools

- **OpenMS** (Framework providing the FeatureFinderMetabo implementation of the isotope-hypothesis enumeration and feature-assembly algorithm) — https://github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (OpenMS/TOPP tool that executes the mass trace detection and isotope-hypothesis feature assembly pipeline)

## Examples

```
FeatureFinderMetabo -in raw_lcms.mzML -out features.featureXML
```

## Evaluation signals

- The output featureXML must be a valid OpenMS feature map.
- Each accepted feature must stem from non-overlapping, SVM-accepted hypotheses (no mass trace occurs in more than one feature).
- Sensitivity and specificity of feature intensities should be above 95% on typical data.
- On average ~2,020 ± 236 features per measurement were detected in the benchmark plasma study.
- ~351 of those features should be singly, ~12 doubly, and ~6 triply charged.

## Limitations

- The SVM-based scoring depends on training data for isotope-abundance classification.
- The algorithm was benchmarked for metabolites and small molecules; performance on other analyte classes is not characterized in the article.
- Hypothesis enumeration is capped at six mass traces (one monoisotopic plus up to five additional), which may miss higher-charge or complex isotope patterns.
- No signals could be detected for the first six concentrations of the propionyl-l-carnitine-d3 standard, illustrating detection limits at low analyte abundance.
- The article's 95% sensitivity/specificity claim does not specify what constitutes 'typical data'.

## Evidence

- [results] Feature assembly enumeration: [evidence span withheld — anchor status: not_found]
- [results] Scoring and SVM filter: [evidence span withheld — anchor status: not_found]
- [results] Greedy non-overlapping acceptance: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract: assembly basis: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [results] Feature count benchmark: "the algorithm detected on average 2,020 ± 236 features per measurement, of which about 351 were singly, 12 doubly, and 6 triply charged"
- [abstract] Precision/recall claim: "Relative to state-of-the art algorithms, ours demonstrated increased precision and recall of the method."

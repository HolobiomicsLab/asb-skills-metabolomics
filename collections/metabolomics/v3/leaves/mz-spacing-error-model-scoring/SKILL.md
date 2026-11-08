---
name: mz-spacing-error-model-scoring
description: Use when mass traces from LC-MS metabolomics data must be grouped into
  isotope features, and candidate trace sets must be judged by whether their pairwise
  m/z distances fit a metabolite isotope-spacing error model.
when_to_use_negative:
- Input is raw, uncentroided LC-MS signal rather than an already detected list of
  mass traces — the m/z-spacing model needs trace-level masses and FWHM extents as
  input
- The analyte class is expected to have non-standard isotope envelopes (e.g., labeled
  experiments like SILAC or TMT) where metabolite isotope-spacing priors and SVM models
  trained on unlabeled metabolites do not apply
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: FeatureFinderMetabo
  role: OpenMS tool implementing the hypothesis enumeration, m/z-spacing error-model
    scoring, elution-correlation scoring, SVM filtering, and greedy deduplication
    that together assemble mass traces into features
  repo: github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:12cd603253138616030ba39ee75adaebdf7ba3e9d0bc1f4a78ad8018c2459bb1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - OpenMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# m/z-spacing error model scoring

## Summary

Scores candidate feature hypotheses by checking that the mass-to-charge spacing between isotopic mass traces matches the expected m/z-distance error model (e.g., ~1.00335 Da per 13C) and combining this with co-elution correlation before accepting a hypothesis as a metabolite feature in FeatureFinderMetabo.

## When to use

Use when mass traces from LC-MS metabolomics data must be grouped into isotope features, and candidate trace sets must be judged by whether their pairwise m/z distances fit a metabolite isotope-spacing error model.

## When NOT to use

- Input is raw, uncentroided LC-MS signal rather than an already detected list of mass traces — the m/z-spacing model needs trace-level masses and FWHM extents as input
- The analyte class is expected to have non-standard isotope envelopes (e.g., labeled experiments like SILAC or TMT) where metabolite isotope-spacing priors and SVM models trained on unlabeled metabolites do not apply

## Inputs

- List of mass traces from LC-MS centroided peak data (output of stage-1 mass trace detection, with chromatographic apex positions and FWHM extents)
- FeatureFinderMetabo parameter file (INI) defining the m/z-distance error model, elution-correlation window, and SVM thresholds

## Outputs

- Per-hypothesis scores: m/z-spacing score, elution-correlation score, combined S_total, and SVM isotope-pattern plausibility
- Set of accepted feature hypotheses (each composed of non-overlapping mass traces) ready for featureXML assembly

## How to apply

After mass trace detection, enumerate feature hypotheses of up to six isotopic mass traces anchored on a monoisotopic seed trace. For every candidate hypothesis, score each pairwise m/z distance between traces with the m/z-distance error model (N_Delta_mz), which penalizes spacings inconsistent with expected isotope shifts for metabolites. Combine these m/z-spacing scores with elution-correlation similarity (requiring at least 70% FWHM overlap between traces) into a weighted total score S_total using peak areas, then pass the combined pattern to an SVM isotope-abundance classifier that decides whether the hypothesis is a plausible metabolite isotope pattern. Only SVM-accepted hypotheses proceed to the greedy, score-ordered deduplication that guarantees no mass trace occurs in more than one feature.

## Related tools

- **FeatureFinderMetabo** (OpenMS tool implementing the hypothesis enumeration, m/z-spacing error-model scoring, elution-correlation scoring, SVM filtering, and greedy deduplication that together assemble mass traces into features) — github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -ini metabolomics.ini -in trace_list.featureXML -out assembled_features.featureXML
```

## Evaluation signals

- Every accepted feature's constituent mass traces must not overlap any other accepted feature — no mass trace occurs in more than one feature
- Accepted hypotheses must have pairwise trace spacings consistent with the trained m/z-distance error model (plausible metabolite isotope pattern, SVM-positive)
- Adjacent isotope traces must show at least 70% FWHM elution overlap
- Output featureXML must be a valid OpenMS feature map whose feature intensities and charge estimates are consistent with the summed trace peak areas
- Sensitivity and specificity of assembled features should be above 95% on typical data, as reported for the algorithm

## Limitations

- Hypotheses are limited to at most six mass traces (a monoisotopic trace plus up to five additional isotope traces)
- Requiring ≥70% FWHM co-elution means isotope traces with strongly shifted or truncated elution profiles may be missed
- The SVM isotope-pattern classifier was trained on typical metabolite isotope patterns; labeled or chemically unusual analytes may be misclassified
- No explicit thresholds or parameter values for the error model are reported in the paper, so exact tolerances must be taken from the tool's INI defaults

## Evidence

- [abstract] Feature assembly enumerates hypotheses and scores by m/z distance: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [discussion] SVM classifier scoring of isotope patterns: "An important component of the scoring function is the detection of likely metabolite profiles by means of statistical learning methods."
- [methods] 70% FWHM overlap requirement for elution correlation: [evidence span withheld — anchor status: not_found]
- [results] FeatureFinderMetabo is the implementing tool: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [discussion] Sensitivity and specificity above 95% on typical data: "The algorithm can determine feature intensities of metabolites with high sensitivity and specificity (above 95% on typical data)."

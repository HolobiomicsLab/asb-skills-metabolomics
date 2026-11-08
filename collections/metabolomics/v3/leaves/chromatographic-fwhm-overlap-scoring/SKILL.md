---
name: chromatographic-fwhm-overlap-scoring
description: Use when assembling LC-MS mass traces into metabolite isotope features
  and you must verify that candidate isotope traces co-elute with the monoisotopic
  trace before accepting a feature hypothesis.
when_to_use_negative:
- Input is already an assembled feature table (use feature linking, e.g. FeatureLinkerUnlabeledQT,
  instead)
- Traces originate from data lacking continuous chromatographic elution, where FWHM
  overlap is not defined
- Identified metabolite isotope profiles must follow a hard-coded theoretical model
  rather than learned isotope patterns — the SVM approach instead learns plausible
  metabolite isotope patterns by statistical learning
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: OpenMS
  role: open-source software framework hosting the mass trace detection and feature-assembly
    scoring
  repo: https://github.com/OpenMS/OpenMS
- name: FeatureFinderMetabo
  role: implements the hypothesis enumeration, FWHM-overlap elution scoring, SVM filtering
    and featureXML output
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:35aae017ba3775263ecd3943486967c46f3cdf77feab529fe62eaa726da9fbd9
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
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# chromatographic-fwhm-overlap-scoring

## Summary

Scores candidate isotope-hypothesis co-elution by requiring ≥70% FWHM overlap between mass traces, as part of the FeatureFinderMetabo feature-assembly scoring in OpenMS. Combining this elution-correlation term with m/z-spacing (N_Delta_mz) scoring and SVM classification prevents isotope hypotheses built from non-coeluting traces from being accepted as features.

## When to use

Use when assembling LC-MS mass traces into metabolite isotope features and you must verify that candidate isotope traces co-elute with the monoisotopic trace before accepting a feature hypothesis.

## When NOT to use

- Input is already an assembled feature table (use feature linking, e.g. FeatureLinkerUnlabeledQT, instead)
- Traces originate from data lacking continuous chromatographic elution, where FWHM overlap is not defined
- Identified metabolite isotope profiles must follow a hard-coded theoretical model rather than learned isotope patterns — the SVM approach instead learns plausible metabolite isotope patterns by statistical learning

## Inputs

- Mass-trace list from FeatureFinderMetabo stage-1 mass trace detection, with chromatographic apex positions and FWHM extents
- Trained SVM model classifying plausible metabolite isotope patterns

## Outputs

- Accepted non-overlapping feature hypotheses in OpenMS featureXML format, with intensities and charge estimates

## How to apply

After stage-1 mass trace detection returns a trace list with chromatographic apex positions and FWHM extents, enumerate isotope hypotheses of up to six traces (a monoisotopic seed plus at most k=5 additional traces). For each candidate hypothesis, score pairwise m/z distances between traces with the N_Delta_mz error model, and compute elution-correlation similarity between traces requiring at least 70% FWHM overlap. Combine the m/z-spacing and co-elution scores into a total score S_total weighted by peak area, and pass plausible hypotheses through the trained SVM metabolite-isotope-pattern classifier. Finally transfer hypotheses greedily in descending S_total order to the accepted feature list, skipping any hypothesis whose traces overlap an already accepted feature, so no mass trace occurs in more than one feature; accepted features are written with intensities and charge estimates to featureXML.

## Related tools

- **OpenMS** (open-source software framework hosting the mass trace detection and feature-assembly scoring) — https://github.com/OpenMS/OpenMS
- **FeatureFinderMetabo** (implements the hypothesis enumeration, FWHM-overlap elution scoring, SVM filtering and featureXML output)

## Examples

```
FeatureFinderMetabo -in input.mzML -out features.featureXML
```

## Evaluation signals

- Every accepted feature stems from a hypothesis whose traces show ≥70% FWHM overlap (elution correlation)
- No mass trace occurs in more than one accepted feature (hypotheses accepted greedily in descending S_total order)
- Output is a valid OpenMS featureXML feature map containing features with intensities and charge estimates
- Feature detection performance matches the paper's reported sensitivity and specificity above 95% on typical data
- Baseline sanity check from the paper: about 2,020 ± 236 features per plasma measurement, ~53% of which (1,066 ± 36) reproducible

## Limitations

- No explicit threshold or parameter values reported for the >95% sensitivity/specificity claim, and 'typical data' is not defined
- Direct comparison baseline details against XCMS/CAMERA are not fully described
- Signal intensities at very low concentrations may fall below detection: no signals were detected for the first six concentrations of propionyl-l-carnitine-d3

## Evidence

- [abstract] assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [full_text] The feature-assembly stage enumerates hypotheses of up to six mass traces, scores them by m/z-distance and elution-correlation criteria: [evidence span withheld — anchor status: not_found]
- [discussion] Model scoring with a hybrid mass deviation and isotope profile scoring results in accurately assembled features.: "Model scoring with a hybrid mass deviation and isotope profile scoring results in accurately assembled features."
- [discussion] It is based on sensitive mass trace detection and hypothesis-driven feature assembly.: "It is based on sensitive mass trace detection and hypothesis-driven feature assembly."
- [readme] OpenMS offers an infrastructure for rapid development of mass spectrometry-related software: [evidence span withheld — anchor status: external_doc]

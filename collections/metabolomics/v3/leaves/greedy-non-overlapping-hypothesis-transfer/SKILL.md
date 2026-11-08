---
name: greedy-non-overlapping-hypothesis-transfer
description: Use when LC-MS mass traces (with apex positions and FWHM extents) must
  be aggregated into non-overlapping, isotope-grouped features, and the grouping must
  resolve competing hypotheses that share traces.
when_to_use_negative:
- Input is already a feature map or feature table — this skill operates on raw mass
  traces, not aggregated features
- Traces come from peptide/protein-centric feature detection rather than small-molecule
  metabolomics; the paper scopes the method to metabolites and a wide range of small
  molecules (e.g. lipidomics, peptidomics) rather than general protein feature detection
- A one-pass grouping is acceptable and overlapping multi-analyte hypotheses need
  not be resolved
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FeatureFinderMetabo
  role: Implements mass trace detection plus hypothesis-driven feature assembly (scoring,
    SVM filtering, greedy non-overlapping hypothesis transfer) and writes the featureXML
    output
  repo: github.com/OpenMS/OpenMS
- name: OpenMS
  role: Open-source framework (version 1.10 at publication) hosting FeatureFinderMetabo
    as a TOPP tool, with TOPPView/INIFileEditor for parameter inspection
  repo: github.com/OpenMS/OpenMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:64f69bdb075bb1887fd3cd4ee045e2686e4fb40eb50b32f58fac0b7cb20409a6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureFinderMetabo
  - OpenMS
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# greedy-non-overlapping-hypothesis-transfer

## Summary

This skill converts detected mass traces into metabolite features by enumerating isotope hypotheses of up to six mass traces, scoring them by m/z-spacing and co-elution criteria, filtering them with an SVM isotope-pattern classifier, and greedily accepting hypotheses in descending total-score order so each mass trace belongs to at most one feature. It is the feature-assembly core of FeatureFinderMetabo in OpenMS.

## When to use

Use when LC-MS mass traces (with apex positions and FWHM extents) must be aggregated into non-overlapping, isotope-grouped features, and the grouping must resolve competing hypotheses that share traces.

## When NOT to use

- Input is already a feature map or feature table — this skill operates on raw mass traces, not aggregated features
- Traces come from peptide/protein-centric feature detection rather than small-molecule metabolomics; the paper scopes the method to metabolites and a wide range of small molecules (e.g. lipidomics, peptidomics) rather than general protein feature detection
- A one-pass grouping is acceptable and overlapping multi-analyte hypotheses need not be resolved

## Inputs

- mass-trace list from stage-1 mass trace detection (chromatographic apex positions and FWHM extents per trace)
- trained SVM isotope-abundance classifier
- OpenMS feature-assembly parameters (INI file, editable via INIFileEditor)

## Outputs

- featureXML feature map containing accepted features with intensities and charge estimates
- per-feature charge estimates (singly/doubly/triply charged)

## How to apply

Load the mass-trace list from stage-1 mass trace detection and enumerate candidate isotope hypotheses H = (T0..Tk), allowing at most k=5 additional traces per monoisotopic seed trace (six mass traces total). Score each pairwise m/z spacing with the N_Delta_mz error model and compute elution-correlation similarity between traces, requiring at least 70% FWHM overlap; combine m/z-spacing and co-elution scores into a total score S_total using peak-area weighting. Classify each hypothesis as a plausible metabolite isotope pattern with the trained SVM, which identifies likely metabolite profiles as an important component of the scoring function. Then transfer hypotheses to the accepted-feature list greedily in descending S_total order, skipping any hypothesis whose traces overlap an already accepted feature — this greedy, score-ordered transfer guarantees no mass trace occurs in more than one feature. Finally, write the accepted features, with intensities and charge estimates, to a featureXML feature map; the paper reports sensitivity and specificity above 95% on typical data when the full pipeline is applied.

## Related tools

- **FeatureFinderMetabo** (Implements mass trace detection plus hypothesis-driven feature assembly (scoring, SVM filtering, greedy non-overlapping hypothesis transfer) and writes the featureXML output) — github.com/OpenMS/OpenMS
- **OpenMS** (Open-source framework (version 1.10 at publication) hosting FeatureFinderMetabo as a TOPP tool, with TOPPView/INIFileEditor for parameter inspection) — github.com/OpenMS/OpenMS

## Examples

```
FeatureFinderMetabo -in raw.mzML -out features.featureXML
```

## Evaluation signals

- Output is a valid OpenMS featureXML feature map whose accepted features each stem from non-overlapping, SVM-accepted hypotheses
- No mass trace occurs in more than one accepted feature (checkable by tracing feature membership)
- Hypothesis sizes respect the cap of up to six isotopic mass traces per feature
- Each accepted hypothesis's traces show at least 70% FWHM overlap in elution
- On typical data the pipeline should reproduce the paper's high sensitivity and specificity (above 95%) and roughly 2,020 ± 236 features per measurement with ~351 singly, 12 doubly, 6 triply charged

## Limitations

- No signals could be detected for the lowest concentrations of some compounds — e.g. no signals for the first six concentrations of propionyl-l-carnitine-d3, either by the algorithm or visual inspection
- No explicit threshold or parameter values are reported for the sensitivity/specificity claim, and 'typical data' is not explicitly defined
- The paper does not give details on the statistical-learning method's features beyond 'statistical learning methods' for metabolite profile detection

## Evidence

- [abstract] assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns: "assembled into features based on mass-to-charge spacing, co-elution information, and a support vector machine-based classifier able to identify potential metabolite isotope patterns"
- [discussion] An important component of the scoring function is the detection of likely metabolite profiles by means of statistical learning methods.: "An important component of the scoring function is the detection of likely metabolite profiles by means of statistical learning methods."
- [results] implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo: "implemented and integrated in the open-source framework OpenMS (version 1.10) as the software tool FeatureFinderMetabo"
- [other] transfers hypotheses in descending total-score order to an accepted list, guaranteeing no mass trace occurs in more than one feature: [evidence span withheld — anchor status: not_found]

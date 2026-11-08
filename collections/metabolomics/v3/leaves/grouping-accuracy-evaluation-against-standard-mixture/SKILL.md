---
name: grouping-accuracy-evaluation-against-standard-mixture
description: Decision support for validating that an LC/MS feature-grouping or compound-spectrum
  extraction step correctly co-assigns features from known standards before trusting
  it on unlabeled samples.
when_to_use_negative:
- The input is raw spectra or an unprocessed peak list that has not yet been feature-detected
  and aligned by XCMS — grouping evaluation presupposes an existing feature table.
- The sample is an unlabeled biological extract with no reference compound identities,
  so no ground-truth grouping exists to compare against.
- The goal is to evaluate adduct/isotope annotation rules rather than the co-elution
  grouping itself; use CAMERA's rule-table annotation checks instead.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Provides the groupFWHM grouping routine and getPeaklist extraction used to
    generate compound-spectrum assignments that are then evaluated against the MM39
    standard mixture
  repo: https://travis-ci.org/sneumann/CAMERA
- name: XCMS
  role: Produces the aligned feature table (m/z, retention time, peak intensity) that
    is the input to the grouping method being evaluated
- name: R
  role: Implementation language in which CAMERA and the evaluation scripts run; CAMERA
    packages are available from the Bioconductor repository
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:ff9e062ef9bf03966b634c1ba9ae5783dda0c6d0ae2085720d8977c387c89e51
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# grouping-accuracy-evaluation-against-standard-mixture

## Summary

Evaluates how faithfully an LC/MS feature-grouping algorithm (e.g. CAMERA's groupFWHM) reconstructs compound spectra by comparing its group assignments against a reference mixture of known compounds, such as the 39-compound MM39 standard spanning 161–822 Da used in the CAMERA paper.

## When to use

Decision support for validating that an LC/MS feature-grouping or compound-spectrum extraction step correctly co-assigns features from known standards before trusting it on unlabeled samples.

## When NOT to use

- The input is raw spectra or an unprocessed peak list that has not yet been feature-detected and aligned by XCMS — grouping evaluation presupposes an existing feature table.
- The sample is an unlabeled biological extract with no reference compound identities, so no ground-truth grouping exists to compare against.
- The goal is to evaluate adduct/isotope annotation rules rather than the co-elution grouping itself; use CAMERA's rule-table annotation checks instead.

## Inputs

- XCMS/CAMERA feature table (m/z, retention time, peak intensity, per-sample intensities) for a standard mixture
- Reference list of known compounds in the mixture with their expected co-eluting feature sets (e.g. MM39 with sum formulas, molar masses, PubChem IDs)
- xsAnnotate object or equivalent grouping result containing pseudospectrum/compound-spectrum index per feature

## Outputs

- Per-feature table mapping each feature to its assigned compound-spectrum index (e.g. via getPeaklist)
- Agreement metrics quantifying grouping accuracy against the reference compound grouping (e.g. fraction of standards whose features are correctly co-assigned)
- Saved assignment table and metrics file (e.g. annotations_fwhm.csv) for cross-method comparison

## How to apply

Run the candidate grouping method (e.g. CAMERA's groupFWHM, which iteratively picks the most intense unassigned feature and windows retention time at ~60% of that feature's chromatographic peak FWHM) on the LC/MS feature table of a reference mixture such as MM39, the 39 known compounds covering 161–822 Da. Extract per-feature group assignments (via getPeaklist in CAMERA) and compare them against the expected grouping where each known standard's co-eluting features define one compound spectrum. Quantify agreement—for example, counting whether all features of a compound fall into one pseudospectrum and whether unrelated features are erroneously merged—and report metrics such as the fraction of compounds with correctly reconstructed spectra (89.7% in positive ion mode and 90.3% in negative ion mode for CAMERA on MM39 in this paper). Record assignment tables and agreement metrics for downstream comparison across grouping variants.

## Related tools

- **CAMERA** (Provides the groupFWHM grouping routine and getPeaklist extraction used to generate compound-spectrum assignments that are then evaluated against the MM39 standard mixture) — https://travis-ci.org/sneumann/CAMERA
- **XCMS** (Produces the aligned feature table (m/z, retention time, peak intensity) that is the input to the grouping method being evaluated)
- **R** (Implementation language in which CAMERA and the evaluation scripts run; CAMERA packages are available from the Bioconductor repository)

## Examples

```
R: xs <- xcmsSet(faahKO_files); xsAnno <- xsAnnotate(xs); xsAnno <- groupFWHM(xsAnno); pl <- getPeaklist(xsAnno); write.csv(pl, 'annotations_fwhm.csv')
```

## Evaluation signals

- Each known standard compound's features should co-locate within a single pseudospectrum/compound-spectrum index in the getPeaklist output.
- The fraction of standards with correctly reconstructed compound spectra should be reported and compared against reference values (89.7% positive ion mode, 90.3% negative ion mode for CAMERA on MM39).
- No features from unrelated standards should be merged into the same compound spectrum (no cross-compound false merges).
- The saved annotations file (e.g. annotations_fwhm.csv) must contain a per-feature assignment column plus a compound-spectrum index and agreement metrics.
- Grouping must assign every feature; the iteration should terminate with no unassigned features remaining.

## Limitations

- Accuracy depends on the quality of the reference standard list; compounds not in the mixture cannot be validated.
- Grouping heuristics like the 60%-FWHM retention-time window can mis-assign features when chromatographic peak shapes are atypical or when features from different compounds have overlapping retention windows.
- Evaluation on a single mixture (e.g. MM39) may not generalize to complex biological matrices with many more overlapping features.
- Detector saturation or extreme peak-shape truncation can distort FWHM estimates and thus grouping windows.

## Evidence

- [methods] groupFWHM grouping mechanism: [evidence span withheld — anchor status: not_found]

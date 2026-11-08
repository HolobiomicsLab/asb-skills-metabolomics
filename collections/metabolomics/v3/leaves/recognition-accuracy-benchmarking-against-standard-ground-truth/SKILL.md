---
name: recognition-accuracy-benchmarking-against-standard-ground-truth
description: Use when you need to quantify how accurately an ISF-detection pipeline
  assigns ISF levels to features, and you have metabolite-standard injections that
  provide known ISF relationships as ground truth.
when_to_use_negative:
- You have only complex biological samples (e.g. plasma) with no metabolite standards,
  so no ground-truth ISF assignments exist to score against
- Your goal is ISF discovery on new data rather than validating recognition accuracy
  of the pipeline itself
- Your data lacks the DDA mzXML files needed for ms2.assignment() and feature.annotation()
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ISFrag
  role: R package whose find.level3/find.level2/find.level1 cascade produces the ISF
    level assignments being benchmarked
  repo: https://github.com/HuanLab/ISFrag
- name: xcms
  role: Extracts the MS1 feature table from metabolite-standard fullscan mzXML files
    before ISF identification
- name: MSnbase
  role: Provides the MS data object output alongside the feature table by generate.featuretable()
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:c4e15f9240cce260fcf04f80a65c43378b7235aa0c6c3d86e699d77dfea40cd2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag
  tools:
  - ISFrag
  - MSnbase
  - xcms
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# recognition-accuracy-benchmarking-against-standard-ground-truth

## Summary

Benchmarks the accuracy with which the ISFrag pipeline (find.level3 → find.level2 → find.level1) recognizes in-source fragment (ISF) features by running the full cascade on metabolite-standard LC-MS data and scoring annotated ISF levels against a ground-truth standard list.

## When to use

Use when you need to quantify how accurately an ISF-detection pipeline assigns ISF levels to features, and you have metabolite-standard injections that provide known ISF relationships as ground truth.

## When NOT to use

- You have only complex biological samples (e.g. plasma) with no metabolite standards, so no ground-truth ISF assignments exist to score against
- Your goal is ISF discovery on new data rather than validating recognition accuracy of the pipeline itself
- Your data lacks the DDA mzXML files needed for ms2.assignment() and feature.annotation()

## Inputs

- Metabolite-standard MS1 feature table (csv with columns mz, rt, rtmin, rtmax, intensity in seconds)
- DDA mzXML files from metabolite-standard injections
- MoNA negative-mode spectral library (MoNA-export-LC-MS-MS_Negative_Mode.msp)
- Ground-truth list of known ISF relationships among the injected standards

## Outputs

- Per-level recognition accuracy rates (correctly recognized ISF features / total ground-truth ISF features per level)
- Annotated feature table with ISF level assignments (export.ISFrag.results())
- Detailed per-precursor ISF relationship tables (export.ISFrag.detailed())

## How to apply

Run the complete ISFrag cascade — ms2.assignment(), feature.annotation() against MoNA-export-LC-MS-MS_Negative_Mode.msp (dot-product score cutoff 0.1), then find.level3(), find.level2(), and find.level1() — on the metabolite-standard MS1 feature table (columns mz, rt, rtmin, rtmax, intensity) and DDA mzXML files. Summarize with get.ISFrag.results() and export via export.ISFrag.results() and export.ISFrag.detailed(). Compare each annotated ISF level assignment against the known ISF relationships of the injected standards, counting a feature as correctly recognized only if its assigned level matches the ground truth. The paper's reference benchmark is 100% correct recognition of level 1 ISF features and over 80% for level 2 ISF features; treat results within tolerance of these rates as a successful reproduction.

## Related tools

- **ISFrag** (R package whose find.level3/find.level2/find.level1 cascade produces the ISF level assignments being benchmarked) — https://github.com/HuanLab/ISFrag
- **xcms** (Extracts the MS1 feature table from metabolite-standard fullscan mzXML files before ISF identification)
- **MSnbase** (Provides the MS data object output alongside the feature table by generate.featuretable())

## Examples

```
install_github("HuanLab/ISFrag"); library(ISFrag); ft <- custom.featuretable(ft_directory, ft_name); ms2.assignment(); find.level3(); find.level2(); find.level1(); get.ISFrag.results()
```

## Evaluation signals

- Level 1 ISF recognition rate equals 100% on the metabolite-standard test data
- Level 2 ISF recognition rate exceeds 80% on the metabolite-standard test data
- Every ground-truth ISF feature from the standards appears in the exported feature table with an assigned ISF level
- get.ISFrag.results() output counts are consistent with the detailed per-precursor tables from export.ISFrag.detailed()

## Limitations

- The >80% level 2 accuracy benchmark indicates level 2 ISF recognition is imperfect; some ground-truth level 2 fragments will be missed or mis-leveled
- Accuracy was established only on metabolite-standard test data; performance on complex mixtures with unknown ISF relationships is not benchmarked this way
- MS annotation depends on the MoNA negative-mode msp library with a dot-product cutoff of 0.1, so standards absent from the library cannot be validated via MS2 annotation

## Evidence

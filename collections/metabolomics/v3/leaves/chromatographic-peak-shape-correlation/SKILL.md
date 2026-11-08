---
name: chromatographic-peak-shape-correlation
description: Use when processing untargeted LC-MS raw data (mzML or Bruker .d) where
  co-eluting ions must be distinguished as genuine metabolites versus in-source fragments
  or alternative adducts before quantification.
when_to_use_negative:
- The parent ion was not detected due to complete fragmentation in the ion source
  — MassCube cannot automatically annotate its ISFs in this case
- Input is already an annotated feature table or you only need metabolite identification
  — MassCube does not directly perform metabolite identification and this skill covers
  grouping/annotation of features, not ID
- A graphical interface or time-series/SERRF normalization is needed — these are not
  available in the current MassCube version
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: Python framework performing peak detection, aligned feature grouping, and
    the two-criterion ISF/adduct annotation via scan-to-scan Pearson peak-shape correlation
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Implementation language; MassCube is installed via pip and invoked as a Python
    library (e.g., read_raw_file_to_obj, untargeted_metabolomics_workflow)
  repo: https://github.com/huaxuyu/masscube/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:995e8986e5800fa326a167649b55d08f36c49e9e4843c741df0f6b8c60762d0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# chromatographic-peak-shape-correlation

## Summary

Annotates in-source fragments (ISFs) and co-eluting adduct/isotope groups in LC-MS data by requiring a candidate fragment's m/z to appear in the parent's MS/MS spectrum AND its scan-to-scan Pearson correlation with the parent peak shape to exceed r > 0.7. This two-criterion strategy is how MassCube grouped 2604 ISFs and 6055 alternative adducts in the Metabolome Atlas of the Aging Mouse Brain dataset.

## When to use

Use when processing untargeted LC-MS raw data (mzML or Bruker .d) where co-eluting ions must be distinguished as genuine metabolites versus in-source fragments or alternative adducts before quantification.

## When NOT to use

- The parent ion was not detected due to complete fragmentation in the ion source — MassCube cannot automatically annotate its ISFs in this case
- Input is already an annotated feature table or you only need metabolite identification — MassCube does not directly perform metabolite identification and this skill covers grouping/annotation of features, not ID
- A graphical interface or time-series/SERRF normalization is needed — these are not available in the current MassCube version

## Inputs

- Raw LC-MS files (mzML or supported Bruker timsTOF .d TDF2 DDA-PASEF) from the Metabolome Atlas of the Aging Mouse Brain or similar untargeted metabolomics studies
- Versioned .mcraw workflow cache under the project tmp directory (reloaded instead of reparsing source)
- Aligned feature table with MS/MS spectra and per-scan peak shapes

## Outputs

- feature_grouping_results.csv with ISF and adduct annotations per aligned feature group
- Annotation tallies (expected: 2604 ISFs and 6055 alternative adducts for the mouse brain dataset)
- Optional chromatographic peak shapes when Params.output_peak_shape = True

## How to apply

After running MassCube's untargeted_metabolomics_workflow (feature detection, gap filling, alignment, reloading from the versioned .mcraw cache), perform aligned feature grouping, first grouping isotope features within each aligned feature group. For each candidate in-source fragment, apply two criteria: (1) the fragment's m/z must be present in the parent ion's MS/MS spectrum, and (2) the scan-to-scan Pearson correlation between the fragment's peak shape and the parent's peak shape must exceed r > 0.7. Only candidates passing both tests are annotated as ISFs; remaining grouped features are then annotated as adducts to complete adduct grouping including in-source fragments. The rationale is that a true ISF co-elutes perfectly with its parent and is supported by fragment evidence in MS/MS, whereas unrelated co-eluting metabolites fail the peak-shape correlation. Correct ISF annotation was demonstrated for D5-glutamine, D5-glutamic acid and endogenous metabolites.

## Related tools

- **MassCube** (Python framework performing peak detection, aligned feature grouping, and the two-criterion ISF/adduct annotation via scan-to-scan Pearson peak-shape correlation) — https://github.com/huaxuyu/masscube
- **Python** (Implementation language; MassCube is installed via pip and invoked as a Python library (e.g., read_raw_file_to_obj, untargeted_metabolomics_workflow)) — https://github.com/huaxuyu/masscube/

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('sample.mzML')  # then run untargeted_metabolomics_workflow; grouped output -> feature_grouping_results.csv
```

## Evaluation signals

- Reproduce the reported tallies on the Metabolome Atlas of the Aging Mouse Brain data: 2604 ISFs and 6055 alternative adducts combined into unique peak groups in feature_grouping_results.csv
- Spot-check known tracers: ISF annotation must be correct for D5-glutamine, D5-glutamic acid and the endogenous metabolites
- Every annotated ISF must satisfy both criteria: fragment m/z present in the parent's MS/MS spectrum AND scan-to-scan Pearson correlation r > 0.7 with the parent peak shape
- Isotope features are grouped within each aligned feature group before ISF/adduct annotation is applied
- Single-file output should omit full chromatographic traces unless Params.output_peak_shape = True was explicitly set

## Limitations

- ISFs cannot be automatically annotated when the parent ion is not detected due to complete fragmentation in the ion source
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios
- No graphical user interface is available in the current version of MassCube
- The .mcraw cache stores derived data; the original vendor data should still be retained as the archival source

## Evidence

- [methods] ISF two-criterion rule: [evidence span withheld — anchor status: not_found]
- [results] Mouse-brain dataset tallies: [evidence span withheld — anchor status: not_found]
- [abstract] Correlation-based grouping strategy: [evidence span withheld — anchor status: not_found]
- [discussion] Complete-fragmentation failure mode: "if the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated using this approach."
- [readme] mcraw cache and output_peak_shape behavior: "Single-file TXT output keeps the `peak_shape` column for compatibility but does not embed full chromatographic traces by default. Set `Params.output_peak_shape = True` when those traces are"

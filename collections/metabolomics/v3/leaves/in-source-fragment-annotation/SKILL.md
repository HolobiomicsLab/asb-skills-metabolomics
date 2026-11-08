---
name: in-source-fragment-annotation
description: Use when processing raw LC-MS (mzML or Bruker .d) nontargeted metabolomics
  data and you need to collapse features into groups annotated as isotopes, adducts,
  or in-source fragments to remove false-positive features.
when_to_use_negative:
- The parent ion is not detected because of complete fragmentation in the ion source
  — MassCube cannot automatically annotate such ISFs
- Input is already a curated feature table lacking MS/MS spectra and scan-level peak
  shapes, since both ISF criteria (parent MS/MS membership and scan-to-scan Pearson
  correlation) require scan-level data
- The goal is direct metabolite identification — MassCube does not directly perform
  metabolite identification, only feature-group annotation
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: Python framework providing read_raw_file_to_obj, the untargeted metabolomics
    workflow, the .mcraw cache, and the peak grouping module used for isotope, ISF,
    and adduct annotation
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Language in which MassCube is implemented and in which the grouping workflow
    is executed
  repo: https://github.com/huaxuyu/masscube/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:9b0b3eeabd55465a30baf23de2c8d1826e210b4bf492502a1d019c154242fa7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# in-source-fragment-annotation

## Summary

Annotates in-source fragments (ISFs) and alternative adducts among co-eluting LC-MS features using MassCube's peak grouping module, which combines MS/MS-spectrum membership with scan-to-scan peak-shape correlation. Applied to the Metabolome Atlas of the Aging Mouse Brain it reports 2604 ISFs and 6055 alternative adducts merged into unique peak groups.

## When to use

Use when processing raw LC-MS (mzML or Bruker .d) nontargeted metabolomics data and you need to collapse features into groups annotated as isotopes, adducts, or in-source fragments to remove false-positive features.

## When NOT to use

- The parent ion is not detected because of complete fragmentation in the ion source — MassCube cannot automatically annotate such ISFs
- Input is already a curated feature table lacking MS/MS spectra and scan-level peak shapes, since both ISF criteria (parent MS/MS membership and scan-to-scan Pearson correlation) require scan-level data
- The goal is direct metabolite identification — MassCube does not directly perform metabolite identification, only feature-group annotation

## Inputs

- Raw LC-MS/MS data files (mzML or supported Bruker .d, e.g. TDF2 DDA-PASEF)
- MassCube data objects created via read_raw_file_to_obj
- MassCube .mcraw versioned workflow cache under the project tmp directory
- Aligned feature table from MassCube's untargeted metabolomics workflow

## Outputs

- feature_grouping_results.csv with features annotated as isotopes, in-source fragments, and alternative adducts
- Counts of ISFs (e.g. 2604) and alternative adducts (e.g. 6055) collapsed into unique peak groups for the Metabolome Atlas of the Aging Mouse Brain dataset

## How to apply

Load raw LC-MS files with MassCube's read_raw_file_to_obj and run the untargeted_metabolomics_workflow through feature detection, gap filling, and alignment, reloading from the versioned .mcraw cache under the project tmp directory instead of reparsing the source. Run the aligned feature grouping module, first grouping isotope features within each aligned feature group. Annotate in-source fragments using two criteria: the candidate fragment's m/z must be present in the parent ion's MS/MS spectrum, and the scan-to-scan Pearson correlation between the fragment's peak shape and the parent's peak shape must exceed r > 0.7. Finally annotate adducts among the remaining grouped features to complete adduct grouping including in-source fragments, and export the single-file feature_grouping_results.csv (set Params.output_peak_shape = True only if full chromatographic traces are needed).

## Related tools

- **MassCube** (Python framework providing read_raw_file_to_obj, the untargeted metabolomics workflow, the .mcraw cache, and the peak grouping module used for isotope, ISF, and adduct annotation) — https://github.com/huaxuyu/masscube
- **Python** (Language in which MassCube is implemented and in which the grouping workflow is executed) — https://github.com/huaxuyu/masscube/

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj("sample.d")
```

## Evaluation signals

- Reproduce the reported tallies on the Metabolome Atlas of the Aging Mouse Brain data: 2604 ISFs and 6055 alternative adducts combined into unique peak groups in feature_grouping_results.csv
- Every annotated ISF satisfies both criteria: its m/z appears in the parent ion's MS/MS spectrum and its scan-to-scan Pearson correlation with the parent peak shape exceeds r > 0.7
- ISF grouping correctly annotates known cases such as D5-glutamine, D5-glutamic acid, and endogenous metabolites
- Isotope grouping completes before ISF and adduct annotation within each aligned feature group
- Single-file TXT output contains no full chromatographic traces unless Params.output_peak_shape = True was explicitly set

## Limitations

- ISFs cannot be automatically annotated when the parent ion is not detected due to complete fragmentation in the ion source
- MassCube does not directly perform metabolite identification; it enhances annotation accuracy but downstream databases are needed for interpretation
- Human-defined rules for peak shapes in algorithmic parameters cannot account for all peak-shape scenarios

## Evidence

- [methods] ISF two-criteria method: [evidence span withheld — anchor status: not_found]
- [abstract] Peak grouping role in pipeline: "Peaks are then grouped for adduct and in-source fragment detection"
- [methods] mcraw cache reload: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [discussion] Complete-fragmentation failure mode: "if the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated using this approach."
- [readme] README capability statement: "Confident annotation of feature groups including isotopes, adducts and in-source fragments."

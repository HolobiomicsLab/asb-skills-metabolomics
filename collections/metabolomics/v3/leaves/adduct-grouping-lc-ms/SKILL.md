---
name: adduct-grouping-lc-ms
description: Use when processing nontargeted LC-MS data (mzML or Bruker .d) where
  many features are co-eluting ions of the same metabolite (isotopes, adducts, ISFs)
  and a deduplicated, annotated feature table is needed.
when_to_use_negative:
- Input is already a curated feature table lacking raw MS/MS spectra — the ISF criterion
  requires each candidate fragment's m/z to be present in the parent ion's MS/MS spectrum
- Parent ions were completely fragmented in the ion source so no parent peak is detected
  — MassCube cannot automatically annotate their ISFs
- Downstream metabolite identification is the goal — MassCube grouping annotates feature
  relationships but does not directly perform metabolite identification
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: Python package providing the peak grouping module for isotope, adduct, and
    in-source fragment annotation
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime environment in which MassCube's grouping workflow and data objects
    are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:3366f1000bc07d9999eaf4f1378d86cfe1f6a47e1891fdc8aa8f08b96ac42d97
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# adduct-grouping-lc-ms

## Summary

Annotates in-source fragments (ISFs) and alternative adducts within aligned LC-MS feature groups using MassCube's peak grouping module, collapsing redundant ions into unique peak groups.

## When to use

Use when processing nontargeted LC-MS data (mzML or Bruker .d) where many features are co-eluting ions of the same metabolite (isotopes, adducts, ISFs) and a deduplicated, annotated feature table is needed.

## When NOT to use

- Input is already a curated feature table lacking raw MS/MS spectra — the ISF criterion requires each candidate fragment's m/z to be present in the parent ion's MS/MS spectrum
- Parent ions were completely fragmented in the ion source so no parent peak is detected — MassCube cannot automatically annotate their ISFs
- Downstream metabolite identification is the goal — MassCube grouping annotates feature relationships but does not directly perform metabolite identification

## Inputs

- Aligned feature groups from MassCube's untargeted metabolomics workflow (after feature detection, gap filling, and alignment) reloaded from the .mcraw cache
- Raw LC-MS data in mzML or Bruker timsTOF .d (TDF2, DDA-PASEF) format
- MS/MS spectra associated with parent ions

## Outputs

- Grouped feature table with ISF and adduct annotations (feature_grouping_results.csv)
- Counts of in-source fragments and alternative adducts collapsed into unique peak groups

## How to apply

After running MassCube's untargeted_metabolomics_workflow (feature detection, gap filling, chromatographic alignment) and reloading the versioned .mcraw cache from the project tmp directory, apply the aligned feature grouping module, first grouping isotope features within each aligned feature group. Then annotate ISFs using two criteria: the candidate fragment's m/z must be present in the parent ion's MS/MS spectrum, and the scan-to-scan Pearson correlation between the fragment's and parent's peak shapes must exceed r > 0.7. Finally annotate adducts among the remaining grouped features to complete adduct grouping including in-source fragments. Export the grouped feature table (single-file output; set Params.output_peak_shape = True only if full chromatographic traces are needed) and record ISF/adduct tallies.

## Related tools

- **MassCube** (Python package providing the peak grouping module for isotope, adduct, and in-source fragment annotation) — https://github.com/huaxuyu/masscube
- **Python** (Runtime environment in which MassCube's grouping workflow and data objects are executed)

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj('sample.d'); # then run untargeted_metabolomics_workflow and the peak grouping module on aligned features
```

## Evaluation signals

- Applied to the Metabolome Atlas of the Aging Mouse Brain dataset, the run reproduces the reported 2604 ISFs and 6055 alternative adducts in the exported feature_grouping_results.csv
- Every annotated ISF satisfies both criteria: fragment m/z present in the parent's MS/MS spectrum and scan-to-scan Pearson correlation of peak shapes r > 0.7
- Isotope features are grouped within each aligned feature group before ISF/adduct annotation is attempted
- Single-file TXT output retains the peak_shape column; full traces appear only when Params.output_peak_shape = True
- Validation on D5-glutamine, D5-glutamic acid and endogenous metabolites: ISFs correctly annotated as in the published mouse-brain run

## Limitations

- If the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated
- No GUI is available in the current version of MassCube
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios

## Evidence

- [other] MassCube groups ISFs by checking that a potential ISF ion also appears in the parent MS/MS spectrum and by examining scan-to-scan Pearson correlations between co-eluting ions: [evidence span withheld — anchor status: not_found]
- [other] Applied to the mouse brain data, MassCube detected 2604 ISFs and 6055 alternative adducts combined into unique peak groups: [evidence span withheld — anchor status: not_found]
- [discussion] if the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated using this approach: "if the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated using this approach."
- [readme] Single-file TXT output keeps the peak_shape column for compatibility but does not embed full chromatographic traces by default: "Single-file TXT output keeps the `peak_shape` column for compatibility but does not embed full chromatographic traces by default. Set `Params.output_peak_shape = True`"
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source: "Feature detection, gap filling, and aligned feature grouping then reload this
cache instead of reparsing the source."

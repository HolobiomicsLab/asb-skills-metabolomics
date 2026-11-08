---
name: metabolomics-workflow-execution
description: Use when processing raw nontargeted LC-MS data (mzML or Bruker .d) from
  acquisition to a grouped, ISF-annotated feature table, e.g. when reproducing adduct/ISF
  grouping counts on the Metabolome Atlas mouse-brain dataset.
when_to_use_negative:
- Input is already a processed feature table — MassCube's workflow starts from raw
  vendor files, not from downstream peak tables
- The parent ion of an in-source fragment was completely fragmented in the source
  and not detected — MassCube cannot automatically annotate such ISFs
- A graphical interface or metabolite identification is required — MassCube has no
  GUI and does not directly perform metabolite identification
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: 'Python framework executing the untargeted metabolomics workflow: peak detection,
    gap filling, alignment, and isotope/adduct/ISF grouping'
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime language for the MassCube API (read_raw_file_to_obj, untargeted_metabolomics_workflow,
    Params settings)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:5c792abaaa56a889a166fbcbf979dbfbfc6d8f0461992d22fbc3d16a906c75ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MassCube
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolomics-workflow-execution

## Summary

Run MassCube's end-to-end untargeted LC-MS metabolomics workflow — raw-file import, peak detection, gap filling, alignment, and isotope/adduct/ISF grouping — producing a single grouped feature table with tallies such as the 2604 ISFs and 6055 alternative adducts reported for the Metabolome Atlas of the Aging Mouse Brain data.

## When to use

Use when processing raw nontargeted LC-MS data (mzML or Bruker .d) from acquisition to a grouped, ISF-annotated feature table, e.g. when reproducing adduct/ISF grouping counts on the Metabolome Atlas mouse-brain dataset.

## When NOT to use

- Input is already a processed feature table — MassCube's workflow starts from raw vendor files, not from downstream peak tables
- The parent ion of an in-source fragment was completely fragmented in the source and not detected — MassCube cannot automatically annotate such ISFs
- A graphical interface or metabolite identification is required — MassCube has no GUI and does not directly perform metabolite identification

## Inputs

- Raw LC-MS/MS data files (mzML or Bruker TDF2 DDA-PASEF .d directories)
- Metabolome Atlas of the Aging Mouse Brain raw files (for the published reproduction target)

## Outputs

- feature_grouping_results.csv with grouped isotope/adduct/ISF annotations (expected: 2604 ISFs and 6055 alternative adducts for the mouse-brain dataset)
- Versioned .mcraw cache under the project tmp directory containing decoded MS1/MS2 scans and peak arrays
- Feature table with detection, gap-filling, alignment, and grouping results

## How to apply

Install masscube (pip install masscube) and load each raw LC-MS file (mzML or Bruker TDF2 .d) with read_raw_file_to_obj, then run the untargeted_metabolomics_workflow through feature detection (mass traces built by signal clustering and Gaussian-filter assisted edge detection), gap filling, and alignment. The workflow decodes each source once and caches it as a versioned .mcraw object under the project tmp directory, so later stages reload the cache instead of reparsing vendor files. Next, perform aligned feature grouping: first group isotope features within each aligned feature group, then annotate in-source fragments using the two ISF criteria — the candidate fragment's m/z must appear in the parent ion's MS/MS spectrum, and scan-to-scan Pearson correlation between the fragment and parent peak shapes must exceed r > 0.7 — and finally group adducts among remaining features. Export the grouped feature table (single-file output; set Params.output_peak_shape = True only if full chromatographic traces are needed) and record ISF and adduct tallies for validation against published counts.

## Related tools

- **MassCube** (Python framework executing the untargeted metabolomics workflow: peak detection, gap filling, alignment, and isotope/adduct/ISF grouping) — https://github.com/huaxuyu/masscube
- **Python** (Runtime language for the MassCube API (read_raw_file_to_obj, untargeted_metabolomics_workflow, Params settings))

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj("sample.d")
```

## Evaluation signals

- Exported feature_grouping_results.csv reports 2604 ISFs and 6055 alternative adducts for the Metabolome Atlas of the Aging Mouse Brain dataset
- ISF annotations satisfy both criteria: fragment m/z present in the parent's MS/MS spectrum and scan-to-scan Pearson correlation r > 0.7 with the parent peak shape
- The .mcraw cache is created under the project tmp directory and reused by gap filling and aligned feature grouping instead of reparsing vendor files
- Feature detection covers 100% of signals, as claimed for MassCube's peak detection
- Single-file output retains the peak_shape column but does not embed full traces unless Params.output_peak_shape = True

## Limitations

- ISFs cannot be automatically annotated when the parent ion is not detected due to complete fragmentation in the ion source
- No graphical user interface is available in the current version of MassCube
- Single-file TXT output does not embed full chromatographic traces by default; enabling Params.output_peak_shape inflates output only when traces are explicitly needed
- MassCube does not directly perform metabolite identification; downstream biological or clinical experiments remain necessary to confirm biological function

## Evidence

- [abstract] Peaks are then grouped for adduct and in-source fragment detection: "Peaks are then grouped for adduct and in-source fragment detection"
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [readme] The untargeted workflow decodes each mzML or supported Bruker `.d` source once and stores a versioned `.mcraw` cache under the project `tmp` directory.: "The untargeted workflow decodes each mzML or supported Bruker `.d` source once and stores a versioned `.mcraw` cache under the project `tmp` directory."
- [discussion] ISFs cannot be automatically annotated when the parent ion is not detected due to complete fragmentation in the ion source: "if the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated using this approach."

---
name: lc-ms-isotope-feature-grouping
description: Use when raw LC-MS data (mzML or Bruker .d) has been feature-detected
  and aligned, and isotope/adduct/ISF peak groups must be annotated to remove false-positive
  features before statistics.
when_to_use_negative:
- The parent ion is not detected because it is completely fragmented in the ion source
  — its ISFs cannot be automatically annotated with this approach
- Input is already a clean feature table without raw scan-level data or parent MS/MS
  spectra, so neither the MS/MS-presence nor the r > 0.7 peak-shape criterion can
  be evaluated
- You need metabolite identification rather than feature grouping — MassCube does
  not directly perform metabolite identification
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MassCube
  role: Python package whose peak grouping module performs isotope grouping, ISF annotation
    via MS/MS-presence and scan-to-scan Pearson correlation (r > 0.7), and adduct
    annotation on aligned features
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime language in which MassCube is installed (pip install masscube) and
    the grouping workflow is invoked
  repo: https://github.com/huaxuyu/masscube/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:78f87165085a3faaf1ead311f09897749475969510b699901e88c266f0384521
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# lc-ms-isotope-feature-grouping

## Summary

Group LC-MS aligned features into isotope, adduct, and in-source fragment (ISF) peak groups using MassCube's peak grouping module, which validates ISF candidates against the parent's MS/MS spectrum and co-elution peak-shape correlation.

## When to use

Use when raw LC-MS data (mzML or Bruker .d) has been feature-detected and aligned, and isotope/adduct/ISF peak groups must be annotated to remove false-positive features before statistics.

## When NOT to use

- The parent ion is not detected because it is completely fragmented in the ion source — its ISFs cannot be automatically annotated with this approach
- Input is already a clean feature table without raw scan-level data or parent MS/MS spectra, so neither the MS/MS-presence nor the r > 0.7 peak-shape criterion can be evaluated
- You need metabolite identification rather than feature grouping — MassCube does not directly perform metabolite identification

## Inputs

- Aligned LC-MS feature table from MassCube's untargeted metabolomics workflow (post feature detection, gap filling, and chromatographic alignment)
- Raw LC-MS files: mzML or Bruker timsTOF .d (TDF2, DDA-PASEF)
- MS/MS spectra of parent ions for ISF spectrum-matching
- Versioned .mcraw workflow cache under the project tmp directory

## Outputs

- Grouped feature table with isotope, adduct, and ISF annotations (exported as feature_grouping_results.csv)
- Counts of annotated ISFs and alternative adducts collapsed into unique peak groups
- Optional peak_shape column with full chromatographic traces (only if Params.output_peak_shape = True)

## How to apply

After running MassCube's untargeted_metabolomics_workflow (feature detection, gap filling, and alignment, reloading from the versioned .mcraw cache under the project tmp directory), call the peak grouping module on aligned feature groups. First group isotope features within each aligned feature group, then annotate in-source fragments using two criteria: the candidate fragment's m/z must be present in the parent ion's MS/MS spectrum, and the scan-to-scan Pearson correlation between fragment and parent peak shapes must exceed r > 0.7. Finally annotate adducts among the remaining grouped features to complete adduct grouping including ISFs. The rationale is that XCMS, MS-DIAL, MZmine, El-MAVEN, OpenMS, and TidyMass do not directly annotate ISFs and often report many false-positive features; MassCube's two-criteria ISF check correctly annotated D5-glutamine, D5-glutamic acid, and endogenous metabolites.

## Related tools

- **MassCube** (Python package whose peak grouping module performs isotope grouping, ISF annotation via MS/MS-presence and scan-to-scan Pearson correlation (r > 0.7), and adduct annotation on aligned features) — https://github.com/huaxuyu/masscube
- **Python** (Runtime language in which MassCube is installed (pip install masscube) and the grouping workflow is invoked) — https://github.com/huaxuyu/masscube/

## Examples

```
from masscube import read_raw_file_to_obj; data = read_raw_file_to_obj("sample.d"); scan = data.scans[data.ms1_idx_arr[0]]
```

## Evaluation signals

- On the Metabolome Atlas of the Aging Mouse Brain data, the exported feature_grouping_results.csv reproduces 2604 annotated ISFs and 6055 alternative adducts combined into unique peak groups
- Every annotated ISF satisfies both criteria: its m/z is present in the parent ion's MS/MS spectrum and its scan-to-scan Pearson correlation with the parent's peak shape exceeds r > 0.7
- Correct annotation of spiked tracers D5-glutamine and D5-glutamic acid alongside endogenous metabolites
- Isotope features are grouped first within each aligned feature group before ISF and adduct annotation is applied to remaining features
- Single-file output keeps the peak_shape column empty of full traces unless Params.output_peak_shape = True was explicitly set

## Limitations

- ISFs cannot be automatically annotated when the parent ion is not detected due to complete fragmentation in the ion source
- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios
- No graphical user interface is available in the current version of MassCube

## Evidence

- [abstract] Peaks are then grouped for adduct and in-source fragment detection: "Peaks are then grouped for adduct and in-source fragment detection"
- [methods] Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source.: "Feature detection, gap filling, and aligned feature grouping then reload this cache instead of reparsing the source."
- [readme] Confident annotation of feature groups including isotopes, adducts and in-source fragments.: "Confident annotation of feature groups including isotopes, adducts and in-source fragments."
- [readme] The untargeted workflow decodes each mzML or supported Bruker `.d` source once and stores a versioned `.mcraw` cache under the project `tmp` directory.: "The untargeted workflow decodes each mzML or supported Bruker `.d` source once and stores a versioned `.mcraw` cache under the project `tmp` directory."
- [discussion] if the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated: "if the parent ion is not detected due to complete fragmentation in the ion source, its corresponding ISFs cannot be automatically annotated using this approach."

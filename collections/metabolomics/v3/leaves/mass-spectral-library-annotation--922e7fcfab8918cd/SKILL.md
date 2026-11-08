---
name: mass-spectral-library-annotation
description: Use when you have extracted MS/MS spectra (.msp) from an untargeted metabolomics
  study where library search leaves many spectra unannotated and you want fingerprint-based
  additional annotations.
when_to_use_negative:
- Spectra already have library hits — this workflow targets MS/MS spectra that were
  not annotated by mass spectral library search
- Input is raw chromatography data not yet deconvolved to MS/MS spectra; run an extraction
  step (e.g. IDSL.CSA) producing .msp first
- You lack a reference structure database to search predicted fingerprints against
  (fingerprint Tanimoto matching needs structures, not spectra)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0089
tools:
- name: IDSL.CSA
  role: Extracts unique MS/MS spectra from raw data and performs library matching
    against NIST 20 and LipidBlast with spectral entropy scoring
- name: IDSL_MINT
  role: Deep learning framework that translates unannotated MS/MS spectra into molecular
    fingerprint descriptors for structure search
  repo: https://github.com/idslme/IDSL_MINT
- name: RDKit
  role: Converts InChI/SMILES entries in MSP blocks to canonical SMILES and computes
    molecular fingerprints for Tanimoto similarity
  repo: https://www.rdkit.org/docs/Install.html
- name: PyTorch
  role: Backend framework in which IDSL_MINT's transformer models are implemented
    and trained
  repo: https://pytorch.org/get-started/locally
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00804-5
    title: 'IDSL_MINT: a deep learning framework to predict molecular fingerprints
      from mass spectra'
  source_content_hash: sha256:cbcf8ae6d31de263d95e5e11ce22ef1c62ba5608e7a2b4d101997f1f66dda0a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/idslme/IDSL_MINT
  tools:
  - IDSL.CSA
  - IDSL_MINT
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mass-spectral-library-annotation

## Summary

Annotate untargeted MS/MS features by library spectral matching (IDSL.CSA against NIST 20 and LipidBlast with spectral entropy >= 0.75), then boost annotation rate by searching IDSL_MINT-predicted molecular fingerprints against LipidMaps structures at Tanimoto >= 0.6.

## When to use

Use when you have extracted MS/MS spectra (.msp) from an untargeted metabolomics study where library search leaves many spectra unannotated and you want fingerprint-based additional annotations.

## When NOT to use

- Spectra already have library hits — this workflow targets MS/MS spectra that were not annotated by mass spectral library search
- Input is raw chromatography data not yet deconvolved to MS/MS spectra; run an extraction step (e.g. IDSL.CSA) producing .msp first
- You lack a reference structure database to search predicted fingerprints against (fingerprint Tanimoto matching needs structures, not spectra)

## Inputs

- Raw LC-MS/MS data processed to MS/MS spectra in NIST .msp format (MSP blocks must include PrecursorMZ entries)
- Mass spectral libraries (NIST 20, LipidBlast, MoNA, GNPS) for baseline spectral matching
- IDSL.CSA library-match results spreadsheet (e.g. csa_spreadsheet.zip from Zenodo)
- IDSL_MINT-predicted fingerprints for unannotated spectra (e.g. ms2fp_st002044_prediction.zip)
- Reference structure database with SMILES/InChI (e.g. LipidMaps .msp / lipidmaps_msp.zip)

## Outputs

- Baseline library-match annotation set (e.g. 536 lipids for ST002044 with spectral entropy >= 0.75)
- Additional fingerprint-based annotations at Tanimoto >= 0.6 (e.g. 238 lipids in both ESI modes)
- Merged per-feature annotation table with per-mode (ESI+/ESI−) counts (annotation_summary.csv)

## How to apply

First extract unique MS/MS spectra from raw data (e.g. with IDSL.CSA, as done for ST002044 yielding 3,386 ESI+ and 1,901 ESI− spectra) and match them against mass spectral libraries such as NIST 20 and LipidBlast, filtering matches with spectral entropy >= 0.75 to establish a baseline annotation set. Next, train mode-specific (ESI+/ESI−) IDSL_MINT transformer models on a curated MoNA/GNPS-derived training set, excluding compounds already covered by NIST 20 and LipidBlast, and predict molecular fingerprints for the spectra the library search failed to annotate. Search each predicted fingerprint against a structure database such as LipidMaps (fingerprints computed with RDKit from canonical SMILES/InChI), retaining top hits with Tanimoto similarity >= 0.6 as high-confidence annotations. Finally merge library-match and fingerprint-based annotation sets, resolving overlaps so each annotated MS/MS feature is counted once, and tabulate per-mode counts. The rationale is that >80% of MS/MS spectra lack any library hit, so fingerprint prediction recovers annotations for spectra invisible to library search.

## Related tools

- **IDSL.CSA** (Extracts unique MS/MS spectra from raw data and performs library matching against NIST 20 and LipidBlast with spectral entropy scoring)
- **IDSL_MINT** (Deep learning framework that translates unannotated MS/MS spectra into molecular fingerprint descriptors for structure search) — https://github.com/idslme/IDSL_MINT
- **RDKit** (Converts InChI/SMILES entries in MSP blocks to canonical SMILES and computes molecular fingerprints for Tanimoto similarity) — https://www.rdkit.org/docs/Install.html
- **PyTorch** (Backend framework in which IDSL_MINT's transformer models are implemented and trained) — https://pytorch.org/get-started/locally

## Examples

```
MINT_workflow --yaml /path/to/yaml/file
```

## Evaluation signals

- Library-match counts must reproduce the study baseline after filtering at spectral entropy >= 0.75 (e.g. 536 lipids for ST002044; article reports 638/3386 ESI+ and 299/1901 ESI− peaks annotated overall)
- Fingerprint-search annotations must retain only top hits with Tanimoto similarity >= 0.6 against the structure database
- Merged accounting must count each annotated MS/MS feature once, with per-mode totals matching the reported 536 + 238 split
- Predicted fingerprints and reference structures must derive from canonical SMILES computed consistently (RDKit) so Tanimoto values are comparable
- Spot-check top fingerprint hits: true positives are defined as top-hit candidates when the predicted fingerprint is searched against LipidMaps

## Limitations

- Case-study models were lipid-focused: training sets covered only 6.96% (ESI+) and 4.58% (ESI−) of LipidMaps compounds; other chemical classes may not be covered
- Library-match annotations depend on library coverage — a large proportion (>80%) of MS/MS spectra have no hit in existing libraries
- Fingerprint-based annotations are limited to the Tanimoto >= 0.6 high-confidence tier; lower-similarity hits are not counted
- MSP parsing requirements: blocks must include PrecursorMZ, and IDSL_MINT selects the longest content when duplicate headers occur

## Evidence

- [results] Tanimoto threshold: "IDSL_MINT new high-confidence annotations (Tanimoto similarity ≥ 0.6)"
- [results] baseline library coverage: "IDSL.CSA can annotate 638 (out of 3386) and 299 (out of 1901) MS/MS peaks in positive and negative modes by matching the spectra against NIST 20 and LipidBlast"
- [abstract] annotation-rate improvement: "IDSL_MINT was benchmarked using the LipidMaps database and improved the annotation rate of a test study for MS/MS spectra that were not originally annotated"
- [readme] MSP compatibility: "IDSL_MINT can extract information from `comment: ` and `comments: ` entries in ***.msp*** files which enables this platform to process MoNA, GNPS"
- [readme] MSP requirement: "MSP blocks must include `PrecursorMZ: ` row entries."

---
name: chromatogram-aird-compression
description: Use when a workflow needs to shrink large MRM raw datasets (e.g. PXD009543,
  PXD031038, Skyline test set) for web-based rendering or storage, or when benchmarking
  file-format compression ratios.
when_to_use_negative:
- The goal is quantitative MRM analysis or manual peak inspection rather than storage
  compression — use MRMPro/Skyline analysis instead
- Data must be shared in an interchange format accepted by third-party tools that
  do not read Aird (use mzML/mzXML, which AirdPro also exports but which are not the
  compression target)
- Files are already in Aird format — recompression offers no further volume benefit
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: AirdPro
  role: GUI client (C#, based on ProteoWizard pwiz_bindings_cli.dll) that converts
    vendor raw files to the Aird format with combinable compressors; supports MRM/SRM
    since V4.4.x/V5.1.0
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: MSConvert
  role: Converts vendor raw files to 32-bit mzML as the comparison baseline for compression
    ratios
- name: AirdSDK
  role: Java/C#/Python SDK for programmatically reading .aird/. files during downstream
    analysis or secondary development
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: MRMPro
  role: Web platform whose Aird-based compression meets the need for fast rendering
    of hundreds of chromatograms in the browser
  repo: github.com/CSi-Studio/MRMPro-Doc
- name: Skyline
  role: Source of the public MRM test dataset used as one of the three evaluation
    datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05685-x
    title: 'MRMPro: a web-based tool to improve the speed of manual calibration for
      multiple reaction monitoring data analysis by mass spectrometry'
  source_content_hash: sha256:f48c63c51251e24a5c5c55032dfd13fa4e94a538516451c1a221b71c0906fed3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - AirdSDK
  - MRMPro
  - MSConvert
  - Skyline
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# chromatogram-aird-compression

## Summary

Convert vendor MRM mass spectrometry raw files to the computation-oriented Aird format using AirdPro, benchmarking on-disk volume against 32-bit mzML (MSConvert) and vendor formats to quantify the >60% (vendor) and >80% (mzML) compression gains reported by the MRMPro paper.

## When to use

Use when a workflow needs to shrink large MRM raw datasets (e.g. PXD009543, PXD031038, Skyline test set) for web-based rendering or storage, or when benchmarking file-format compression ratios.

## When NOT to use

- The goal is quantitative MRM analysis or manual peak inspection rather than storage compression — use MRMPro/Skyline analysis instead
- Data must be shared in an interchange format accepted by third-party tools that do not read Aird (use mzML/mzXML, which AirdPro also exports but which are not the compression target)
- Files are already in Aird format — recompression offers no further volume benefit

## Inputs

- Vendor MRM raw files (e.g. SCIEX wiff/wiff2, Agilent .d folders) from public PRIDE datasets PXD009543 and PXD031038 and the Skyline test dataset
- AirdPro GUI client (V5/V6) for vendor-to-Aird conversion
- MSConvert (ProteoWizard) for vendor-to-32-bit-mzML conversion

## Outputs

- Per-dataset Aird files (.aird data file + . index file)
- storage_volume_comparison.csv with per-dataset volumes and Aird-vs-vendor and Aird-vs-mzML ratios

## How to apply

Download the vendor raw files (e.g. from PRIDE for PXD009543 and PXD031038) and convert them along two parallel paths: to 32-bit mzML with MSConvert, and to Aird with AirdPro (which builds on ProteoWizard pwiz_bindings_cli.dll and supports MRM acquisition since V4.4/V5.1). Record the total on-disk volume for each dataset in vendor, mzML, and Aird form; an Aird dataset consists of a .aird data file plus a . index file stored in the same directory with the same name. Compute the Aird ratio relative to vendor and relative to mzML per dataset, expecting roughly 40%/18% for PXD009543, 5%/20% for the Skyline test set, and 15%/14% for PXD031038. The rationale is that Aird's combinable compressors (ZDPD, ZDVB, ComboComp with Brotli/Snappy/Zstd/Zlib) yield both a smaller file and faster decoding, enabling fast web-side rendering of hundreds of chromatograms.

## Related tools

- **AirdPro** (GUI client (C#, based on ProteoWizard pwiz_bindings_cli.dll) that converts vendor raw files to the Aird format with combinable compressors; supports MRM/SRM since V4.4.x/V5.1.0) — https://github.com/CSi-Studio/AirdPro/releases/
- **MSConvert** (Converts vendor raw files to 32-bit mzML as the comparison baseline for compression ratios)
- **AirdSDK** (Java/C#/Python SDK for programmatically reading .aird/. files during downstream analysis or secondary development) — https://github.com/CSi-Studio/Aird-SDK
- **MRMPro** (Web platform whose Aird-based compression meets the need for fast rendering of hundreds of chromatograms in the browser) — github.com/CSi-Studio/MRMPro-Doc
- **Skyline** (Source of the public MRM test dataset used as one of the three evaluation datasets)

## Evaluation signals

- Measured Aird volume is ~40% of vendor and ~18% of mzML for PXD009543; ~5%/20% for the Skyline test set; ~15%/14% for PXD031038
- Overall compression meets the paper's claim of >60% reduction vs vendor and >80% vs mzML
- Each converted dataset produces a paired .aird and . file with the same filename in the same directory, scannable by AirdScanUtil
- The . AirdInfo index reports valid type=MRM, fileSize, and totalCount for the converted acquisition
- Recorded volumes for all three formats populate storage_volume_comparison.csv without missing per-dataset ratios

## Limitations

- Reported ratios are dataset-dependent (5%–40% of vendor volume across the three evaluation sets), so single-dataset results may not generalize
- AirdPro conversion has historically suffered memory overflow issues on large-scale conversions, only resolved in V6.0.0
- Aird is computation-oriented rather than a universal interchange format; reading it requires AirdSDK (Java, C#, or Python)

## Evidence

- [intro] An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side: "An efficient compression algorithm to meet the needs of fast rendering of hundreds of chromatograms on the web side"
- [abstract] which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format: "which reduced data volume by more than 60% and 80% respectively compared to vendor and mzML format"
- [readme] AirdPro is a GUI client for conversion from vendor files to Aird files: [evidence span withheld — anchor status: external_doc]
- [readme] Aird Index File and Aird Data File show be stored in the same directory: [evidence span withheld — anchor status: external_doc]
- [readme] Resolves memory overflow issues during large-scale conversions: [evidence span withheld — anchor status: external_doc]

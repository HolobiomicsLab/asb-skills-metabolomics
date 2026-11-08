---
name: compression-file-size-benchmark-tabulation
description: Decision support for choosing a mass-spectrometry storage format when
  you need to compare file sizes (and optionally decode cost) of a lossy-precision
  format like Aird against lossless reference formats (mzML, mgf, mz5) and the original
  vendor RAW across multiple public datasets.
when_to_use_negative:
- When the goal is to measure decoding/encoding time (3× faster ZDPD claim) or m/z-compression
  performance of ZDPD alone (49% size reduction vs Zlib) — that needs the compression
  benchmark skill, not this size-table one.
- When the input is already a converted (mzML/mgf/mz5/aird) file with no vendor RAW
  counterpart, since the table's baseline is the RAW size per dataset.
- When the raw data are ion mobility, ScanningSWATH, or MRM files, because the paper
  states those Aird modes were not yet supported at publication.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSConvert
  role: ProteoWizard converter used to produce the reference mzML, mgf, and mz5 formats
    whose sizes are tabulated against Aird.
- name: AirdPro
  role: GUI client that converts vendor RAW files into .aird + .json pairs, auto-detecting
    acquisition mode (DDA vs DIA/SWATH) and applying default m/z precision; the resulting
    file sizes feed the size_comparison_table.csv rows.
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Developer tool used to read the .aird files; relevant only for verifying the
    generated Aird outputs, not for the size table itself.
  repo: https://github.com/CSi-Studio/Aird-SDK
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:f79657532c736acd53517e297315186a01fc30815d8ba7e84e324518c0f3bd7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MSConvert
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# compression-file-size-benchmark-tabulation

## Summary

Converts a set of vendor mass-spectrometry files and reference formats (mzML, mgf, mz5, Aird) into per-file byte sizes and tabulates relative compression ratios, so that claims like 'Aird averages 2.0 GB vs RAW in DDA (54%)' or 'Aird equals 77% of RAW in DIA/SWATH' can be checked against actual data.

## When to use

Decision support for choosing a mass-spectrometry storage format when you need to compare file sizes (and optionally decode cost) of a lossy-precision format like Aird against lossless reference formats (mzML, mgf, mz5) and the original vendor RAW across multiple public datasets.

## When NOT to use

- When the goal is to measure decoding/encoding time (3× faster ZDPD claim) or m/z-compression performance of ZDPD alone (49% size reduction vs Zlib) — that needs the compression benchmark skill, not this size-table one.
- When the input is already a converted (mzML/mgf/mz5/aird) file with no vendor RAW counterpart, since the table's baseline is the RAW size per dataset.
- When the raw data are ion mobility, ScanningSWATH, or MRM files, because the paper states those Aird modes were not yet supported at publication.

## Inputs

- Vendor RAW mass-spectrometry files (e.g. Thermo, SCIEX TripleTOF/Q-Exactive, Agilent .d) downloaded from ProteomeXchange/MetaboLights accessions PXD021390, PXD018139, IPX0001509000, MTBLS2119, XCMS1197236, PXD002952, PXD004712
- mzML, mgf, and mz5 files produced by MSConvert for the same RAW inputs
- .aird + .json pairs produced by AirdPro with default m/z precision (mzPrecision = 0.0001)

## Outputs

- size_comparison_table.csv with exactly 7 dataset rows and one column per format (RAW, mzML, mgf, mz5, .aird)
- Per-dataset byte-size ratio (Aird vs RAW/mzML for DDA; Aird vs RAW for DIA) suitable for comparison against the published averages of 2.0 GB (54%) in DDA and 24.6 GB (77%) in DIA/SWATH

## How to apply

Take each of the seven public accessions listed in the Aird paper (PXD021390, PXD018139, IPX0001509000, MTBLS2119, XCMS1197236, PXD002952, PXD004712) and record the byte size of the original vendor RAW file. Convert every RAW into mzML, mgf, and mz5 with MSConvert (ProteoWizard), and into Aird (.aird + .json pair in the same directory with the same base name) using AirdPro, letting it auto-detect DDA vs DIA/SWATH and keeping the default m/z precision mzPrecision = 0.0001. Write one row per dataset with one column per format (RAW, mzML, mgf, mz5, .aird) to size_comparison_table.csv, exactly 7 rows, then compute the relative ratio of Aird to the other formats per acquisition mode — average Aird/RAW ≈ 54% (2.0 GB) for DDA and 77% (24.6 GB) for the DIA/SWATH dataset. Rationale: Aird's claimed advantage is high compression rate via Zlib-Diff-PforDelta on the m/z array plus skipping zero-intensity points, so the size ratio is the primary correctness signal for a conversion-side skill.

## Related tools

- **MSConvert** (ProteoWizard converter used to produce the reference mzML, mgf, and mz5 formats whose sizes are tabulated against Aird.)
- **AirdPro** (GUI client that converts vendor RAW files into .aird + .json pairs, auto-detecting acquisition mode (DDA vs DIA/SWATH) and applying default m/z precision; the resulting file sizes feed the size_comparison_table.csv rows.) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Developer tool used to read the .aird files; relevant only for verifying the generated Aird outputs, not for the size table itself.) — https://github.com/CSi-Studio/Aird-SDK

## Examples

```
AirdPro.exe --input PXD021390_RAW --output out/ --format aird  # then record RAW, mzML, mgf, mz5, .aird sizes into size_comparison_table.csv (7 rows, one column per format)
```

## Evaluation signals

- size_comparison_table.csv contains exactly 7 dataset rows (one per accession) and one column per format.
- For each dataset, .aird and .json pairs share the same base name in the same directory, matching the Aird storage convention.
- Computed average Aird/RAW ratio across the DDA datasets lands near 54% (≈2.0 GB average), and the DIA/SWATH dataset's Aird file is ≈77% of its RAW size (≈24.6 GB).
- mzML, mgf, and mz5 sizes were produced by MSConvert for the same RAW inputs, so no missing or unmatched conversion exists in the table.
- The Aird conversion used the default m/z precision (mzPrecision = 0.0001) as specified, so size differences cannot be attributed to a non-default precision choice.

## Limitations

- The seven accessions cover SCIEX TripleTOF 5600/6600, Thermo Q-Exactive and Q-Exactive Plus, and Agilent instruments — results may not generalize to other vendors or acquisition modes.
- The paper's discussion of ion mobility and ScanningSWATH notes these formats were still under development, so size comparisons for those modes are out of scope.
- Aird conversion is lossy at the m/z level (controllable precision via integer delta coding), so a smaller size does not imply bit-for-bit equivalence with the vendor or mzML representation.
- No benchmark scripts are provided to reproduce the 49% ZDPD vs Zlib or 3× decoding speed claims in the paper; the same is true here — the size table is a manual or re-implemented pipeline, not a shipped artifact.

## Evidence

- [abstract] Seven datasets from SCIEX, Thermo, and Agilent instruments were compared against mzML, mgf, and mz5 formats: "We have downloaded seven datasets from ProteomeXchange and Metabolights. They are from different SCIEX, Thermo, and Agilent instruments."
- [results] Aird file size relative to RAW in DDA mode: [evidence span withheld — anchor status: not_found]
- [results] Aird file size relative to RAW in DIA/SWATH mode: [evidence span withheld — anchor status: not_found]
- [abstract] Aird uses JSON for metadata storage: "Aird uses JavaScript Object Notation for metadata storage."
- [abstract] Conversion pipeline via MSConvert: "Then we convert the raw data into mzML, mgf, and mz5 file formats by MSConvert"
- [methods] AirdPro reads vendor files via ProteoWizard SDK: "AirdPro uses the integrated SDK provided by ProteoWizard [9] to read vendor file data for the first step of conversion"
- [methods] Ignore zero-intensity points to reduce size: "Ignore Zero Intensity Point. Aird would not store the zero-intensity pairs as most computation scenarios will not use these data."
- [methods] Aird indexing strategy description: "According to the MS file's acquisition mode, AirdPro will reorder the spectrums for better random access performance"
- [intro] Aird precision control mechanism: "After determining the required precision, Aird converts the floating-point m/z array to an integer array."
- [discussion] AirdPro release and open source availability: "AirdPro is written in C# and is an opensource project for download at https://github.com/CSi-Studio/AirdPro/releases/."
- [readme] Aird file naming convention: [evidence span withheld — anchor status: not_found]
- [readme] Supported acquisition methods in AirdPro: [evidence span withheld — anchor status: not_found]

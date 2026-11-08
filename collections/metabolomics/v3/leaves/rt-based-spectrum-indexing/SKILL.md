---
name: rt-based-spectrum-indexing
description: Use when converting vendor MS data to the Aird format and you need fast
  random access to spectra by retention time, i.e. when building the .json metadata
  alongside the .aird data file.
when_to_use_negative:
- The target acquisition mode is ScanningSWATH or Ion Mobility — per the paper these
  formats were still under development and lack index support.
- The output format is mzML, mgf, or mz5 — this index is specific to the Aird format's
  JSON metadata layer, not these formats.
- You only need generic text-file conversion with no random-access readback; without
  an Aird reader the extra index building is wasted effort.
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0091
tools:
- name: AirdPro
  role: C# GUI conversion client that reads vendor files with the ProteoWizard SDK,
    reorders spectra by acquisition mode, and builds the correlation indexes (RT-MS,
    MS1-MS2 group, precursor m/z range-MS2 group) in the JSON metadata
  repo: https://github.com/CSi-Studio/AirdPro/releases/
- name: Aird-SDK
  role: Java/C#/Python SDK that consumes the generated .json index to read spectrum
    data from the .aird file quickly during analysis
  repo: https://github.com/CSi-Studio/Aird-SDK
- name: MSConvert
  role: ProteoWizard tool used to produce mzML, mgf, and mz5 comparison formats against
    which the Aird indexed format was benchmarked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-021-04490-0
    title: 'Aird: a computation-oriented mass spectrometry data format enables a higher
      compression ratio and less decoding time'
  source_content_hash: sha256:9ef7ab9336e6f1743b9282c44157aab5f7e78a6ffe17eaef51a00a7432f63cef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Aird-SDK
  - AirdPro
  - MSConvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# RT-based spectrum indexing

## Summary

Builds the RT-MS index in Aird's JSON metadata layer so MS data are arranged linearly by retention time and each spectrum's RT and start position are recorded, letting programs read the .aird data file quickly during analysis.

## When to use

Use when converting vendor MS data to the Aird format and you need fast random access to spectra by retention time, i.e. when building the .json metadata alongside the .aird data file.

## When NOT to use

- The target acquisition mode is ScanningSWATH or Ion Mobility — per the paper these formats were still under development and lack index support.
- The output format is mzML, mgf, or mz5 — this index is specific to the Aird format's JSON metadata layer, not these formats.
- You only need generic text-file conversion with no random-access readback; without an Aird reader the extra index building is wasted effort.

## Inputs

- Compressed spectrum blocks (.aird data file) produced from vendor files (e.g. SCIEX wiff, Thermo, Agilent .d) via AirdPro/ProteoWizard
- Acquisition mode classification of spectra (DDA, DIA/SWATH, PRM)
- Per-spectrum retention times and byte start positions

## Outputs

- RT-MS index entries (retention time + start position per spectrum) serialized in the AirdMetaData JSON index file
- A .json index file stored alongside the .aird file with the same base name

## How to apply

After AirdPro reads vendor data via the ProteoWizard SDK and compresses the m/z and intensity arrays, arrange the MS data linearly by retention time. For each spectrum, store its retention time and the start position of its data block as an RT-MS index entry in the AirdMetaData JSON structure. This index is built alongside acquisition-mode-specific indexes: the MS1-MS2 group index for DDA (contiguous MS1 blocks with adjacent MS2 groups) and the precursor m/z range-MS2 group index for DIA/SWATH (MS2 grouped per SWATH window). The rationale is that correlation indexes in the metadata let programs read data more quickly during analysis than scanning the whole file. The resulting JSON index file must be written in the same directory as the .aird file with an identical base name so utilities such as AirdScanUtil can locate the pair.

## Related tools

- **AirdPro** (C# GUI conversion client that reads vendor files with the ProteoWizard SDK, reorders spectra by acquisition mode, and builds the correlation indexes (RT-MS, MS1-MS2 group, precursor m/z range-MS2 group) in the JSON metadata) — https://github.com/CSi-Studio/AirdPro/releases/
- **Aird-SDK** (Java/C#/Python SDK that consumes the generated .json index to read spectrum data from the .aird file quickly during analysis) — https://github.com/CSi-Studio/Aird-SDK
- **MSConvert** (ProteoWizard tool used to produce mzML, mgf, and mz5 comparison formats against which the Aird indexed format was benchmarked)

## Examples

```
pip install AirdSDK
```

## Evaluation signals

- The .json index file and .aird data file are stored in the same directory with the same base name, so AirdScanUtil.class can scan both files together.
- The JSON parses against the published Aird Schema (AirdMetaData.json in the Aird-SDK repository).
- Each RT-MS index entry records both the spectrum's retention time and its start position in the .aird file.
- Spectrum ordering matches the acquisition mode: MS1 spectra are grouped into contiguous MS1 blocks with corresponding MS2 groups adjacent (DDA), or MS2 data are grouped per SWATH precursor m/z window (DIA).
- Downstream readers can retrieve spectra by RT without scanning the whole file — the paper's benchmark criteria include random access performance for indexed reading.

## Limitations

- Aird's supporting acquisition formats at the time of the paper are DDA, DIA/SWATH, and PRM; ScanningSWATH and Ion Mobility are under development, so the index strategy does not cover them.
- The index is only usable with an Aird-aware reader (Aird-SDK); the paper notes that without specific libraries these files are not readable.
- The pairing convention (same directory, same base name for .json and .aird) is a hard requirement; violating it breaks file discovery.

## Evidence

- [methods] RT-MS index definition (methods): "RT-MS index: This is one of the most traditional indexing strategies"
- [methods] Spectrum reordering for random access (methods): "According to the MS file's acquisition mode, AirdPro will reorder the spectrums for better random access performance"
- [methods] Purpose of the correlation indexes (methods): "Aird builds additional correlation indexes in the metadata so that the program can read the data more quickly during analysis"
- [readme] JSON index/data file pairing convention (README): "Aird Index File and Aird Data File show be stored in the same directory with the same file"
- [abstract] JSON-based metadata storage (abstract): "Aird uses JavaScript Object Notation for metadata storage."

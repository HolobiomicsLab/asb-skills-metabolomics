---
name: spectral-library-segregation-ionization-chromatography
description: Use when a merged OMSL (e.g. GNPS .MGF, MoNA, MassBank, MS-Dial exports)
  must be split into positive/negative, GC/LC, and experimental/in silico partitions
  before annotation with MS-Dial, MZmine, or Flash Entropy Search.
when_to_use_negative:
- The input is a single homogeneous library already presplit by ionization mode (e.g.
  MS-Dial-VS17 PI/negative exports) requiring no further segregation
- The goal is general metadata cleaning or SPLASH-key deduplication rather than producing
  mode/technique-specific library partitions
- Spectra lack ionization-mode, chromatographic, and origin annotations and no filename
  suffix can supply them — classification would be unreliable
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: FragHub
  role: Executes the classification and segregation of spectra into output partitions
    by ionization mode, chromatography, and origin
  repo: github.com/eMetaboHUB/FragHub
- name: spectra-hash (SPLASH)
  role: Provides per-spectrum hash keys used to track processed spectra between update
    runs while partitions are rebuilt
  repo: github.com/berlinguyinca/spectra-hash
- name: MS-Dial
  role: Downstream consumer of the segregated, standardized libraries
- name: MZmine
  role: Downstream consumer of the segregated, standardized libraries
- name: Flash Entropy Search
  role: Downstream search engine that consumes the partitioned libraries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:4db0f06a9ae47f33a54bc17dd4553bc4d28835eeba4fe80ef53749c8b1e53ab7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - Flash Entropy Search
  - FragHub
  - MS-Dial
  - MZmine
  - Python
  - spectra-hash (SPLASH)
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-library-segregation-ionization-chromatography

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions an integrated mass spectral library into separate output files by ionization mode (positive/negative), chromatographic technique (GC/LC-MS), and data origin (experimental vs predicted/in silico), so downstream search tools receive homogeneous, correctly tagged spectra.

## When to use

Use when a merged OMSL (e.g. GNPS .MGF, MoNA, MassBank, MS-Dial exports) must be split into positive/negative, GC/LC, and experimental/in silico partitions before annotation with MS-Dial, MZmine, or Flash Entropy Search.

## When NOT to use

- The input is a single homogeneous library already presplit by ionization mode (e.g. MS-Dial-VS17 PI/negative exports) requiring no further segregation
- The goal is general metadata cleaning or SPLASH-key deduplication rather than producing mode/technique-specific library partitions
- Spectra lack ionization-mode, chromatographic, and origin annotations and no filename suffix can supply them — classification would be unreliable

## Inputs

- Harmonized spectrum records with ionization mode, chromatographic method, and predicted/experimental origin fields
- Input library files (.MSP, .MGF, .JSON, .CSV, .XML) from GNPS, MoNA, MassBank, MS-Dial-VS17, or in-house MZmine3 output
- Filenames carrying '_insilico' or '_GC' suffixes when spectra lack explicit origin or GC annotations

## Outputs

- Partitioned library files (.MSP, .MGF, .JSON, .CSV) split by positive/negative ionization mode, GC/LC separation, and experimental-vs-predicted origin
- Deduplicated spectra per partition (identical InChIKeys and fragment lists removed)

## How to apply

After parsing and metadata harmonization, classify each spectrum by ionization mode (positive/negative), separation technique (GC/LC-MS), and origin (experimental/predicted), marking the 'predicted' field explicitly. If a source file contains only in-silico spectra or only GC spectra and this is not stated in the spectrum fields, suffix the filename with '_insilico' or '_GC' (e.g. 'GMD_20111121_MDN35_ALK_MSP_GC.msp') so the classifier can assign them. Normalize instrument and ionization descriptors (QTOF, FT, ESI, APCI) using a HUPO PSI-MS hierarchical decision tree before segregating. Within each partition, remove duplicates with identical InChIKeys and fragment lists, then write each subset to the chosen output formats (.MSP, .MGF, .JSON, .CSV). The rationale is that MS-Dial, MZmine, and Flash Entropy Search consume mode- and technique-specific libraries, so segregated outputs enhance annotation utility.

## Related tools

- **FragHub** (Executes the classification and segregation of spectra into output partitions by ionization mode, chromatography, and origin) — github.com/eMetaboHUB/FragHub
- **spectra-hash (SPLASH)** (Provides per-spectrum hash keys used to track processed spectra between update runs while partitions are rebuilt) — github.com/berlinguyinca/spectra-hash
- **MS-Dial** (Downstream consumer of the segregated, standardized libraries)
- **MZmine** (Downstream consumer of the segregated, standardized libraries)
- **Flash Entropy Search** (Downstream search engine that consumes the partitioned libraries)

## Examples

```
FragHub executable with input "GMD_20111121_MDN35_ALK_MSP_GC.msp" (filename-suffixed GC library), selecting output partitions and formats (.MSP, .MGF, .JSON, .CSV) via the GUI tabs, then START
```

## Evaluation signals

- Every output spectrum carries a valid ionization-mode, GC/LC, and predicted/experimental assignment; 'UNKNOWN' placeholders are used only where source metadata was absent
- Within each partition, no two spectra share identical InChIKeys and fragment lists (duplicate removal verified)
- MS-Dial, MZmine, and Flash Entropy Search can import the partitioned .MSP/.MGF/.JSON/.CSV outputs without errors
- Spectra lacking SMILES, InChI, precursor m/z, and adduct information are absent from all partitions (inconsistent spectra removed)
- Filename-based tags ('_insilico', '_GC') correctly propagate to the origin and chromatographic classification of the affected spectra

## Limitations

- The 20_12_2024 FragHub release removed MacOS and Linux support, restricting how the executable-based segregation workflow can be run
- Collision energy harmonization was not addressed (~70 different nonstandardized formats), so partitions still mix collision-energy conventions
- License changed from MIT to CC-BY-NC 4.0, imposing non-commercial restrictions on redistributed segregated libraries

## Evidence

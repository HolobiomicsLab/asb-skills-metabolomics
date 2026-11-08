---
name: spotwise-acquisition-peak-to-image-conversion
description: Decision support for converting spot-wise or line-by-line LA-ICP-MS vendor
  exports (Agilent MassHunter .b batches, Perkin Elmer Elan 'XL', Thermo Qtegra) into
  images without manual preprocessing.
when_to_use_negative:
- Input is already a precalibrated or preprocessed image array (e.g. a NumPy export)
  — direct loading suffices.
- The acquisition is imaging-mode data in imzML format — use the dedicated imzML import
  wizard instead of vendor batch parsing.
- Element peak positions differ substantially across isotopes, making marker-guided
  integration unreliable for spot-wise data.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Pew2
  role: GUI application that performs drag-and-drop import of spot-wise vendor batches
    and assembles isotope signal arrays into images
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library implementing the vendor-format import and export logic that
    underpins the conversion
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Backing array library used to hold the 2D isotope image arrays
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:6dea5cfd7f68ba530253f62e82a599703c4a54475ee8c6e8221749e961e44352
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Perkin Elmer ICP-MS
  - Pew2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# spotwise-acquisition-peak-to-image-conversion

## Summary

Convert raw spot-wise LA-ICP-MS acquisitions from Agilent, Perkin Elmer, or Thermo vendor exports directly into image arrays by using a single marker element to guide peak integration across isotopes, eliminating manual preprocessing.

## When to use

Decision support for converting spot-wise or line-by-line LA-ICP-MS vendor exports (Agilent MassHunter .b batches, Perkin Elmer Elan 'XL', Thermo Qtegra) into images without manual preprocessing.

## When NOT to use

- Input is already a precalibrated or preprocessed image array (e.g. a NumPy export) — direct loading suffices.
- The acquisition is imaging-mode data in imzML format — use the dedicated imzML import wizard instead of vendor batch parsing.
- Element peak positions differ substantially across isotopes, making marker-guided integration unreliable for spot-wise data.

## Inputs

- Raw Agilent MassHunter batch directory (.b, G7200B or later) with one sample per line
- Perkin Elmer ELAN .xl directory
- Thermo Qtegra or iCap CSV/LDR export
- Vendor instrument metadata (isotope names, MS acquisition time, laser parameters)

## Outputs

- 2D isotope image arrays in Pew2's internal NumPy archive format
- Isotope name list extracted from acquisition metadata
- Image configuration with applied MS acquisition time, laser spot size, and scan speed

## How to apply

Drag-and-drop the raw vendor export into Pew2 — for spot-wise data, Pew2 detects the vendor format (Agilent batch directory, Perkin Elmer 'XL', Thermo Qtegra export) and parses per-line signal arrays for each isotope. Because spot-wise acquisitions require peak positions to be located within each line, one element is designated a 'marker' that is present across the entire sample; its peaks guide the integration windows of all other elements, which must have similar peak positions. During import, any available instrument parameters such as MS acquisition time are read from the vendor metadata and applied to the image configuration. The per-isotope signal arrays are then assembled into 2D image arrays using the laser pixel aspect defined by acquisition time and laser spot size/speed (spot size and speed are entered manually after import if needed to set image aspect and scale bar).

## Related tools

- **Pew2** (GUI application that performs drag-and-drop import of spot-wise vendor batches and assembles isotope signal arrays into images) — https://github.com/djdt/pewpew
- **pewlib** (Python library implementing the vendor-format import and export logic that underpins the conversion) — https://github.com/djdt/pewlib
- **NumPy** (Backing array library used to hold the 2D isotope image arrays)

## Examples

```
pewpew  # drag-and-drop Agilent .b batch into the GUI; or pip install pewlib and use its loader in a script
```

## Evaluation signals

- Image preserves the per-line pixel ordering from the raw vendor export.
- Instrument parameters such as MS acquisition time from the vendor file are reflected in the image configuration.
- Scale bar and image aspect are correct after manual entry of laser spot size and speed.
- At least one marker element is present across the entire sample and its peak positions align with those of all other elements.

## Limitations

- Spot-wise import requires at least one element to be present across the entirety of the sample and for peak positions of each element to be similar.
- Laser spot size and speed may need to be entered manually after import to guarantee correct image aspect and scale bar sizing.
- Support for vendors beyond Agilent, Perkin Elmer, and Thermo relies on generic text image or file-per-line importers rather than native parsing.

## Evidence

- [results] Drag-and-drop importation of raw Agilent batches (Mass-Hunter G7200B or later), Perkin Elmer Elan "XL", and Thermo Fischer Qtegra exports, as well as both text image and file-per-line exports: [evidence span withheld — anchor status: not_found]
- [readme] Pewlib is a library for importing, processing and exporting LA-ICP-MS data. Currently exports from Agilent, Thermo and PerkinElmer software is supported: [evidence span withheld — anchor status: external_doc]

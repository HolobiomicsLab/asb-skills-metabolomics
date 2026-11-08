---
name: machine-readable-peak-list-export
description: Use when a spectrum has already been located via a USI (GNPS task, MassBank
  accession, MetaboLights/Workbench dataset, ProteomeXchange scan) and the underlying
  peak list must be retrieved as structured data for downstream scripts or reproducible
  reporting.
when_to_use_negative:
- The spectrum must be created from raw vendor data or a non-supported repository
  — the resolver only handles the USI types listed (GNPS, MassBank, MS2LDA, MassIVE,
  MetaboLights, Metabolomics Workbench, ProteomeXchange).
- The target is a library-spectrum comparison requiring spectral matching, peak alignment,
  or similarity scoring — this skill only exports and visualizes a single peak list,
  it does not compare spectra.
- The identifier is a finalized mzspec USI whose source repository is outside the
  resolver's supported set; the service returns no usable data for unsupported sources.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Metabolomics Spectrum Resolver
  role: Web service that resolves a USI to its peak list and serves it as JSON and
    CSV; also renders SVG/PNG/QR outputs from the same USI.
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
- name: spectrum_utils
  role: Python plotting library underneath the resolver's rendering, ensuring the
    exported peak data matches the drawn figure.
  repo: https://github.com/mwang87/MetabolomicsSpectrumResolver
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2020.05.09.086066
    title: Universal MS/MS Visualization and Retrieval with the Metabolomics Spectrum
      Resolver Web Service
  source_content_hash: sha256:7fbb3e345f74b59857b1469377351eac10e369c4fe186b33eb0bdd90cd14e5f0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  repo_url: https://github.com/mwang87/MetabolomicsSpectrumResolver
  tools:
  - Metabolomics Spectrum Resolver
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# machine-readable-peak-list-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Export the peak list behind a USI-referenced MS/MS spectrum in machine- and human-readable formats (JSON, CSV) alongside rendered figures, using the Metabolomics Spectrum Resolver so spectra can be reused, piped into scripts, or embedded in publications.

## When to use

Use when a spectrum has already been located via a USI (GNPS task, MassBank accession, MetaboLights/Workbench dataset, ProteomeXchange scan) and the underlying peak list must be retrieved as structured data for downstream scripts or reproducible reporting.

## When NOT to use

- The spectrum must be created from raw vendor data or a non-supported repository — the resolver only handles the USI types listed (GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange).
- The target is a library-spectrum comparison requiring spectral matching, peak alignment, or similarity scoring — this skill only exports and visualizes a single peak list, it does not compare spectra.
- The identifier is a finalized mzspec USI whose source repository is outside the resolver's supported set; the service returns no usable data for unsupported sources.

## Inputs

- USI string for a supported source (GNPS task/library, MassBank accession, MS2LDA motif, MassIVE, MetaboLights, Metabolomics Workbench, ProteomeXchange scan)
- Peak list data (e.g. MGF peak table or library spectrum) used to build the USI
- Optional rendering parameters (mz_min, mz_max, max_intensity, width, height, annotate_peaks, annotate_threshold, annotation_rotation) when figures are exported alongside the data

## Outputs

- JSON file of spectrum peaks (machine-readable) from /json/
- CSV file of comma-separated peaks (human-readable) from /csv/
- Optional SVG/PNG figure and QR code rendered from the same USI
- Optional PROXI v0.1 /spectra JSON response

## How to apply

Construct a draft-metabolomics USI (prefix 'mzdraft' until the spec stabilizes; the service also serves 'mzspec') and submit it to the Spectrum Resolver's /json/ endpoint to obtain the peak list as JSON (machine readable) and to the /csv/ endpoint for a comma-separated peak table (human readable). The same USI can simultaneously drive /svg/ or /png/ figure rendering and /qrcode/ generation, so the data export and the visualization come from one identifier and stay consistent. Validate the payload against the peak data used to build the USI (e.g. the m/z and intensity pairs from the source MGF or library entry) before treating the export as canonical. Because the tool sits on top of spectrum_utils, the exported peaks are the same values the rendered figure draws, which is what makes publication figure and data file reproducible from the same call.

## Related tools

- **Metabolomics Spectrum Resolver** (Web service that resolves a USI to its peak list and serves it as JSON and CSV; also renders SVG/PNG/QR outputs from the same USI.) — https://github.com/mwang87/MetabolomicsSpectrumResolver
- **spectrum_utils** (Python plotting library underneath the resolver's rendering, ensuring the exported peak data matches the drawn figure.) — https://github.com/mwang87/MetabolomicsSpectrumResolver

## Examples

```
curl "https://metabolomics-usi.ucsd.edu/json/?usi=mzdraft:GNPS:TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra/specs_ms.mgf:scan:1943" > peaks.json && curl "https://metabolomics-usi.ucsd.edu/csv/?usi=mzdraft:GNPS:TASK-c95481f0c53d42e78a61bf899e9f9adb-spectra/specs_ms.mgf:scan:1943" > peaks.csv
```

## Evaluation signals

- The /json/ response contains a list of {m/z, intensity} pairs whose m/z range matches the mz_min/mz_max bounds (or the full spectrum when no bounds are set).
- The /csv/ output has the same peak count and values as the /json/ response — the two exports must be mutually consistent.
- Rendering the same USI with /svg/ or /png/ produces peaks that visually align with the exported peak list (the exported data is the ground truth for the figure).
- A QR code generated from the same USI encodes the identical identifier, so any consumer scanning it lands on the same peak data.
- The USI prefix ('mzdraft' vs 'mzspec') is documented in the output so downstream users know the identifier's stability status.

## Limitations

- The metabolomics USI format is still a draft: identifiers use 'mzdraft' instead of 'mzspec' and are subject to change, so exported data may break if the spec is revised.
- USI support is limited to the listed sources; spectral data outside GNPS, MassBank, MS2LDA, MassIVE, MetaboLights, Metabolomics Workbench, or ProteomeXchange cannot be resolved.
- The service is designed for retrieval and rendering of already-processed peak lists — it does not perform peak detection, deconvolution, or spectral matching.
- ProteomeXchange USI support currently covers mainly proteomics data; metabolomics repository data is handled via the extended draft USI formats instead.

## Evidence

- [abstract] allow for consistent data export (in human and machine-readable forms) and publication-ready visualisations of tandem mass spectrometry spectra: "allow for consistent data export (in human and machine-readable forms) and publication-ready visualisations of tandem mass spectrometry spectra"

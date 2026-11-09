---
name: usi-spectra-retrieval-from-public-repositories
description: Use when you have USIs (e.g. `mzspec:GNPS2:TASK-<id>-nf_output/clustering/specs_ms.mgf:scan:<n>`)
  and need to fetch the corresponding MS/MS spectra to feed a molecular-networking
  or spectral-alignment pipeline.
when_to_use_negative:
- You only have raw LC-MS/MS vendor files (e.g. Thermo .RAW, mzML) that are not indexed
  in a USI-resolvable public repository — ingest them through molecular-networking
  tasks first.
- You need de novo library search or spectral matching only, without multiple alignment
  across a molecular family.
- The source dataset is not exposed through a USI-supporting public source (MassIVE/GNPS,
  Metabolomics Workbench, PRIDE, GNPS2).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3520
tools:
- name: MMSA
  role: Consumes USI-resolved spectra as input to its multiple spectrum alignment
    pipeline (setscreation -> spectraalignment).
  repo: github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: Metabolomics Spectrum Resolver
  role: Public USI-resolution service used to fetch spectra from MassIVE/GNPS, Metabolomics
    Workbench, PRIDE and GNPS2.
- name: GNPS2
  role: Source repository of molecular-networking tasks whose output MGF spectra are
    addressed via `mzspec:GNPS2:TASK-<id>-nf_output/clustering/specs_ms.mgf:scan:<n>`.
- name: NetworkFamily_MultipleAlignment_Website (Flask/Dash app)
  role: Web interface that accepts USI input, resolves spectra, and triggers 'Process
    and Save JSON'.
  repo: github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:21145f67037d5c41a9edea2d672388bd3befcb1cd0a335e0ea2396e016b30fdf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Flask
  - GNPS2
  - MMSA
  - Metabolomics Spectrum Resolver
  - NetworkFamily_MultipleAlignment_Website (Flask/Dash app)
  - NetworkX
  - Plotly
  - Python Dash
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# usi-spectra-retrieval-from-public-repositories

## Summary

Retrieves tandem mass spectra from public MS/MS data sources (MassIVE/GNPS, Metabolomics Workbench, PRIDE, GNPS2) via Universal Spectrum Identifiers, feeding spectral-alignment and molecular-networking workflows such as MMSA.

## When to use

Use when you have USIs (e.g. `mzspec:GNPS2:TASK-<id>-nf_output/clustering/specs_ms.mgf:scan:<n>`) and need to fetch the corresponding MS/MS spectra to feed a molecular-networking or spectral-alignment pipeline.

## When NOT to use

- You only have raw LC-MS/MS vendor files (e.g. Thermo .RAW, mzML) that are not indexed in a USI-resolvable public repository — ingest them through molecular-networking tasks first.
- You need de novo library search or spectral matching only, without multiple alignment across a molecular family.
- The source dataset is not exposed through a USI-supporting public source (MassIVE/GNPS, Metabolomics Workbench, PRIDE, GNPS2).

## Inputs

- One-per-line Universal Spectrum Identifiers of the form `mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>`
- Alternative: a GNPS2 Task ID plus molecular-networking component number
- Alternative: FBMN Task ID and component numbers

## Outputs

- Resolved MS/MS spectra (peak lists) ready for preprocessing and multiple spectral alignment
- JSON file of processed sets/spectra written to `./temp/sets` (`SETS_TEMP_PATH`)

## How to apply

Format each spectrum reference as a Universal Spectrum Identifier, one per line, following the pattern `mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>`. Submit these USIs through the web application's `/setscreation/` input interface (which accepts USIs, GNPS Task ID plus component number, or FBMN task identifiers). The backend resolves the USIs through the metabolomics resolver, which pulls the spectra from any public data source that supports USIs — including MassIVE/GNPS, Metabolomics Workbench, PRIDE, and GNPS2. Click 'Process and Save JSON' to resolve the spectra and persist the alignment-ready JSON to `SETS_TEMP_PATH` (`./temp/sets`). Downstream, the resolved spectra are preprocessed (top-6 peak filtering within ±25 m/z, exclusion of peaks within ±17 m/z of the precursor, square-root intensity transformation, normalization) before MST-based multiple alignment in MMSA.

## Related tools

- **MMSA** (Consumes USI-resolved spectra as input to its multiple spectrum alignment pipeline (setscreation -> spectraalignment).) — github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **Metabolomics Spectrum Resolver** (Public USI-resolution service used to fetch spectra from MassIVE/GNPS, Metabolomics Workbench, PRIDE and GNPS2.)
- **GNPS2** (Source repository of molecular-networking tasks whose output MGF spectra are addressed via `mzspec:GNPS2:TASK-<id>-nf_output/clustering/specs_ms.mgf:scan:<n>`.)
- **NetworkFamily_MultipleAlignment_Website (Flask/Dash app)** (Web interface that accepts USI input, resolves spectra, and triggers 'Process and Save JSON'.) — github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website

## Examples

```
python main.py  # then in the web UI paste USIs into /setscreation/, e.g. mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>, and click 'Process and Save JSON'
```

## Evaluation signals

- Each USI returns a non-empty peak list from the metabolomics resolver — no resolver errors or empty spectra in the fetched set.
- Number of retrieved spectra matches the number of USIs supplied (or the number of scans in the GNPS2 component).
- Precursor m/z values of the retrieved spectra span the expected molecular family range when sorted, matching the GNPS2 component annotation.
- The 'Process and Save JSON' step writes a valid JSON file to `./temp/sets` that the downstream Plotly/Dash alignment visualizer can load without errors.
- Preprocessing invariants hold after retrieval: top-6 peaks per ±25 m/z window, no peaks within ±17 m/z of the precursor, intensities square-root transformed and normalized.

## Limitations

- Small peak alignments (e.g. y2 peaks in the albicidin family) are not fully captured by the downstream greedy multiple alignment, so retrieval alone does not guarantee complete consensus peak sets.
- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3) or rhamnose bound to C12 (m/z 385.219, [M+Na]+ of C18H34O7) in the rhamnolipid family — retrieval of those spectra does not guarantee their alignment.
- Only USI-supporting public sources (MassIVE/GNPS, Metabolomics Workbench, PRIDE, GNPS2) can be queried; unindexed local or proprietary data cannot be resolved through this path.
- The Flask/Dash web application must be reachable (e.g. `http://localhost:5000` via Docker Compose or local `python main.py`) — no offline/batch CLI mode is described in the article.

## Evidence

- [results] input Universal Spectrum Identifiers from any public data source that supports USIs through the metabolomics resolver: [evidence span withheld — anchor status: not_found]

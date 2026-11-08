---
name: lc-ms-peak-screening-and-identification
description: Use when performing semi-targeted LC-MS metabolomics analysis in MetaPro
  after peak integration, to apply quality-screening criteria and library-based peak
  identification using HMDB/METLIN-style spectral matching.
when_to_use_negative:
- Input is already a curated feature/quantification table with identities assigned
  — no peak screening or spectral matching is needed
- Acquisition is untargeted DDA with no analyte/library table (mz and rt columns)
  available for semi-targeted analysis
- Spectral library quality is poor or lacks relevant metabolite spectra — identification
  accuracy will be unreliable per the article's own caveat
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaPro
  role: Web application executing the quantification pipeline's peak screening and
    identification stages via Method parameters; supports manual Reset integration
    params / Reset TargetRT / Manual Integration operations
  repo: https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- name: AirdPro
  role: Converts vendor LC-MS files to the .aird computational format that MetaPro's
    EIC extraction reads prior to screening and identification
  repo: https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- name: MongoDB
  role: Backend database started alongside MetaPro to persist project, method, and
    analysis results
- name: Redis
  role: In-memory cache/backend service started alongside MetaPro to speed up analysis
    of MS data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-023-02018-6
    title: 'MetaPro: a web-based metabolomics application for LC-MS data batch inspection
      and library curation'
  source_content_hash: sha256:fbc0b0bd00b5ec1e28b8b430f94e61d459653b2de44edf0c0e94d1209c3c6773
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AirdPro
  - MetaPro
  - MongoDB
  - Redis
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lc-ms-peak-screening-and-identification

## Summary

Applies MetaPro's peak screening (five configurable criteria) and peak identification (three strategies) stages of the eight-step targeted quantification pipeline to filter and annotate integrated LC-MS peaks against curated spectral libraries.

## When to use

Use when performing semi-targeted LC-MS metabolomics analysis in MetaPro after peak integration, to apply quality-screening criteria and library-based peak identification using HMDB/METLIN-style spectral matching.

## When NOT to use

- Input is already a curated feature/quantification table with identities assigned — no peak screening or spectral matching is needed
- Acquisition is untargeted DDA with no analyte/library table (mz and rt columns) available for semi-targeted analysis
- Spectral library quality is poor or lacks relevant metabolite spectra — identification accuracy will be unreliable per the article's own caveat

## Inputs

- Integrated peak tables from MetaPro's targeted analysis of TripleTOF or QE HF batches (converted .aird files)
- Analyte and internal standard library tables (.txt/.tsv with at least mz and rt columns)
- Curated MS/MS spectral libraries (e.g., HMDB, METLIN, or self-made homemade libraries)

## Outputs

- Screened peak sets filtered by the chosen peak screening criterion
- Metabolite identifications assigned by the chosen peak identification strategy
- Per-overview targeted detection quantification results exported as a .xlsx report

## How to apply

After EIC extraction (against the Aird computational format), smoothing, noise estimation, peak selection and integration are run, feed integrated peaks into MetaPro's screening and identification stages. Select one of the five peak screening criteria via the Method-like parameter set to retain peaks passing quality thresholds, then choose one of the three peak identification strategies that compare acquired spectra against curated spectral libraries (e.g., HMDB or METLIN) to assign metabolite identities. When automatic results are unsatisfactory, use the manual operations 'Reset integration params', 'Reset TargetRT', or 'Manual Integration' (choosing two data points for two-point start/end re-integration) and re-apply the changes batch-wide. Rationale: identification accuracy depends on spectral library quality, so curating high-quality authentic spectra before identification improves assignments. Export the final per-overview targeted detection results to a named .xlsx quantification report.

## Related tools

- **MetaPro** (Web application executing the quantification pipeline's peak screening and identification stages via Method parameters; supports manual Reset integration params / Reset TargetRT / Manual Integration operations) — https://github.com/CSi-Studio/MetaPro/releases/tag/v1.1.8
- **AirdPro** (Converts vendor LC-MS files to the .aird computational format that MetaPro's EIC extraction reads prior to screening and identification) — https://github.com/CSi-Studio/AirdPro/releases/tag/1.1
- **MongoDB** (Backend database started alongside MetaPro to persist project, method, and analysis results)
- **Redis** (In-memory cache/backend service started alongside MetaPro to speed up analysis of MS data)

## Examples

```
bash MetaproStart.sh  # then open localhost:8080, Create Project/Library, Set Method parameters (5 screening criteria, 3 identification strategies), and click Target Search
```

## Evaluation signals

- The Method page exposes exactly 5 peak screening criteria and 3 peak identification methods selectable via the parameter set
- The pipeline runs end-to-end on the tutorial TripleTOF (16 files) and QE HF (20 files) datasets loaded from .aird conversions
- A non-empty .xlsx quantification report is produced for each overview after identification
- Peaks failing the chosen screening criterion are excluded from the exported results, and manually re-integrated peaks (two-point start/end) update batch-wide
- Identified metabolites are consistent with matches against the curated HMDB/METLIN-style spectral library used

## Limitations

- Metabolite identification via spectral database comparison is highly dependent on spectral library quality; public or commercial libraries lack spectra of many metabolites
- MetaPro's tutorial data covers DDA-acquired data; other acquisition modes may not be supported in the described workflow
- MetaPro v1.1.8 is for academic use only; commercial use requires contacting the developers

## Evidence

- [methods] exposes four EIC smoothing methods, three EIC noise estimation methods, three peak selection methods, two peak integration methods, five peak screening criteria, and three peak identification methods: "five peak screening criteria, and three peak identification methods"
- [intro] comparing a given spectrum with curated spectral databases such as HMDB (Wishart et al., 2022) and METLIN (Smith et al., 2005): "comparing a given spectrum with curated spectral databases such as HMDB (Wishart et al., 2022) and METLIN (Smith et al., 2005)"
- [abstract] With curated authentic or high-quality spectra, it can improve identification accuracy using different peak identification strategies.: "With curated authentic or high-quality spectra, it can improve identification accuracy using different peak identification strategies."
- [readme] Click on `Manual Integration`, you can choose two data points to conduct manual integration functions.: "Click on `Manual Integration`, you can choose two data points to conduct manual integration functions."
- [intro] the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library: "the metabolite identification accuracy of this method is highly dependent on the quality of the spectral library"

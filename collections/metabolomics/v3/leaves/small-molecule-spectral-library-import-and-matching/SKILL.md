---
name: small-molecule-spectral-library-import-and-matching
description: Decision support for building small-molecule identification/confirmation
  workflows in Skyline when candidate metabolite targets need spectral confirmation
  against public fragmentation libraries rather than in-house spectral libraries.
when_to_use_negative:
- When the goal is untargeted feature finding and alignment — Skyline's paradigm is
  extracting known targets followed by detection and quantification, not MS1 feature-finding
  workflows like XCMS, MZmine 2, or MS-DIAL
- When a fully validated, in-house small-molecule spectral library already exists
  and no external public-library import is needed
- When latest-generation or vendor-specific workflows are required — the paper cautions
  Skyline may lag commercial software until support is implemented
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Skyline
  role: Targeted small-molecule platform into which fragmentation libraries are imported
    and against which extracted chromatograms are scored; extension code lives in
    pwiz_tools/Skyline
  repo: https://github.com/ProteoWizard/pwiz-cpp
- name: METLIN
  role: Public small-molecule MS/MS fragmentation library source for import
- name: MassBank
  role: Public repository of mass spectral data used as a fragmentation-library source
- name: HMDB 4.0
  role: Human Metabolome Database used as a fragmentation-library source
- name: Panorama
  role: Web-based repository for sharing the Skyline documents and raw data used in
    the matching workflow
  repo: https://panoramaweb.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.9b00640
    title: 'Skyline for Small Molecules: A Unifying Software Package for Quantitative
      Metabolomics'
  source_content_hash: sha256:0cf1363ee3a4c1d344361404e20c7b7506ade1324032fac40f9844b1a1a0df15
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ProteoWizard/pwiz-cpp
  tools:
  - HMDB
  - HMDB 4.0
  - METLIN
  - MassBank
  - Panorama
  - Skyline
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# small-molecule-spectral-library-import-and-matching

## Summary

A method for extending Skyline's small-molecule targeted analysis with externally sourced fragmentation libraries (METLIN, MassBank, HMDB, NIST), then scoring extracted fragment-ion chromatograms against those library entries for SRM and HRMS acquisitions.

## When to use

Decision support for building small-molecule identification/confirmation workflows in Skyline when candidate metabolite targets need spectral confirmation against public fragmentation libraries rather than in-house spectral libraries.

## When NOT to use

- When the goal is untargeted feature finding and alignment — Skyline's paradigm is extracting known targets followed by detection and quantification, not MS1 feature-finding workflows like XCMS, MZmine 2, or MS-DIAL
- When a fully validated, in-house small-molecule spectral library already exists and no external public-library import is needed
- When latest-generation or vendor-specific workflows are required — the paper cautions Skyline may lag commercial software until support is implemented

## Inputs

- Small-molecule fragmentation library entries (spectra, precursor and fragment m/z, collision energies) from METLIN, MassBank, HMDB, or NIST
- Vendor-converted LC-MS/MS raw data files (SRM and/or HRMS acquisitions) loadable into Skyline, e.g. via the Panorama Public deposit from the study
- Skyline document with small-molecule targets and configured transition settings (small-molecule tolerance settings, retention-time windows)
- Optional transition list with heavy-labeled internal standards for the target panel

## Outputs

- Imported fragmentation library attached to a Skyline small-molecule document
- Per-target match table with spectral match scores, matched fragment counts, and library accession numbers
- Agreement summary of library-matched identifications between SRM and HRMS acquisitions

## How to apply

Per the source paper this is explicitly future work, not a shipped Skyline feature, so implement it as an extension: (1) acquire library entries from METLIN, MassBank, HMDB, or NIST containing precursor/fragment m/z, collision energies, and spectra; (2) convert them into Skyline transition-list/spectral-library form and import them alongside a small-molecule Skyline document configured for SRM or HRMS extraction; (3) extract fragment-ion chromatograms for each target and score peaks against the imported library using dot-product matching over shared fragments; (4) filter matches by retention-time and product-ion m/z agreement within the document's small-molecule tolerance settings; (5) export per-target match scores, matched fragment counts, and library accession numbers. Because Skyline lives in pwiz_tools/Skyline inside the ProteoWizard/pwiz repository, library import is most naturally added there following its modular, pluggable library framework and C# conventions.

## Related tools

- **Skyline** (Targeted small-molecule platform into which fragmentation libraries are imported and against which extracted chromatograms are scored; extension code lives in pwiz_tools/Skyline) — https://github.com/ProteoWizard/pwiz-cpp
- **METLIN** (Public small-molecule MS/MS fragmentation library source for import)
- **MassBank** (Public repository of mass spectral data used as a fragmentation-library source)
- **HMDB 4.0** (Human Metabolome Database used as a fragmentation-library source)
- **Panorama** (Web-based repository for sharing the Skyline documents and raw data used in the matching workflow) — https://panoramaweb.org/

## Evaluation signals

- Exported match table contains scored entries for every queried library accession (no missing accessions)
- SRM vs HRMS agreement summary is cross-checkable against manually inspected Skyline chromatograms
- Matched fragments are consistent with the imported library entries (dot-product peak match over shared fragments, plus product-ion m/z agreement within small-molecule tolerance settings)
- Retention-time agreement of matched targets falls within the document's scheduled time window (e.g. the 1-minute window centered on measured retention times used in the paper's method export)

## Limitations

- Fragmentation-library workflows for Metlin, MassBank, HMDB, and NIST are NOT yet implemented in Skyline — the paper lists them only as future work, so any implementation is custom
- Skyline may lag behind commercial software for latest-generation or vendor-specific workflows until support is implemented
- No single tool will be best for all workflows; expansion of GC-MS and ion mobility small-molecule support was still ongoing

## Evidence

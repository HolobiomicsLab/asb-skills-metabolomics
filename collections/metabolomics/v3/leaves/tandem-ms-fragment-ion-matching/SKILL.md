---
name: tandem-ms-fragment-ion-matching
description: Use when per-sample MGF MS2 spectra have been clustered and cleaned into
  consensus spectra and structural annotations must be assigned by spectral-library
  matching, ranked by MQscore and m/z error.
when_to_use_negative:
- Input is raw LC–MS/MS vendor data that has not yet been preprocessed — the pre_process
  step must first detect MS1 features and extract MS2 spectra into MGF files
- The spectra are MS1-only features with no MS2 fragmentation, so there are no fragment
  ions to match against a spectral library
- The goal is molecular-network visualization alone (SSMN in Cytoscape) rather than
  structural annotation of consensus spectra
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Tremolo
  role: Searches clean consensus spectra against the ISDB MS2 library for structural
    annotation in step 6 of the NP3 MS Workflow
- name: NP3 MS Workflow
  role: Wraps the library-identification step (step 6) and the gnps_library_search
    GNPS2 experimental-library search (step 6.1) within the run command
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:ae9abbbf6b8ede8f61eb3aee4b3b2992972546ba75790a18cd9f70fb52d7cae0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielatrivella/NP3_MS_Workflow
  repo_url: https://github.com/danielatrivella/NP3_MS_Workflow
  tools:
  - NP3 MS Workflow
  - Tremolo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Tandem MS fragment ion matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates clean consensus MS2 spectra by matching their fragment ions against spectral libraries (In Silico MS2 DataBase / GNPS) using the Tremolo tool within the NP3 MS Workflow, ranking candidates by MQscore and precursor m/z error in ppm.

## When to use

Use when per-sample MGF MS2 spectra have been clustered and cleaned into consensus spectra and structural annotations must be assigned by spectral-library matching, ranked by MQscore and m/z error.

## When NOT to use

- Input is raw LC–MS/MS vendor data that has not yet been preprocessed — the pre_process step must first detect MS1 features and extract MS2 spectra into MGF files
- The spectra are MS1-only features with no MS2 fragmentation, so there are no fragment ions to match against a spectral library
- The goal is molecular-network visualization alone (SSMN in Cytoscape) rather than structural annotation of consensus spectra

## Inputs

- Clean consensus MS2 spectra from NP3 MS Workflow cleaning step 5 (per-sample MGF files in Mascot Generic Format)
- Metadata Table describing LC–MS/MS samples and sample groups
- MS2 spectral library: In Silico MS2 DataBase (ISDB) / UNPD-enriched library, or the locally retrieved ALL_GNPS_NO_PROPOGATED.mgf split by ion mode
- ion_mode parameter (positive or negative) for the gnps_library_search command

## Outputs

- Curated library annotation table with columns prefixed 'curated_lib_annotation_*'
- Ranked annotation candidates per clean consensus spectrum with MQscore and m/z error (ppm)
- Fragment ion peak matches supporting each annotation

## How to apply

Run Library Identification (step 6) of the NP3 MS Workflow, which uses Tremolo to search clean consensus spectra — produced by the clustering, quantification and cleaning steps (3–5) using the NP3-shifted cosine — against the In Silico MS2 DataBase (ISDB) enriched with UNPD origin and chemical-class information. Supply the ion_mode parameter (positive or negative) so the library search selects the appropriate library split, e.g. the ALL_GNPS library is split into positive- and negative-ion MGF files in setup (v1.6.0). Judge candidates by MQscore and precursor m/z error in ppm, then curate the final annotation columns prefixed 'curated_lib_annotation_*'. Optionally execute the gnps_library_search command (automated as step 6.1 of run) to match against experimental GNPS2 libraries offline. Correctness is checked against the BRA-346 case study, where m/z 343.187 (rt 4.24 min) was annotated as TMC-86A with MQscore 0.27, m/z error 1.87 ppm, and fragment ions 325.176, 251.103, 186.113, 168.102, 150.091, and 130.050.

## Related tools

- **Tremolo** (Searches clean consensus spectra against the ISDB MS2 library for structural annotation in step 6 of the NP3 MS Workflow)
- **NP3 MS Workflow** (Wraps the library-identification step (step 6) and the gnps_library_search GNPS2 experimental-library search (step 6.1) within the run command) — https://github.com/danielatrivella/NP3_MS_Workflow

## Evaluation signals

- Reproduce Table 1 of the BRA-346 case study: m/z 343.187 (rt 4.24 min) annotated as TMC-86A with MQscore 0.27 and m/z error 1.87 ppm
- Verify the reported fragment-ion matches for the top annotation (325.176, 251.103, 186.113, 168.102, 150.091, 130.050) appear among matched peaks
- Check that the ion_mode-appropriate library (positive- or negative-ion MGF) was selected for the search, per the v1.6.0 library split
- Confirm the output uses the 'curated_lib_annotation_*' column naming from Manual Section 4.6.5 (v1.6.0)
- Verify annotations are ranked by MQscore and that in-silico (ISDB) suggestions are distinguishable from experimental GNPS2 library hits

## Limitations

- Primary search is against in silico spectra (ISDB), so annotations are candidate suggestions ranked by MQscore and require manual curation rather than definitive identifications
- A v1.5.1 bug fix notes that when the noise cutoff is applied, filtered IVAMNs must reset their index to prevent matching errors when joining jobs (join_jobs step 7)
- The ALL_GNPS_NO_PROPOGATED experimental library annotations were enriched only up to April 2026 in the local setup; re-running setup is required after version changes

## Evidence

- [methods] TMC-86A annotation validation details: [evidence span withheld — anchor status: not_found]

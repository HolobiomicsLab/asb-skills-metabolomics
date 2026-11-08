---
name: spectral-library-annotation-isdb
description: Use when clean consensus spectra produced by NP3 MS Workflow clustering/cleaning
  need chemical structural annotation against ISDB via Tremolo, e.g. to identify a
  top-ranked bioactive ion such as m/z 343.187 (BCC 0.80) as TMC-86A.
when_to_use_negative:
- Input is raw vendor LC–MS/MS data or per-sample MGFs that have not been clustered
  and cleaned into clean consensus spectra (steps 3–5 not yet run)
- The task is peptide/proteomics database searching or genome-based metabolite prediction
  rather than small-molecule MS2 library matching of natural products
- Spectra are already annotated and curated; re-running identification would only
  duplicate the existing curated_lib_annotation_* columns
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2258
tools:
- name: NP3 MS Workflow
  role: Executes Library Identification (step 6) on clean consensus spectra and exposes
    the gnps_library_search command (step 6.1) for integrated GNPS2 library search
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Tremolo
  role: Spectral-search engine used in step 6 to match clean consensus spectra against
    the ISDB, returning MQscore-ranked annotations
- name: In Silico MS2 DataBase (ISDB)
  role: Target spectral library (with UNPD compound origin/chemical class information)
    against which clean consensus spectra are searched
- name: GNPS (Global Natural Products Social Molecular Networking)
  role: Source of experimental spectral libraries (GNPS2 Library Search workflow,
    CCMSLIB entries such as CCMSLIB00000840551) for corroboration of ISDB annotations
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:def63c93cad08688cfbfc452690a4ade9c1b17998940747d459a51e64d21ca55
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
  - Cytoscape
  - GNPS
  - GNPS (Global Natural Products Social Molecular Networking)
  - In Silico MS2 DataBase (ISDB)
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  - Tremolo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spectral-library-annotation-isdb

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Structurally annotates clean consensus MS2 spectra from the NP3 MS Workflow by searching them against the In Silico MS2 DataBase (ISDB) with Tremolo, ranking candidate structures by MQscore and m/z error, and optionally against GNPS/GNPS2 experimental libraries.

## When to use

Use when clean consensus spectra produced by NP3 MS Workflow clustering/cleaning need chemical structural annotation against ISDB via Tremolo, e.g. to identify a top-ranked bioactive ion such as m/z 343.187 (BCC 0.80) as TMC-86A.

## When NOT to use

- Input is raw vendor LC–MS/MS data or per-sample MGFs that have not been clustered and cleaned into clean consensus spectra (steps 3–5 not yet run)
- The task is peptide/proteomics database searching or genome-based metabolite prediction rather than small-molecule MS2 library matching of natural products
- Spectra are already annotated and curated; re-running identification would only duplicate the existing curated_lib_annotation_* columns

## Inputs

- Clean consensus spectra (MS2) from NP3 MS Workflow clustering/cleaning, per-sample MGF files and final consensus MGF
- In Silico MS2 DataBase (ISDB) / UNPD-ISDB spectral library (retrieved and enriched during the setup command)
- ALL_GNPS_NO_PROPOGATED.mgf library split by ion mode (positive/negative) for GNPS library search
- Ion mode of the run (e.g. positive mode for the BRA-346 and MA9 case studies)

## Outputs

- Library annotation table mapping each clean consensus spectrum (m/z, rt) to candidate structures ranked by MQscore with m/z error in ppm and fragment-ion matches
- Final curated library annotation columns (curated_lib_annotation_*) and chemical statistics/reports (e.g. superclass distribution)
- Grafana-style chemical-space PCA report of identified compounds for comparison against a reference dataset

## How to apply

After NP3 MS Workflow steps 3–5 have produced clean consensus spectra (clustered with the NP3-shifted cosine), run Library Identification (step 6): Tremolo searches each clean consensus spectrum against the In Silico MS2 DataBase (ISDB), returning candidate structures ranked by MQscore with the corresponding m/z error in ppm. Inspect the top suggestions for the prioritized ion — for m/z 343.187 in the BRA-346 data set (MSV00091453), TMC-86A was among the suggestions with MQscore 0.27 and m/z error 1.87 ppm, supported by fragment ions 325.176, 251.103, 186.113, 168.102, 150.091, and 130.050. For cross-checking against experimental spectra, a GNPS library match can corroborate the annotation (e.g. m/z 679.272, TMC-95, score 0.75, 0.09 ppm error); from version 1.6.0 the ALL_GNPS MGF library is split by ion mode (positive/negative) and the ion_mode parameter selects the appropriate library, while the gnps_library_search command (step 6.1) runs GNPS2 Library Search fully integrated with NP3 results. Remember ISDB is an in silico database, so results are suggestions that should be curated using MQscore, ppm error, and fragment-ion agreement rather than accepted blindly.

## Related tools

- **NP3 MS Workflow** (Executes Library Identification (step 6) on clean consensus spectra and exposes the gnps_library_search command (step 6.1) for integrated GNPS2 library search) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Tremolo** (Spectral-search engine used in step 6 to match clean consensus spectra against the ISDB, returning MQscore-ranked annotations)
- **In Silico MS2 DataBase (ISDB)** (Target spectral library (with UNPD compound origin/chemical class information) against which clean consensus spectra are searched)
- **GNPS (Global Natural Products Social Molecular Networking)** (Source of experimental spectral libraries (GNPS2 Library Search workflow, CCMSLIB entries such as CCMSLIB00000840551) for corroboration of ISDB annotations)

## Evaluation signals

- Reproducing the BRA-346 case study must return m/z 343.187 annotated as TMC-86A with MQscore 0.27 and m/z error 1.87 ppm, matching Table 1
- The annotation's fragment ions must include 325.176, 251.103, 186.113, 168.102, 150.091, and 130.050
- For the MA9 data set, the m/z 679.272 variant must receive a GNPS spectral library match with score 0.75 and m/z error 0.09 ppm
- The correct ion-mode library must be selected: the ALL_GNPS MGF is split into positive and negative ion files, chosen via the ion_mode parameter of gnps_library_search
- Curated annotation columns must be named with the curated_lib_annotation_* prefix, and GNPS identifications counted via gnps_SpectrumID rather than possibly-empty SMILES

## Limitations

- ISDB is an in silico database, so annotations are ranked suggestions (e.g. MQscore 0.27 for TMC-86A) requiring manual curation rather than definitive identifications
- After upgrading to version 1.6.0 (ion-mode split of the ALL_GNPS library) or 1.4.0 (GNPS2 offline libraries), the setup command must be re-executed for the changes to take effect
- GNPS2 library annotations were pre-enriched at a fixed date (April 2026 snapshot retrieved locally during setup), so coverage is not live-updated offline

## Evidence

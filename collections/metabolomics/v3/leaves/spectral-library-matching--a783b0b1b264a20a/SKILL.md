---
name: spectral-library-matching
description: Use when you have NP3 MS Workflow clean consensus spectra (post-clustering,
  post-cleaning) and need to assign structural annotations to m/z ions — e.
when_to_use_negative:
- Input is raw vendor LC-MS/MS data that has not yet been preprocessed and clustered
  into clean consensus spectra by NP3 steps 2–5 — the library search requires those
  spectra as input.
- The goal is quantification or bioactivity-correlation ranking (BCC computation)
  without needing structural annotation — use the quantification/biocorrelation step
  instead.
- The compound of interest has no experimental GNPS spectrum and in-silico ISDB matching
  is not acceptable for the validation standard — spectral library matching alone
  cannot confirm structure.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: NP3 MS Workflow
  role: Executes Library Identification (step 6) via the run command; performs the
    setup that retrieves and ion-mode-splits the GNPS library and enriches annotations.
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Tremolo
  role: The spectral-matching engine that searches clean consensus spectra against
    the UNPD-ISDB library.
- name: GNPS (Global Natural Products Social Molecular Networking)
  role: Source of experimental spectral libraries (ALL_GNPS_NO_PROPOGATED) and deposited
    reference spectra (e.g. CCMSLIB00000840551, CCMSLIB00000840365) used as match
    targets.
- name: Cytoscape
  role: Visualization of the annotated SSMN to inspect library-annotation assignments
    on molecular-network nodes.
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:1bc1f7389bc3e0f200907996d23b5d5d89f9082deee8f9990ede2cef07b78ae3
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
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  - Tremolo
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-library-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches clean consensus MS2 spectra against spectral libraries (GNPS experimental data or the in-silico UNPD-ISDB) via the Tremolo tool to annotate LC-MS/MS features within the NP3 MS Workflow, assigning library hits and chemical class/origin information used in downstream bioactivity-correlation ranking.

## When to use

Use when you have NP3 MS Workflow clean consensus spectra (post-clustering, post-cleaning) and need to assign structural annotations to m/z ions — e.g. to identify a candidate like TMC-95 or a 4-deoxyphorbol ester — or when you need annotation quality (score and m/z error) to validate a bioactivity-ranked hit.

## When NOT to use

- Input is raw vendor LC-MS/MS data that has not yet been preprocessed and clustered into clean consensus spectra by NP3 steps 2–5 — the library search requires those spectra as input.
- The goal is quantification or bioactivity-correlation ranking (BCC computation) without needing structural annotation — use the quantification/biocorrelation step instead.
- The compound of interest has no experimental GNPS spectrum and in-silico ISDB matching is not acceptable for the validation standard — spectral library matching alone cannot confirm structure.

## Inputs

- Clean consensus spectra (MGF-format) from NP3 MS Workflow steps 3–5, per ion mode
- A spectral library: UNPD-ISDB (in-silico, from UNPD) or the pre-split ALL_GNPS_NO_PROPOGATED.mgf (positive/negative) retrieved during the setup command
- ion_mode parameter (positive or negative) declared by the user
- Optionally, deposited GNPS reference spectra (CCMSLIB accession IDs) for validating known compounds

## Outputs

- Library annotation table per clean consensus spectrum with SpectrumID, SMILES, origin, and chemical-class metadata (curated_lib_annotation_* columns)
- GNPS-style match score and precursor m/z error (ppm) per annotation
- Annotated spectra feeding the IVAMN and the filtered SSMN (visualizable in Cytoscape)
- Chemical statistics in the final reports (counts via gnps_SpectrumID, superclass distribution)

## How to apply

After NP3 steps 3–5 (clustering, quantification, cleaning using the NP3-shifted cosine function) produce clean consensus spectra, run Library Identification (step 6) with the Tremolo tool to search those spectra against a spectral library — the In Silico MS2 DataBase (UNPD-ISDB, generated from UNPD with origin and chemical-class enrichment) for de novo/dereplication candidates, or the deposited experimental GNPS library for known compounds. If using the offline GNPS2 route, execute the setup command first to retrieve ALL_GNPS_NO_PROPOGATED.mgf, split by ion mode, and enrich annotations; the gnps_library_search / run command's step 6.1 then searches clean consensus spectra, choosing the positive- or negative-ion library depending on the declared ion_mode. Annotation results feed the IVAMN/PageRank [M+H]+ deconvolution (step 8) and the SSMN, and the resulting annotations (e.g. GNPS score 0.75 with 0.09 ppm error for m/z 679.272 → TMC-95, or CCMSLIB00000840365 for m/z 607.32) are cross-checked against fragment ions and retention time before filtering BCCs against blank/culture media. Decision point: use experimental GNPS matching when deposited reference spectra exist (e.g. CCMSLIB00000840551/552 for 4-deoxyphorbol esters); use in-silico ISDB when they do not.

## Related tools

- **NP3 MS Workflow** (Executes Library Identification (step 6) via the run command; performs the setup that retrieves and ion-mode-splits the GNPS library and enriches annotations.) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Tremolo** (The spectral-matching engine that searches clean consensus spectra against the UNPD-ISDB library.)
- **GNPS (Global Natural Products Social Molecular Networking)** (Source of experimental spectral libraries (ALL_GNPS_NO_PROPOGATED) and deposited reference spectra (e.g. CCMSLIB00000840551, CCMSLIB00000840365) used as match targets.)
- **Cytoscape** (Visualization of the annotated SSMN to inspect library-annotation assignments on molecular-network nodes.)

## Examples

```
nextflow run main.np3 --mode run --inputdir <mgf_dir> --metadata <metadata.tsv> --ion_mode positive  # executes steps 2–10 incl. step 6.1 GNPS2 library search
```

## Evaluation signals

- For known compounds, the returned annotation must reproduce the deposited accession: e.g. m/z 679.272 (rt 5.65 min) annotated as TMC-95 with GNPS score 0.75 and m/z error 0.09 ppm.
- In Data set I, the m/z 607.32 clean consensus spectrum must be annotated to CCMSLIB00000840365 (4-deoxyphorbol ester analogue), and the 11 m/z 589.31 isomers must all receive GNPS annotations.
- Ion-mode correctness: the gnps_library_search must select the positive-ion library file for [M+H]+ analyses (or negative for negative mode) — cross-check that the declared ion_mode matches the chosen ALL_GNPS split.
- Annotation coverage sanity: chemical statistics in the final reports must count GNPS identifications by gnps_SpectrumID (not SMILES), and superclass distributions must reflect the occurrence of annotated m/z ions after removing blanks and bed samples.
- Retrieved SMILES from GNPS2 Library Search must pass the setup validation routine before being used in enrichment.

## Limitations

- Offline GNPS2 identification requires re-running the setup command to retrieve and ion-mode-split ALL_GNPS_NO_PROPOGATED.mgf; annotations were pre-enriched on a fixed date (April 2026), so newer library entries are not visible offline.
- In-silico UNPD-ISDB matches are structural hypotheses, not confirmed spectra — the workflow's own reported quality metrics (e.g. GNPS score 0.75, 0.09 ppm) are needed to assert annotation confidence.
- Matches are only as good as the library coverage: ions absent from GNPS or the ISDB remain unannotated and will be missed in the chemical statistics.
- Noise cutoff applied at the clustering step (now an absolute base-peak-intensity threshold) removes low-intensity MS2 before the library search, so minority compounds below that cutoff cannot be matched.

## Evidence

- [methods] annotated as TMC-95 with GNPS score 0.75 and m/z error 0.09 ppm: [evidence span withheld — anchor status: not_found]

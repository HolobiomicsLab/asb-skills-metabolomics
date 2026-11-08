---
name: bioactivity-correlation-scoring
description: Use when a proteomics-style LC–MS/MS metabolomics run has per-fraction
  bioactivity values (IC50 or inhibition groups) in the NP3 Metadata Table and you
  must rank clean consensus spectra by bioactivity correlation to prioritize natural-product
  leads.
when_to_use_negative:
- The input is already a ranked feature table without raw LC–MS/MS files or bioactivity
  metadata — the skill requires the full Metadata Table plus LC–MS/MS data to compute
  clean consensus spectra and BCCs.
- The analysis goal is purely chemical (e.g. molecular networking without any bioactivity
  information) — BCC scoring is meaningless without fraction-level inhibition/IC50
  values.
- Only blank or culture media samples are present — the filter step would remove every
  ion, leaving nothing to rank.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: NP3 MS Workflow
  role: 'Runs the biocorrelation step (step 9): preprocessing, clustering/cleaning
    of MS2 spectra with the NP3-shifted cosine, and computation of BCCs between clean
    consensus spectra and bioactivity groups, plus filtering against blanks/media.'
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Tremolo
  role: Annotates the top-ranked BCC ions by searching clean consensus spectra against
    the In Silico MS2 DataBase (ISDB), yielding MQscore and m/z error (e.g. TMC-86A,
    MQscore 0.27, 1.87 ppm).
- name: GNPS2 Library Search
  role: Optional offline/experimental library identification (step 6.1) used to confirm
    annotations of high-bioactivity ions (e.g. TMC-95, GNPS score 0.75, 0.09 ppm).
- name: Cytoscape
  role: Visualization of the filtered SSMN and mounting of reports from the clean
    count table after BCC ranking.
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:c59b7f6dd793b224322c804efbd4364087100f7fed937ed0281271c3899aa0af
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
  - GNPS2 Library Search
  - NP3 MS Workflow
  - Tremolo
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# bioactivity-correlation-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes bioactivity correlation coefficients (BCCs) between clean consensus MS2 spectra and fraction-level bioactivity measurements (e.g. proteasome inhibition), then filters and ranks the spectra so that the ions most correlated with activity surface as drug-discovery leads in the NP3 MS Workflow.

## When to use

Use when a proteomics-style LC–MS/MS metabolomics run has per-fraction bioactivity values (IC50 or inhibition groups) in the NP3 Metadata Table and you must rank clean consensus spectra by bioactivity correlation to prioritize natural-product leads.

## When NOT to use

- The input is already a ranked feature table without raw LC–MS/MS files or bioactivity metadata — the skill requires the full Metadata Table plus LC–MS/MS data to compute clean consensus spectra and BCCs.
- The analysis goal is purely chemical (e.g. molecular networking without any bioactivity information) — BCC scoring is meaningless without fraction-level inhibition/IC50 values.
- Only blank or culture media samples are present — the filter step would remove every ion, leaving nothing to rank.

## Inputs

- Metadata Table describing LC–MS/MS samples and sample groups with bioactivity values (e.g. proteasome inhibition groups or IC50-derived scores)
- Raw LC–MS/MS data files (positive mode) from MassIVE/GNPS deposits such as MSV00091453 and MSV000091455
- Clean consensus spectra produced by the NP3 MS Workflow clustering/cleaning steps (per-sample MGF-derived)

## Outputs

- BCC-ranked table of clean consensus spectra (e.g. bcc_ranking_table.csv, tmc95_variant_bcc.csv)
- Filtered list of m/z ions absent from blank and culture media samples, ranked by bioactivity correlation
- Structural annotations for top-ranked ions (e.g. TMC-86A, TMC-95) with MQscore/GNPS score and m/z error

## How to apply

Build the Metadata Table linking each LC–MS/MS file (extracts, fractions, blanks, culture media) to its bioactivity score, e.g. the BRA-346 proteasome inhibition groups or the MA9 IC50 series in Supporting Information Notes S13/S14. Run the NP3 MS Workflow preprocessing and the 'run' command so that MS1 features are detected, MS2 spectra are extracted into per-sample MGF files, clustered, quantified and cleaned with the NP3-shifted cosine, and each clean consensus spectrum is matched against ISDB/Tremolo or GNPS2 libraries. Because IC50 values make the correlation inverse, lower BCC means stronger bioactivity association; filter the BCC results to keep only m/z ions absent from blank and culture media samples, then rank the clean consensus spectra by BCC. For example, m/z 343.187 in BRA-346 ranks highest with BCC 0.80, and m/z 679.272 in MA9 ranks lowest with BCC −0.90, discriminating closely related diastereomer variants (−0.60 and −0.56).

## Related tools

- **NP3 MS Workflow** (Runs the biocorrelation step (step 9): preprocessing, clustering/cleaning of MS2 spectra with the NP3-shifted cosine, and computation of BCCs between clean consensus spectra and bioactivity groups, plus filtering against blanks/media.) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Tremolo** (Annotates the top-ranked BCC ions by searching clean consensus spectra against the In Silico MS2 DataBase (ISDB), yielding MQscore and m/z error (e.g. TMC-86A, MQscore 0.27, 1.87 ppm).)
- **GNPS2 Library Search** (Optional offline/experimental library identification (step 6.1) used to confirm annotations of high-bioactivity ions (e.g. TMC-95, GNPS score 0.75, 0.09 ppm).)
- **Cytoscape** (Visualization of the filtered SSMN and mounting of reports from the clean count table after BCC ranking.)

## Examples

```
np3_ms_workflow run --metadata metadata_bra346.tsv --ion_mode positive --steps 2-10 --bcc_ranking_table bcc_ranking_table.csv
```

## Evaluation signals

- The top-ranked ion in BRA-346 must be m/z 343.187 (rt 4.24 min) with BCC 0.80, matching Table 1.
- The lowest-scoring ion in MA9 must be m/z 679.272 (rt 5.65 min) with BCC −0.90, and the co-eluting diastereomer variants m/z 679.2732 and 679.2717 must score −0.60 and −0.56 respectively (Table 2).
- Filtered results must contain no m/z ions detected in blank or culture media samples.
- Annotations of top ions must carry a library score and m/z error within reported ranges (e.g. TMC-95 GNPS score 0.75, 0.09 ppm; TMC-86A MQscore 0.27, 1.87 ppm).
- Consistency of the reproduced BCC ranking across runs must match the stored fixed results used by the workflow's test suite.

## Limitations

- The correlation is inverse when IC50 values are used, so lower (more negative) BCC means stronger bioactivity — sign convention must be respected when ranking.
- Ions present in blank or culture media samples are removed by the filter step, which can discard true hits if the medium contributes to the signal.
- Discrimination of diastereomer-level variants relies on accurate m/z separation (e.g. 679.2717 vs 679.272 vs 679.2732) and may be limited by MS resolving power or spectral library coverage.

## Evidence

- [abstract] relative quantification of the precursor ions for bioactivity correlation scoring: "relative quantification of the precursor ions for bioactivity correlation scoring"
- [methods] Biocorrelation (step 9) computing bioactivity correlation coefficients of each clean consensus spectrum against the IC50-based biocorrelation groups: [evidence span withheld — anchor status: not_found]
- [methods] filtered to only keep the m/z ions that did not appear in the blank or culture media samples: "filtered to only keep the m/z ions that did not appear in the blank or culture media samples"
- [methods] the m/z ion 679.272 was attributed with the lowest score (BCC = −0.90): "the m/z ion 679.272 was attributed with the lowest score (BCC = −0.90)"
- [methods] One clean consensus spectrum was ranked with the highest value: m/z 343.187 (BCC 0.80): "One clean consensus spectrum was ranked with the highest value: m/z 343.187 (BCC 0.80)"

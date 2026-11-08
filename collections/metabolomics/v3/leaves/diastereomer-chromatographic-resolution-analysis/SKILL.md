---
name: diastereomer-chromatographic-resolution-analysis
description: Use when an LC–MS/MS dataset contains multiple closely related chromatographic
  features with nearly identical precursor m/z (e.g., nominal m/z 679.
when_to_use_negative:
- When the bioactive compound is the sole chromatographic peak for a given m/z and
  no diastereomer/isomer siblings of the same nominal mass exist, a standard dereplication
  + Biocorrelation rank is sufficient and variant-level discrimination adds no value
- When BCC scores are not inverse-coded (e.g., the biocorrelation uses a different
  activity metric than IC50 where higher = stronger), the decision logic 'lowest BCC
  = most bioactive' does not apply without re-deriving the mapping between BCC sign
  and activity strength
- When the user does not have measured bioactivity values (e.g., IC50) to correlate
  against, since the discrimination relies entirely on ranking by biocorrelation coefficient,
  not on MS/MS spectra alone
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NP3 MS Workflow
  role: Executes preprocessing, clustering, quantification, cleaning, and Biocorrelation
    (steps 2–9) to produce per-variant BCC scores from the LC–MS/MS dataset and Metadata
    Table, enabling ranking of co-eluting ion variants by bioactivity correlation.
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: GNPS
  role: Spectral library search used to confirm the identity of the top-scoring ion
    variant (e.g., m/z 679.272 as TMC-95), providing a library match score and m/z
    error used as a validation signal.
  repo: https://gnps.ucsd.edu
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:b594541b382d9b27707a93862b5814d1059fc0d0036625761bff22beafdbdd05
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
  - GNPS
  - NP3 MS Workflow
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# diastereomer-chromatographic-resolution-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A bioactivity-correlation-based analysis for resolving diastereomer-level ion variants of the same nominal precursor mass in LC–MS/MS drug-discovery datasets, using NP3 MS Workflow BCC ranking to discriminate which epimer/isomer node is the true bioactive one.

## When to use

Use when an LC–MS/MS dataset contains multiple closely related chromatographic features with nearly identical precursor m/z (e.g., nominal m/z 679.27 variants) and you need to determine which variant correlates most strongly with measured bioactivity, rather than treating them as a single dereplicated hit.

## When NOT to use

- When the bioactive compound is the sole chromatographic peak for a given m/z and no diastereomer/isomer siblings of the same nominal mass exist, a standard dereplication + Biocorrelation rank is sufficient and variant-level discrimination adds no value
- When BCC scores are not inverse-coded (e.g., the biocorrelation uses a different activity metric than IC50 where higher = stronger), the decision logic 'lowest BCC = most bioactive' does not apply without re-deriving the mapping between BCC sign and activity strength
- When the user does not have measured bioactivity values (e.g., IC50) to correlate against, since the discrimination relies entirely on ranking by biocorrelation coefficient, not on MS/MS spectra alone

## Inputs

- LC–MS/MS raw data from MassIVE (e.g., MSV000091455) of a bioactive natural product extract and its derived chromatographic fractions, including blank and culture media samples
- Metadata Table describing the LC–MS/MS samples and sample groups, with measured proteasome-inhibition IC50 values entered for both fractionation series simultaneously
- Bioactivity correlation output from NP3 MS Workflow Biocorrelation (step 9) with per-variant BCC scores, retention times, and precursor m/z for each clean consensus spectrum

## Outputs

- Ranked per-variant BCC table (e.g., tmc95_variant_bcc.csv) listing m/z, retention time, and bioactivity correlation for each chromatographic variant of the target compound family
- Identification of the single bioactive epimer/isomer node (e.g., m/z 679.272, BCC −0.90) distinguished from co-eluting variants (e.g., BCC −0.60 and −0.56) of the same nominal mass
- GNPS spectral library match confirmation for the top-scoring variant (e.g., score 0.75, m/z error 0.09 ppm)

## How to apply

1) Process the LC–MS/MS data (e.g., MSV000091455 from A. moriforme MA9 extract and derived chromatographic fractions) with NP3 MS Workflow, positive mode, including preprocessing, clustering, quantification, cleaning, and Biocorrelation against the measured IC50 values, entering the measured proteasome-inhibition IC50 values for both fractionation series simultaneously via the Metadata Table (per Supporting Information Note S14). 2) Because IC50 values are used, the BCC is inverse (lower = stronger bioactivity association). 3) Filter the BCC results to retain only m/z ions absent from blank and culture media samples, then rank the clean consensus spectra by BCC. 4) Extract the BCC rows for the co-eluting isomer nodes of interest (e.g., m/z 679.272 rt 5.65 min, m/z 679.2732 rt 5.25 min, m/z 679.2717 rt 5.50 min) and compare their scores: the node with the lowest (most negative) BCC is the bioactive epimer. 5) Validate the annotation of the top-scoring node against the GNPS spectral library (e.g., score 0.75, m/z error 0.09 ppm) to confirm identity (TMC-95). 6) Write the variant-level BCC comparison to a CSV such as tmc95_variant_bcc.csv. The rationale: NP3 MS Workflow's shifted-cosine-based clean consensus spectra plus IC50-based Biocorrelation can discriminate the true bioactive ion variant from its epimer/isomer siblings of the same nominal mass, as demonstrated by BCC = −0.90 for the TMC-95 node versus −0.60 and −0.56 for its close variants.

## Related tools

- **NP3 MS Workflow** (Executes preprocessing, clustering, quantification, cleaning, and Biocorrelation (steps 2–9) to produce per-variant BCC scores from the LC–MS/MS dataset and Metadata Table, enabling ranking of co-eluting ion variants by bioactivity correlation.) — https://github.com/danielatrivella/NP3_MS_Workflow
- **GNPS** (Spectral library search used to confirm the identity of the top-scoring ion variant (e.g., m/z 679.272 as TMC-95), providing a library match score and m/z error used as a validation signal.) — https://gnps.ucsd.edu

## Examples

```
nextflow run danielatrivella/NP3_MS_Workflow --metadata tmc95_metadata.csv --ion_mode positive --run biocorrelation --output tmc95_bcc_results.csv
```

## Evaluation signals

- The reproduced BCC values must match Table 2 of the paper, with m/z 679.272 at BCC −0.90 as the lowest-scoring (most bioactive) isomer and the two co-eluting diastereomer nodes at BCC −0.60 and −0.56.
- The m/z ions retained for ranking must all be absent from the blank and culture media samples; any variant detected in blanks must be excluded before ranking.
- The top-scoring variant must be independently confirmed by GNPS spectral library match (e.g., score 0.75, m/z error 0.09 ppm for TMC-95) so that the bioactivity attribution is not a false-positive annotation.
- All BCC values must be interpreted with the correct sign convention: because IC50 values are used, lower (more negative) BCC indicates stronger bioactivity association, not weaker.
- Extracted ion chromatograms for the ion at m/z 679.272 (rt 5.65 min) should show the highest intensities in the inhibitory fractions, corroborating the BCC-based attribution.

## Limitations

- NP3 MS Workflow's Biocorrelation is built around IC50-style inhibitory activity, so the workflow's default interpretation of BCC sign is inverse; applying it to other activity readouts requires re-deriving the mapping between BCC sign and activity strength.
- Discrimination at the diastereomer level relies on the chromatographic separation of the variants (distinct retention times of 5.65, 5.25, and 5.50 min); variants that co-elute exactly cannot be separated by this workflow.
- The method is validated for positive-mode LC–MS/MS; applying it to negative-mode data would require re-validation of the NP3-shifted-cosine clean consensus and Biocorrelation behavior.

## Evidence

---
name: blank-and-media-ion-filtering
description: Use when running the NP3 MS Workflow bioactivity-correlation step on
  fractionated bioassay LC–MS/MS data (e.g. MSV000091455 or MSV00091453) and blank
  injection and culture-media/bed control samples are available in the Metadata Table.
when_to_use_negative:
- The data set has no blank injection or culture-media (bed) control samples, so no
  media-derived m/z can be excluded on occurrence evidence.
- You are using pre-filtered feature or ion tables (e.g. an external annotation table)
  rather than NP3 MS Workflow clean consensus spectra, where per-sample occurrence
  is already resolved.
- The goal is peak-level MS1 quantification for statistics (e.g. PCA or differential
  abundance) rather than BCC ranking of clean consensus spectra for bioactivity prioritization.
tools:
- name: NP3 MS Workflow
  role: Produces the clean consensus spectra, per-sample occurrence information, and
    since v1.5.0 automatically excludes m/z appearing in bed samples from [M+H]+ network
    construction; the BCC filter for blank/media ions is applied on its step 9 output
    before step 10 (SSMN with damped pruning) and BCC ranking.
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: GNPS
  role: Downstream annotation of the surviving filtered ions via GNPS/Tremolo library
    search (e.g. TMC-95 match at score 0.75, m/z error 0.09 ppm) so that top-ranked
    filtered ions can be assigned structures.
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:225fee37eec11ece14d38626a1fca506aac7235968617a1f4923e255b35bd770
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
  - NP3 MS Workflow
  - Tremolo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# blank-and-media-ion-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removes MS1 feature ions that also appear in LC–MS/MS blank or culture-media samples from the NP3 MS Workflow's clean consensus spectra before bioactivity correlation (BCC) ranking, so background and medium-derived contaminants cannot rank as top bioactive hits.

## When to use

Use when running the NP3 MS Workflow bioactivity-correlation step on fractionated bioassay LC–MS/MS data (e.g. MSV000091455 or MSV00091453) and blank injection and culture-media/bed control samples are available in the Metadata Table.

## When NOT to use

- The data set has no blank injection or culture-media (bed) control samples, so no media-derived m/z can be excluded on occurrence evidence.
- You are using pre-filtered feature or ion tables (e.g. an external annotation table) rather than NP3 MS Workflow clean consensus spectra, where per-sample occurrence is already resolved.
- The goal is peak-level MS1 quantification for statistics (e.g. PCA or differential abundance) rather than BCC ranking of clean consensus spectra for bioactivity prioritization.

## Inputs

- Clean consensus spectra with BCC values from NP3 MS Workflow steps 3–9
- Metadata Table describing LC–MS/MS samples and sample groups (blank, culture media/bed, and bioactive fraction groups with IC50 bioactivity scores)

## Outputs

- BCC-filtered and BCC-ranked clean consensus spectra table (bcc_ranking_table.csv) excluding blank/media ions
- Filtered [M+H]+ SSMN networks lacking bed-sample-derived nodes (NP3 MS Workflow ≥1.5.0 default)

## How to apply

After NP3 MS Workflow steps 3–9 produce clean consensus spectra with BCC values (computed against IC50-based groups, where lower/negative BCC indicates stronger bioactivity association), apply this filter before constructing the ranked table: retain only m/z ions that did not appear in the blank or culture media (bed) samples. In NP3 MS Workflow ≥1.5.0 this is partly automated: [M+H]+ networks are built excluding m/z that appear in bed samples, so those ions are already removed from the protonated networks prior to correlation. For remaining unfiltered ions, cross-check each candidate's per-sample occurrence table against the blank and media sample groups declared in the Metadata Table and drop any ion present in either group. The filtered clean consensus spectra are then ranked by BCC (lowest BCC first for IC50-driven inverse correlation, e.g. m/z 679.272 at BCC = −0.90 for TMC-95 in MSV000091455) and written out as bcc_ranking_table.csv.

## Related tools

- **NP3 MS Workflow** (Produces the clean consensus spectra, per-sample occurrence information, and since v1.5.0 automatically excludes m/z appearing in bed samples from [M+H]+ network construction; the BCC filter for blank/media ions is applied on its step 9 output before step 10 (SSMN with damped pruning) and BCC ranking.) — https://github.com/danielatrivella/NP3_MS_Workflow
- **GNPS** (Downstream annotation of the surviving filtered ions via GNPS/Tremolo library search (e.g. TMC-95 match at score 0.75, m/z error 0.09 ppm) so that top-ranked filtered ions can be assigned structures.)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('clean_consensus_bcc.csv'); bed=set(pd.read_csv('bed_samples_mz.csv').mz); df=df[~df.mz.isin(bed)].sort_values('bcc'); df.to_csv('bcc_ranking_table.csv',index=False)"
```

## Evaluation signals

- For the MSV000091455 reproduction: after filtering, m/z 679.272 (rt 5.65 min, TMC-95) must be present and must be the top-ranked ion with the lowest BCC = −0.90.
- The closely spaced TMC-95 diastereomer variants m/z 679.2717 (BCC −0.56) and m/z 679.2732 (BCC −0.60) must survive filtering but rank below m/z 679.272, confirming discrimination was preserved.
- No ion present in the blank or bed sample group (as declared in the Metadata Table) appears anywhere in bcc_ranking_table.csv.
- In NP3 MS Workflow ≥1.5.0, the automatically built [M+H]+ networks contain zero nodes whose m/z occurs in bed samples — verify by cross-checking node list against the bed sample occurrence table.
- BCC sign/orientation is preserved: for IC50-derived groups the lowest (most negative) BCC corresponds to the most bioactive ion (inverse correlation), matching Table 1/Table 2 conventions.

## Limitations

- The filter depends on complete and correctly labeled blank and culture-media sample groups in the Metadata Table; mislabeled controls leak contaminants into the ranking.
- It does not separate genuine bioactive ions from co-eluting isomers or diastereomers that happen to be absent from media — BCC discrimination among variants (e.g. m/z 679.272 vs 679.2717) must be interpreted per variant.
- Since v1.5.0 the bed-sample exclusion in [M+H]+ networks is a default behavior, not a user-tunable parameter; other filter steps (K-most similar neighbors, minimum number of matched peaks in SSMN) are handled separately.
- Blank/media presence alone does not prove a contaminant — an ion present in media but genuinely bioactive would be incorrectly removed.

## Evidence

- [methods] BCC filtering against blank/culture media: "filtered to only keep the m/z ions that did not appear in the blank or culture media samples"
- [methods] ranking clean consensus spectra by BCC: "Then, the clean consensus spectra were ranked according to the BCCs (Table 1)"
- [methods] m/z 679.272 top-ranked TMC-95: "the m/z ion 679.272 was attributed with the lowest score (BCC = −0.90)"
- [readme] M+H networks exclude bed samples: "The [M+H]+ networks are now created without culture media by default, the m/z that appears in bed samples are removed from the protonated networks."
- [results] m/z 679.272 enriched in inhibitory fractions: "Extracted ion chromatograms for the ion at m/z 679.272 (rt 5.65 min) showed that it exhibited the highest intensities in the inhibitory fractions"

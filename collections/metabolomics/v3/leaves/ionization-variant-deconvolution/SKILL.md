---
name: ionization-variant-deconvolution
description: Use when processing LC–MS/MS untargeted metabolomics data in NP3 MS Workflow
  (step 7) where a single compound yields multiple m/z ions across ionization states,
  and protonated ([M+H]+) species must be identified and selected as representative
  nodes before bioactivity correlation scoring.
when_to_use_negative:
- Data is targeted metabolomics with known, pre-defined transitions rather than untargeted
  LC–MS/MS feature detection
- Feature table is already fully deconvolved (e.g. from an upstream isotope/adduct
  annotation tool) and no adduct/in-source-fragment clustering is needed
- The dataset lacks ionization variants of the same compound (single m/z per compound)
  — running IVAMN adds no information and risks splitting true single-peak features
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NP3 MS Workflow
  role: Pipeline that executes IVAMN deconvolution as step 7 of the automated run
    command, including PageRank [M+H]+ selection and noise filtering
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: GNPS
  role: Provides the community MS2 knowledge base and library annotations (e.g. CCMSLIB00000840365)
    consumed by the IVAMN's annotation enrichment before deconvolution
  repo: https://gnps2.org/workflows
- name: Tremolo
  role: Library search tool (step 6) that annotates clean consensus spectra in UNPD-ISDB
    before those spectra are deconvolved by IVAMN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:c9b142a2c6576a0f642c0f6bb341bdb7813c8de93fcedafac2211a37c94496a0
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ionization-variant-deconvolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deconvolve ionization variants of the same natural product in untargeted LC–MS/MS data using an ionization variant annotation molecular network (IVAMN) with PageRank-selected [M+H]+ representative ions, so that downstream bioactivity correlation is computed on the protonated species rather than on adduct/in-source artifacts.

## When to use

Use when processing LC–MS/MS untargeted metabolomics data in NP3 MS Workflow (step 7) where a single compound yields multiple m/z ions across ionization states, and protonated ([M+H]+) species must be identified and selected as representative nodes before bioactivity correlation scoring.

## When NOT to use

- Data is targeted metabolomics with known, pre-defined transitions rather than untargeted LC–MS/MS feature detection
- Feature table is already fully deconvolved (e.g. from an upstream isotope/adduct annotation tool) and no adduct/in-source-fragment clustering is needed
- The dataset lacks ionization variants of the same compound (single m/z per compound) — running IVAMN adds no information and risks splitting true single-peak features

## Inputs

- Clean consensus spectra table from NP3 MS Workflow steps 3–6 (MS2 spectra clustered, quantified, and identified)
- Metadata Table describing LC–MS/MS samples, sample groups, and bioactivity scores
- Per-sample MGF files produced by the preprocessing step
- Library annotation results from Tremolo/GNPS2 library search (with propagated annotations and IonMode field)

## Outputs

- IVAMN components (molecular network components) where protonated ion nodes have been selected by PageRank
- Representative [M+H]+ ion assignments per compound (e.g. m/z 679.272 for TMC-95 in the MA9 dataset)
- Deconvolved feature table used for BCC computation and SSMN construction
- Noise-filtered component index with sorted output (largest components first, edges sorted by source node ID)

## How to apply

After library identification (Tremolo against UNPD-ISDB) and GNPS annotation propagation, run the IVAMN deconvolution step (step 7) of the NP3 MS Workflow `run` command. The IVAMN clusters the clean consensus spectra into components where protonated ions ([M+H]+), adducts, and in-source fragments co-cluster; apply the noise cutoff and the [M+H]+ ion detection rules (fragmentation rules and annotation) to each component. Within each component, run the PageRank algorithm on the molecular network graph to select the representative protonated ion node; this [M+H]+ node is used downstream for quantification and bioactivity correlation. Set the noise_cutoff to an absolute value of minimum base peak intensity (without normalization) that an MS2 must exceed to survive clustering. In version 1.5.0+ the [M+H]+ networks are built without culture media by default and m/z present in bed/blank samples are removed before deconvolution.

## Related tools

- **NP3 MS Workflow** (Pipeline that executes IVAMN deconvolution as step 7 of the automated run command, including PageRank [M+H]+ selection and noise filtering) — https://github.com/danielatrivella/NP3_MS_Workflow
- **GNPS** (Provides the community MS2 knowledge base and library annotations (e.g. CCMSLIB00000840365) consumed by the IVAMN's annotation enrichment before deconvolution) — https://gnps2.org/workflows
- **Tremolo** (Library search tool (step 6) that annotates clean consensus spectra in UNPD-ISDB before those spectra are deconvolved by IVAMN)

## Examples

```
nextflow run np3.nf --run --input_metadata metadata.tsv --mzml_dir ./MA9 --ion_mode positive --noise_cutoff 1000
```

## Evaluation signals

- For a known positive control (e.g. TMC-95 in the A. moriforme MA9 dataset, MSV000091455), the selected representative ion must be the protonated m/z 679.272 (rt 5.65 min) with BCC = −0.90 and fragments m/z 661.261, 622.214, 594.219, 549.198, 295.107
- The TMC-86A positive-control in BRA-346 (MSV00091453) must yield the protonated m/z 343.187 with BCC 0.80, confirming PageRank [M+H]+ selection works reproducibly
- The component graph must be reproducible: IVAMN output ordering must place the largest components first and edges sorted by source node ID (version 1.4.2 invariant)
- No ion that appears in blank or culture media samples may be present in the protonated [M+H]+ networks (default behavior since v1.5.0)
- IonMode from the GNPS2 library annotation must be consistent with the ion mode used for deconvolution (positive vs negative MGF library)

## Limitations

- The noise_cutoff must be set as an absolute base peak intensity (not normalized) — misconfigured values can drop real minor MS2 features or retain noise
- When joining previous results (join_jobs command) the noise cutoff must reset the filtered IVAMN index to prevent matching errors in downstream filters
- Deconvolution quality depends on prior steps (clustering rounds, GNPS/Tremolo annotation quality); poor library coverage propagates into poor [M+H]+ selection
- SMILES retrieved from GNPS2 library search are validated during setup; unvalidated/empty SMILES reduce the enrichment quality feeding IVAMN

## Evidence

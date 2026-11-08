---
name: taqman-custom-array-probe-mapping
description: Use when ΔCt qPCR data from TaqMan Custom Arrays (Applied Biosystems)
  must be mapped to gene symbols and converted into per-gene significance calls against
  a baseline cell line such as RWPE-EV.
when_to_use_negative:
- Input is metabolomics data (mass spectrometry metabolite profiling or KEGG metabolite-set
  GSEA), not qPCR ΔCt measurements
- Expression values come from genome-wide profiling (e.g., RNA-seq or microarray)
  rather than the seven-gene TaqMan Custom Array panel with a defined probe list
- There is no engineered baseline control line (e.g., RWPE-EV) to serve as the reference
  for the one-sample T-Test
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Taqman Custom Arrays (Applied Biosystems)
  role: Source of the qPCR ΔCt measurements and the probe/primer list used to map
    each gene symbol to its assay
- name: GraphPad Prism 5.0
  role: Performs the one-sample T-Test per gene per engineered line versus the RWPE-EV
    baseline at p<0.05
- name: RNeasy Micro Kit (Qiagen)
  role: Isolates the total RNA that is the input material for the ΔCt qPCR expression
    data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:9b8b41f1c7f0cb72787a0e5efed2d15e83a93b77e3b523764f349dd41c637a7a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism 5.0
  - RNeasy Micro Kit
  - RNeasy Micro Kit (Qiagen)
  - Taqman Custom Arrays
  - Taqman Custom Arrays (Applied Biosystems)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# taqman-custom-array-probe-mapping

## Summary

Maps metabolic enzyme genes (e.g., GLUT-1, hexokinase 2, glutaminase, choline kinase alpha) to their TaqMan probe identifiers and compiles ΔCt qPCR results into a direction-of-change table across engineered prostate epithelial lines. Provides the transcriptional readout that links oncogenic AKT1/MYC activation to specific metabolic enzyme changes.

## When to use

Use when ΔCt qPCR data from TaqMan Custom Arrays (Applied Biosystems) must be mapped to gene symbols and converted into per-gene significance calls against a baseline cell line such as RWPE-EV.

## When NOT to use

- Input is metabolomics data (mass spectrometry metabolite profiling or KEGG metabolite-set GSEA), not qPCR ΔCt measurements
- Expression values come from genome-wide profiling (e.g., RNA-seq or microarray) rather than the seven-gene TaqMan Custom Array panel with a defined probe list
- There is no engineered baseline control line (e.g., RWPE-EV) to serve as the reference for the one-sample T-Test

## Inputs

- ΔCt qPCR expression table for the seven metabolic genes across RWPE-EV, RWPE-AKT1 and RWPE-MYC cells (total RNA isolated with RNeasy Micro Kit, Qiagen)
- TaqMan probe/primer mapping list from Text S1 (Taqman Custom Arrays, Applied Biosystems)
- Paper-reported direction-of-change statements per gene per line for validation

## Outputs

- Gene-by-cell-line table with gene symbol, mapped TaqMan probe, ΔCt-based direction of change (increased/decreased/unchanged), and one-sample T-Test p-value
- Per-gene significance calls at p<0.05 versus the RWPE-EV baseline

## How to apply

Load the ΔCt expression table for the target genes (here seven metabolic genes from RWPE-EV, RWPE-AKT1 and RWPE-MYC total RNA isolated with the RNeasy Micro Kit, Qiagen). Map each gene symbol to its corresponding TaqMan probe entry from the custom-array probe/primer list (documented in Text S1 of the source paper). For each gene in each engineered line, run a one-sample T-Test (GraphPad Prism 5.0 convention) against the RWPE-EV baseline with significance defined at p<0.05. Encode direction as increased/decreased/unchanged from the sign and significance of each test — e.g., GLUT-1 and hexokinase 2 increase with AKT1, glutaminase increases with MYC. Annotate each row with cell line, gene symbol, and the supporting result statement, then confirm the compiled table reproduces the paper's reported directions for all 7 genes across the 3 cell lines.

## Related tools

- **Taqman Custom Arrays (Applied Biosystems)** (Source of the qPCR ΔCt measurements and the probe/primer list used to map each gene symbol to its assay)
- **GraphPad Prism 5.0** (Performs the one-sample T-Test per gene per engineered line versus the RWPE-EV baseline at p<0.05)
- **RNeasy Micro Kit (Qiagen)** (Isolates the total RNA that is the input material for the ΔCt qPCR expression data)

## Evaluation signals

- Every gene symbol in the compiled table resolves to exactly one TaqMan probe in the Text S1 probe/primer list
- All 7 genes × 3 cell lines have a direction call (increased/decreased/unchanged), and each 'increased'/'decreased' call is backed by a one-sample T-Test p<0.05
- The compiled table reproduces the paper's reported directions — e.g., GLUT-1 and hexokinase 2 increased with AKT1; glutaminase increased with MYC; choline kinase alpha and cholinephosphotransferase-1 increased with MYC
- Each row carries its cell line label, gene symbol, and the verbatim supporting result statement from the paper

## Limitations

- Significance is defined only at p<0.05 under the GraphPad Prism 5.0 one-sample T-Test convention; genes near the threshold may flip direction with different test conventions
- Direction calls are valid only relative to the specific RWPE-EV baseline; they do not generalize to other control lines or culture conditions
- Coverage is limited to the seven metabolic genes on the custom TaqMan array — no genome-wide transcriptional inference is possible from this skill

## Evidence

- [methods] Map each gene to the corresponding Taqman probe (Taqman Custom Arrays, Applied Biosystems; probe/primer list in Text S1).: [evidence span withheld — anchor status: not_found]
- [methods] Load the RWPE-EV, RWPE-AKT1 and RWPE-MYC ΔCt expression data (total RNA isolated with RNeasy Micro Kit, Qiagen) for the seven metabolic genes.: [evidence span withheld — anchor status: not_found]
- [methods] Apply one-sample T-Test (GraphPad Prism 5.0 convention) per gene per engineered line versus the RWPE-EV baseline, with significance defined at p<0.05.: [evidence span withheld — anchor status: not_found]
- [results] glycolytic components such as the glucose transporter GLUT-1 and hexokinase 2 mRNA were increased upon AKT1 overexpression/activation in RWPE-1 cells.: [evidence span withheld — anchor status: not_found]
- [methods] confirm the compiled table reproduces the paper's reported directions for all 7 genes across the 3 cell lines: [evidence span withheld — anchor status: not_found]

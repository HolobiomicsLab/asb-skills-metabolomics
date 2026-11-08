---
name: one-sample-ttest-significance-testing
description: Use when you have ΔCt qPCR expression values (e.g. Taqman Custom Arrays)
  for a fixed gene panel across an engineered cell line and its parental baseline
  (e.g. RWPE-AKT1 or RWPE-MYC vs RWPE-EV) and need significance-backed direction-of-change
  calls.
when_to_use_negative:
- When the comparison is between two independent experimental groups rather than one
  engineered line against a fixed baseline (e.g. RWPE-EV) — a two-sample design is
  needed instead
- When the input is metabolomics pathway-enrichment data (over 50 KEGG metabolite
  sets tested by single-sample GSEA), which this paper analyzed with GSEA rather than
  per-gene T-Tests
- When raw Ct values without ΔCt normalization are supplied, since the one-sample
  T-Test is defined here on the ΔCt scale against the EV baseline
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: GraphPad Prism 5.0
  role: Runs the one-sample T-Tests per gene per engineered line versus the RWPE-EV
    baseline, defining significance at p<0.05
- name: Taqman Custom Arrays (Applied Biosystems)
  role: Generates the ΔCt expression measurements for the seven metabolic genes via
    gene-specific probe/primer assays (Text S1)
- name: RNeasy Micro Kit (Qiagen)
  role: Isolates the total RNA that is the source material for the ΔCt qPCR input
    data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:d5432e3883a5c560d7840ca03cb3633a4d185fb6236718f5da2512d141ac7990
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism 5.0
  - RNeasy Micro Kit
  - RNeasy Micro Kit (Qiagen)
  - Taqman Custom Arrays (Applied Biosystems)
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# one-sample-ttest-significance-testing

## Summary

A significance-testing skill that applies a one-sample T-Test (GraphPad Prism 5.0 convention) per target gene per engineered cell line against the RWPE-EV baseline, encoding each result as increased/decreased/unchanged at p<0.05. It converts ΔCt qPCR measurements into a reproducible direction-of-change table with statistical support for each claim.

## When to use

Use when you have ΔCt qPCR expression values (e.g. Taqman Custom Arrays) for a fixed gene panel across an engineered cell line and its parental baseline (e.g. RWPE-AKT1 or RWPE-MYC vs RWPE-EV) and need significance-backed direction-of-change calls.

## When NOT to use

- When the comparison is between two independent experimental groups rather than one engineered line against a fixed baseline (e.g. RWPE-EV) — a two-sample design is needed instead
- When the input is metabolomics pathway-enrichment data (over 50 KEGG metabolite sets tested by single-sample GSEA), which this paper analyzed with GSEA rather than per-gene T-Tests
- When raw Ct values without ΔCt normalization are supplied, since the one-sample T-Test is defined here on the ΔCt scale against the EV baseline

## Inputs

- ΔCt expression table for seven metabolic genes across RWPE-EV, RWPE-AKT1, and RWPE-MYC cells (Taqman Custom Arrays, Applied Biosystems; total RNA isolated with RNeasy Micro Kit, Qiagen)
- Gene-to-Taqman-probe mapping (probe/primer list in Text S1)
- RWPE-EV baseline ΔCt values used as the reference mean for each one-sample test

## Outputs

- Gene × cell line table with direction of change (increased/decreased/unchanged) and one-sample T-Test p-values at the p<0.05 threshold
- Per-gene annotation tying each direction call to the supporting result statement from the paper

## How to apply

Load the ΔCt expression data for each metabolic gene (e.g. GLUT-1, hexokinase 2, glutaminase, choline kinase alpha, cholinephosphotransferase-1) from total RNA isolated with the RNeasy Micro Kit and assayed on Taqman Custom Arrays. Map each gene to its Taqman probe/primer entry (probe list in Text S1) so every measurement is traceable to a specific assay. For each gene in each engineered line (RWPE-AKT1, RWPE-MYC), run a one-sample T-Test using GraphPad Prism 5.0 convention against the RWPE-EV baseline, with significance defined at p<0.05. Encode direction of change as increased/decreased/unchanged from the sign and significance of each test — for example, GLUT-1 and hexokinase 2 increased with AKT1, and glutaminase increased with MYC. Annotate each row with cell line, gene symbol, and the supporting result statement (e.g. 'RWPE-MYC cells showed increased expression of two key enzymes of the glycerophospholipid metabolism, choline kinase alpha and cholinephosphotransferase-1').

## Related tools

- **GraphPad Prism 5.0** (Runs the one-sample T-Tests per gene per engineered line versus the RWPE-EV baseline, defining significance at p<0.05)
- **Taqman Custom Arrays (Applied Biosystems)** (Generates the ΔCt expression measurements for the seven metabolic genes via gene-specific probe/primer assays (Text S1))
- **RNeasy Micro Kit (Qiagen)** (Isolates the total RNA that is the source material for the ΔCt qPCR input data)

## Evaluation signals

- Every claim of increased expression is supported by a one-sample T-Test with p<0.05
- Compiled table reproduces the paper's reported directions for all 7 genes across all 3 cell lines (RWPE-EV, RWPE-AKT1, RWPE-MYC)
- Direction encoding (increased/decreased/unchanged) is consistent with the sign and significance of each per-gene test
- Each row carries cell line, gene symbol, and the paper's supporting result statement
- Known directions are recovered: GLUT-1 and hexokinase 2 increased with AKT1; glutaminase increased with MYC; choline kinase alpha and cholinephosphotransferase-1 increased with MYC

## Limitations

- Significance is defined solely at p<0.05 under the GraphPad Prism 5.0 one-sample T-Test convention; no multiplicity correction across the 7 genes is described
- Direction calls depend on the RWPE-EV baseline as the fixed reference; results are only meaningful relative to that baseline
- The skill covers transcriptional changes in 7 metabolic genes only; metabolite-level effects (e.g. aerobic glycolysis metabolites with AKT1, lipid metabolism with MYC) were addressed by separate mass spectrometry and GSEA analyses

## Evidence

- [methods] Apply one-sample T-Test (GraphPad Prism 5.0 convention) per gene per engineered line versus the RWPE-EV baseline, with significance defined at p<0.05.: [evidence span withheld — anchor status: not_found]
- [methods] Load the RWPE-EV, RWPE-AKT1 and RWPE-MYC ΔCt expression data (total RNA isolated with RNeasy Micro Kit, Qiagen) for the seven metabolic genes.: [evidence span withheld — anchor status: not_found]
- [methods] confirm the compiled table reproduces the paper's reported directions for all 7 genes across the 3 cell lines: [evidence span withheld — anchor status: not_found]
- [results] glycolytic components such as the glucose transporter GLUT-1 and hexokinase 2 mRNA were increased upon AKT1 overexpression/activation in RWPE-1 cells.: [evidence span withheld — anchor status: not_found]

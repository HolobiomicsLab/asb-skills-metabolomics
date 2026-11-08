---
name: metabolic-gene-mrna-expression-profiling
description: Use when ΔCt qPCR expression data (e.g. Taqman Custom Array raw values)
  for a fixed panel of metabolic genes must be converted into a gene × cell-line table
  of statistically supported direction-of-change calls versus a baseline control line.
when_to_use_negative:
- Input is untargeted mass-spectrometry metabolomics data (use GSEA over KEGG metabolite
  sets instead — this skill covers the mRNA/qPCR arm only)
- Expression data lack a common baseline control line (e.g. RWPE-EV), making one-sample
  T-Tests versus baseline undefined
- The goal is protein-level validation of AKT1/MYC status by immunoblotting rather
  than metabolic-enzyme mRNA quantitation
tools:
- name: GraphPad Prism 5.0
  role: Performs the one-sample T-Tests per gene per line versus the RWPE-EV baseline,
    with significance at p<0.05
- name: Taqman Custom Arrays (Applied Biosystems)
  role: qPCR platform providing ΔCt measurements for the 7 metabolic genes using the
    probe/primer set in Text S1
- name: RNeasy Micro Kit (Qiagen)
  role: Total RNA isolation from RWPE-1 engineered lines prior to Taqman qPCR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:953bf44d4417a51fae2a9a6865d01c461c814f4f13f62d7cce0e2985c2ce3871
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism 5.0
  - RNeasy Micro Kit
  - RNeasy Micro Kit (Qiagen)
  - Taqman Custom Arrays (Applied Biosystems)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolic-gene-mrna-expression-profiling

## Summary

Compiles a per-gene qPCR mRNA expression table for metabolic enzymes (e.g. GLUT-1, hexokinase 2, glutaminase, choline kinase alpha) across engineered prostate epithelial lines, testing each gene against an empty-vector baseline with one-sample T-Tests to encode increased/decreased/unchanged direction of change.

## When to use

Use when ΔCt qPCR expression data (e.g. Taqman Custom Array raw values) for a fixed panel of metabolic genes must be converted into a gene × cell-line table of statistically supported direction-of-change calls versus a baseline control line.

## When NOT to use

- Input is untargeted mass-spectrometry metabolomics data (use GSEA over KEGG metabolite sets instead — this skill covers the mRNA/qPCR arm only)
- Expression data lack a common baseline control line (e.g. RWPE-EV), making one-sample T-Tests versus baseline undefined
- The goal is protein-level validation of AKT1/MYC status by immunoblotting rather than metabolic-enzyme mRNA quantitation

## Inputs

- ΔCt qPCR expression values for metabolic genes across engineered cell lines (RWPE-EV, RWPE-AKT1, RWPE-MYC), total RNA isolated with RNeasy Micro Kit
- Taqman probe/primer mapping for the 7-gene metabolic panel (Text S1)
- Cell-line annotation (engineered construct per sample)

## Outputs

- Gene × cell-line expression table with direction-of-change calls (increased/decreased/unchanged) per metabolic gene
- Per-gene one-sample T-Test p-values versus the RWPE-EV baseline (significance threshold p<0.05)
- Annotated result statements linking each significant direction call to the paper's reported finding (e.g. glycolytic GLUT-1 and hexokinase 2 mRNA increased with AKT1; glutaminase increased with MYC)

## How to apply

Load the ΔCt expression data for each engineered line (e.g. RWPE-EV, RWPE-AKT1, RWPE-MYC, total RNA isolated with the RNeasy Micro Kit) for the target gene panel. Map each gene symbol to its Taqman probe/primer from the supplementary probe list (Text S1). Apply a one-sample T-Test per gene per engineered line versus the RWPE-EV baseline using GraphPad Prism 5.0 conventions, defining significance at p<0.05. Encode each cell as increased, decreased, or unchanged based on the sign and significance of the test, then annotate each row with the cell line, gene symbol, and the supporting result statement from the paper (e.g. increased GLUT-1 and hexokinase 2 mRNA with AKT1; increased glutaminase with MYC; increased choline kinase alpha and cholinephosphotransferase-1 with MYC). Validate that the table reproduces the paper's reported directions for all 7 genes across the 3 lines and that every 'increased' claim is backed by p<0.05.

## Related tools

- **GraphPad Prism 5.0** (Performs the one-sample T-Tests per gene per line versus the RWPE-EV baseline, with significance at p<0.05)
- **Taqman Custom Arrays (Applied Biosystems)** (qPCR platform providing ΔCt measurements for the 7 metabolic genes using the probe/primer set in Text S1)
- **RNeasy Micro Kit (Qiagen)** (Total RNA isolation from RWPE-1 engineered lines prior to Taqman qPCR)

## Evaluation signals

- All 7 metabolic genes are present for all 3 cell lines (21 cells in the direction-of-change table)
- Every cell labeled 'increased' or 'decreased' carries a one-sample T-Test p-value < 0.05; 'unchanged' cells have non-significant p-values
- Direction calls reproduce the paper's reported signs: GLUT-1 and hexokinase 2 increased with AKT1; glutaminase increased with MYC; choline kinase alpha and cholinephosphotransferase-1 increased with MYC
- Each gene row maps to a Taqman probe from the Text S1 probe/primer list
- Each row is annotated with the cell line, gene symbol, and the paper's supporting result statement

## Limitations

- Direction calls depend on the RWPE-EV baseline; conclusions are specific to RWPE-1-derived engineered lines and may not transfer to other cell systems
- The panel is limited to 7 metabolic genes measured by custom Taqman probes, so it captures only targeted enzyme-level transcriptional changes, not genome-wide expression
- Significance uses the paper's p<0.05 threshold on one-sample T-Tests; no multiple-testing correction is described, so marginal calls should be interpreted cautiously

## Evidence

- [other] Glycolytic components such as the glucose transporter GLUT-1 and hexokinase 2 mRNA were increased upon AKT1 overexpression/activation in RWPE-1 cells.: [evidence span withheld — anchor status: not_found]

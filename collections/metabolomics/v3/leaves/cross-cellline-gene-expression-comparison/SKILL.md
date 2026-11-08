---
name: cross-cellline-gene-expression-comparison
description: Use when comparing transcript levels of candidate metabolic genes (e.g.
  GLUT-1, hexokinase 2, glutaminase, choline kinase alpha) between engineered isogenic
  cell lines and their common parental control to test whether an oncogene drives
  enzyme-expression changes.
when_to_use_negative:
- Input is already an enrichment or pathway-level metabolite set (e.g. KEGG single-sample
  GSEA scores) — those describe pathway direction rather than individual gene ΔCt
  changes and require a different statistical treatment.
- The comparison is across patient tumor specimens rather than engineered cell lines
  — in that case use the phosphoAKT1/MYC k-means subgrouping described in the paper
  instead of a ΔCt-vs-baseline design.
edam_operation: http://edamontology.org/operation_0315
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3523
tools:
- name: GraphPad Prism 5.0
  role: Runs the one-sample T-Test of each gene's ΔCt against the RWPE-EV baseline
    and reports the p-value used to call increase/decrease
- name: Taqman Custom Arrays (Applied Biosystems)
  role: qPCR platform carrying the gene-specific probes/primers listed in Text S1
    that generate the Ct values
- name: RNeasy Micro Kit (Qiagen)
  role: Isolates total RNA from each RWPE-1 derivative for qPCR input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:f4cc9473e13345f1cbf23ebc70309555050ce0a389f791db38b8ddac0b6121d4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism 5.0
  - RNeasy Micro Kit
  - RNeasy Micro Kit (Qiagen)
  - Taqman Custom Arrays (Applied Biosystems)
  edam_operation: http://edamontology.org/operation_0315
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-cellline-gene-expression-comparison

## Summary

Compares mRNA expression of metabolic target genes across isogenic prostate epithelial cell lines (non-transformed RWPE-1 controls vs AKT1- or MYC-transformed derivatives) to attribute metabolome changes to specific oncogenic transcriptional programs.

## When to use

Use when comparing transcript levels of candidate metabolic genes (e.g. GLUT-1, hexokinase 2, glutaminase, choline kinase alpha) between engineered isogenic cell lines and their common parental control to test whether an oncogene drives enzyme-expression changes.

## When NOT to use

- Input is already an enrichment or pathway-level metabolite set (e.g. KEGG single-sample GSEA scores) — those describe pathway direction rather than individual gene ΔCt changes and require a different statistical treatment.
- The comparison is across patient tumor specimens rather than engineered cell lines — in that case use the phosphoAKT1/MYC k-means subgrouping described in the paper instead of a ΔCt-vs-baseline design.

## Inputs

- Total RNA from RWPE-1 derivatives (RWPE-EV, RWPE-AKT1, RWPE-MYC), isolated with the RNeasy Micro Kit
- Raw Taqman Ct values for the seven metabolic genes, with gene-to-probe mapping from the Text S1 probe/primer list
- A one-sample T-Test configuration per gene/line pair against the RWPE-EV baseline, threshold p<0.05

## Outputs

- A per-gene × per-cell-line table of direction-of-change calls (increased/decreased/unchanged) with ΔCt values and one-sample T-Test p-values
- Annotated rows linking each call to its supporting statement (e.g. GLUT-1 and hexokinase 2 increased with AKT1; glutaminase increased with MYC)

## How to apply

Isolate total RNA from each engineered RWPE-1 derivative and the empty-vector (RWPE-EV) baseline with the RNeasy Micro Kit (Qiagen). Run each metabolic-target gene on Taqman Custom Arrays (Applied Biosystems), mapping each gene to its published probe/primer pair (listed in Text S1), and record raw Ct values. Convert to ΔCt relative to housekeeping controls, then for each gene in each engineered line (RWPE-AKT1, RWPE-MYC) apply a one-sample T-Test of the ΔCt against the RWPE-EV baseline (GraphPad Prism 5.0 convention) with significance at p<0.05. Encode direction as increased/decreased/unchanged from the sign and significance of each test, then annotate each row with the cell line, gene symbol, and a supporting result statement — e.g. 'RWPE-MYC cells showed increased expression of two key enzymes of the glycerophospholipid metabolism, choline kinase alpha and cholinephosphotransferase-1'. Validate that the compiled table reproduces the paper's reported direction for all seven metabolic genes across all three lines, with every 'increased' call backed by a p<0.05 one-sample T-Test.

## Related tools

- **GraphPad Prism 5.0** (Runs the one-sample T-Test of each gene's ΔCt against the RWPE-EV baseline and reports the p-value used to call increase/decrease)
- **Taqman Custom Arrays (Applied Biosystems)** (qPCR platform carrying the gene-specific probes/primers listed in Text S1 that generate the Ct values)
- **RNeasy Micro Kit (Qiagen)** (Isolates total RNA from each RWPE-1 derivative for qPCR input)

## Examples

```
# ΔCt one-sample T-Test per gene/line vs RWPE-EV baseline (GraphPad Prism 5.0 convention), threshold p<0.05
t_test(akt1_dct, popmean=ev_dct_mean, alternative='two-sided')
```

## Evaluation signals

- The compiled table must reproduce the paper's reported direction for all 7 metabolic genes across all 3 cell lines.
- Every row labeled 'increased' must trace to a one-sample T-Test p<0.05 against the RWPE-EV baseline.
- Each table row must carry a concrete supporting result statement from the paper, not a generic annotation.
- Gene-to-probe mapping must resolve every gene to a Taqman probe listed in Text S1 — no unmapped genes in the output.

## Limitations

- The design is specific to the RWPE-1 isogenic system (EV/AKT1/MYC) and the seven metabolic genes with Taqman probes in Text S1; extension to other gene panels or cell backgrounds would need re-validation.
- Significance is defined by one-sample T-Test p<0.05 only; no multiple-testing correction is reported, so direction calls near the threshold are fragile.
- Reported directions reflect mRNA change only; the paper couples these calls to metabolite-level findings (e.g. GLUT-1/hexokinase 2 mRNA increase alongside AKT1-driven glycolysis metabolite accumulation) rather than validating protein or flux changes.

## Evidence

- [other] Load the RWPE-EV, RWPE-AKT1 and RWPE-MYC ΔCt expression data (total RNA isolated with RNeasy Micro Kit, Qiagen) for the seven metabolic genes.: [evidence span withheld — anchor status: not_found]
- [other] Apply one-sample T-Test (GraphPad Prism 5.0 convention) per gene per engineered line versus the RWPE-EV baseline, with significance defined at p<0.05.: [evidence span withheld — anchor status: not_found]
- [other] GLUT-1 and hexokinase 2 mRNA were increased upon AKT1 overexpression/activation in RWPE-1 cells: [evidence span withheld — anchor status: not_found]
- [other] RWPE-MYC cells showed increased expression of two key enzymes of the glycerophospholipid metabolism, choline kinase alpha and cholinephosphotransferase-1: "RWPE-MYC cells showed increased expression of two key enzymes of the glycerophospholipid metabolism, choline kinase alpha and cholinephosphotransferase-1"
- [results] Yes — glycolytic components such as the glucose transporter GLUT-1 and hexokinase 2 mRNA were increased upon AKT1 overexpression/activation in RWPE-1 cells.: [evidence span withheld — anchor status: not_found]
- [methods] RWPE-EV, RWPE-AKT1 and RWPE-MYC cell lines are compared on Taqman Custom Arrays for seven metabolic genes: [evidence span withheld — anchor status: not_found]

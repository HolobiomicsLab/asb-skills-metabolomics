---
name: direction-of-change-annotation
description: Use when ΔCt qPCR data for metabolic enzyme genes are collected across
  oncogene-transformed versus control epithelial cells and each gene's direction of
  change must be tabulated with statistical support.
when_to_use_negative:
- Data are mass-spectrometry metabolite intensities rather than qPCR ΔCt values —
  metabolite direction calls need metabolite-level statistics, not Taqman probe annotation
- Comparisons are not against a common control baseline (e.g. two transformed lines
  compared directly without a RWPE-EV anchor) — one-sample T-Test against the baseline
  would be misapplied
- Tumour-level phosphoAKT1/MYC immunoblot densitometry is the input, since those data
  are clustered by K-means rather than tested per-gene direction of change
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: GraphPad Prism 5.0
  role: Performs the one-sample T-Test of ΔCt per gene per engineered line against
    the RWPE-EV baseline, establishing the p<0.05 significance that drives the direction-of-change
    call.
- name: Taqman Custom Arrays (Applied Biosystems)
  role: Generates the ΔCt expression measurements for the metabolic genes using the
    specific probe/primer pairs listed in Text S1.
- name: RNeasy Micro Kit (Qiagen)
  role: Isolates total RNA that serves as the upstream substrate for the Taqman qPCR
    reactions feeding this skill.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:dcf7c5a2f5db578e428cd00de3ca5456c1f1b1b9e13e018fe14c6b2986ee1757
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism 5.0
  - RNeasy Micro Kit
  - RNeasy Micro Kit (Qiagen)
  - Taqman Custom Arrays (Applied Biosystems)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# direction-of-change-annotation

## Summary

Annotates qPCR-derived expression of metabolic target genes (e.g. GLUT-1, hexokinase 2, glutaminase, choline kinase alpha) as increased / decreased / unchanged per engineered prostate cell line, using one-sample T-Tests against the RWPE-EV baseline. Encodes the transcriptional counterpart of AKT1/MYC-driven metabolome shifts reported by mass-spectrometry metabolomics.

## When to use

Use when ΔCt qPCR data for metabolic enzyme genes are collected across oncogene-transformed versus control epithelial cells and each gene's direction of change must be tabulated with statistical support.

## When NOT to use

- Data are mass-spectrometry metabolite intensities rather than qPCR ΔCt values — metabolite direction calls need metabolite-level statistics, not Taqman probe annotation
- Comparisons are not against a common control baseline (e.g. two transformed lines compared directly without a RWPE-EV anchor) — one-sample T-Test against the baseline would be misapplied
- Tumour-level phosphoAKT1/MYC immunoblot densitometry is the input, since those data are clustered by K-means rather than tested per-gene direction of change

## Inputs

- ΔCt qPCR values per gene per cell line (RWPE-EV, RWPE-AKT1, RWPE-MYC) obtained from Taqman Custom Arrays
- Taqman probe/primer assignment table for the 7 metabolic target genes (Text S1)
- Total RNA isolated with RNeasy Micro Kit (Qiagen) from each engineered cell line

## Outputs

- Direction-of-change table: rows = metabolic genes, columns = engineered cell lines, cells = increased / decreased / unchanged
- Per-gene-per-line p-value from one-sample T-Test (GraphPad Prism 5.0 convention, significance p<0.05)
- Annotated result statement per row citing the paper's finding (e.g. increased choline kinase alpha and cholinephosphotransferase-1 in RWPE-MYC cells)

## How to apply

1) Collect ΔCt values for each metabolic gene from Taqman Custom Arrays run on total RNA isolated with the RNeasy Micro Kit (Qiagen). 2) Map each gene to its specific Taqman probe/primer pair (list provided in Text S1 of the paper) so that probe identity is tracked alongside the gene symbol. 3) For each engineered cell line (RWPE-AKT1, RWPE-MYC) compare ΔCt against the RWPE-EV baseline using a one-sample T-Test in GraphPad Prism 5.0, defining significance at p<0.05. 4) Encode direction of change as increased / decreased / unchanged based on the sign and significance of each test — for example GLUT-1 and hexokinase 2 were increased in RWPE-AKT1 cells and glutaminase was increased in RWPE-MYC cells. 5) Annotate each row with the cell line, gene symbol and the supporting result statement from the paper (e.g. 'RWPE-MYC cells showed increased expression of two key enzymes of the glycerophospholipid metabolism, choline kinase alpha and cholinephosphotransferase-1'). 6) Validate the compiled table against the paper's reported directions for all 7 genes across the 3 cell lines, confirming that any claim of increase is backed by a one-sample T-Test p<0.05. The rationale is to translate metabolome changes seen by mass-spectrometry metabolomics into their transcriptional counterparts at key metabolic enzymes.

## Related tools

- **GraphPad Prism 5.0** (Performs the one-sample T-Test of ΔCt per gene per engineered line against the RWPE-EV baseline, establishing the p<0.05 significance that drives the direction-of-change call.)
- **Taqman Custom Arrays (Applied Biosystems)** (Generates the ΔCt expression measurements for the metabolic genes using the specific probe/primer pairs listed in Text S1.)
- **RNeasy Micro Kit (Qiagen)** (Isolates total RNA that serves as the upstream substrate for the Taqman qPCR reactions feeding this skill.)

## Examples

```
GraphPad Prism 5.0 one-sample T-Test on ΔCt values per gene per line (RWPE-AKT1 and RWPE-MYC vs RWPE-EV baseline), significance threshold p<0.05, followed by direction-of-change annotation as increased / decreased / unchanged.
```

## Evaluation signals

- The compiled table reproduces the paper's reported direction for all 7 genes across the 3 cell lines (RWPE-EV, RWPE-AKT1, RWPE-MYC).
- Every cell labelled increased or decreased is backed by a one-sample T-Test result with p<0.05.
- Each row carries the correct Taqman probe identity traced through Text S1.
- Direction labels are consistent with the paper's explicit result statements (e.g. glutaminase increased in RWPE-MYC cells).

## Limitations

- Only 7 metabolic genes are covered by the Taqman panel — the table does not represent genome-wide transcription.
- Statistical support is limited to one-sample T-Test p<0.05; no correction for multiple testing is applied.
- The table reflects the paper's specific cell lines (RWPE-1 derived) and may not generalise to other epithelial transformation models.

## Evidence

- [other] Ta tman Custom Arrays and probe/primer list: [evidence span withheld — anchor status: not_found]
- [other] One-sample T-Test and direction encoding: [evidence span withheld — anchor status: not_found]
- [other] Example of direction call and paper support: [evidence span withheld — anchor status: not_found]
- [other] Validation rule: [evidence span withheld — anchor status: not_found]
- [other] Biological rationale linking metabolome to transcriptome: [evidence span withheld — anchor status: not_found]

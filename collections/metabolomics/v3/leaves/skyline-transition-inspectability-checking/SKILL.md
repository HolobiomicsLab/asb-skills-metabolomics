---
name: skyline-transition-inspectability-checking
description: Use when curating a targeted peptide precursor panel from DIA-NN-derived
  quantification of retrospective DIA files, and only precursors whose transitions
  can be manually peak-shape-inspected in Skyline should be kept.
when_to_use_negative:
- When the DIA files have not yet been processed by DIA-NN with 1% FDR filtration
  — there is no quantified precursor list to inspect in Skyline
- When the goal is bulk LC-MS file quality assessment rather than per-transition curation
  — iDIA-QC's 15-metric ML pipeline is the appropriate tool for that task
- When candidate peptides have not yet passed the upstream inclusion criteria (consistency,
  length, missed cleavages, reactive-residue/tryptic-inhibitory exclusion), since
  Skyline inspection is the final filtering step, not a substitute for it
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Skyline
  role: Manual inspection of transition peak shapes to decide which peptide precursors
    are retained in the high-quality panel
- name: DIA-NN
  role: Analyzes the DIA files (v1.7.15) against the spectral library to derive the
    peptide quantification that supplies candidate precursors for Skyline inspection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:31108084da3aa5d801bd6daa010d33882ab5377ae769c1dd52dc15354bb25ce1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/guomics-lab/iDIA-QC
  tools:
  - DIA-NN
  - Skyline
  - iDIA-QC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# skyline-transition-inspectability-checking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A final manual curation step that uses Skyline to inspect transition-level peak shapes of peptide precursors quantified by DIA-NN from retrospective DIA runs, retaining only precursors whose transitions are inspectable and of acceptable quality. In the iDIA-QC study this step narrowed the inclusion-criteria-filtered mouse-liver-digest panel to 934 high-quality precursors.

## When to use

Use when curating a targeted peptide precursor panel from DIA-NN-derived quantification of retrospective DIA files, and only precursors whose transitions can be manually peak-shape-inspected in Skyline should be kept.

## When NOT to use

- When the DIA files have not yet been processed by DIA-NN with 1% FDR filtration — there is no quantified precursor list to inspect in Skyline
- When the goal is bulk LC-MS file quality assessment rather than per-transition curation — iDIA-QC's 15-metric ML pipeline is the appropriate tool for that task
- When candidate peptides have not yet passed the upstream inclusion criteria (consistency, length, missed cleavages, reactive-residue/tryptic-inhibitory exclusion), since Skyline inspection is the final filtering step, not a substitute for it

## Inputs

- DIA-NN (v1.7.15) peptide quantification table from retrospective mouse-liver-digest DIA files (e.g., the 221-file subset) at 1% FDR at peptide precursor and protein levels
- Published mouse spectral library containing 110,126 precursors and 10,542 protein groups
- Precursor list already passed through inclusion criteria: cross-file consistency, 7-15 aa length window, no missed cleavages, no reactive-residue or tryptic-inhibitory peptides
- Skyline transition lists for the candidate precursors

## Outputs

- Curated panel of Skyline-inspectable peptide precursors (peptide_precursor_panel.tsv with exactly 934 rows)
- Physicochemical profile of the panel: GRAVY scores in [-2.5, 2.3] and a length distribution with >=90% of peptides 7-15 aa
- Per-precursor Skyline transition peak-shape inspection verdicts (retain/reject)

## How to apply

First analyze the retrospective mouse-liver-digest DIA raw files with DIA-NN (v1.7.15) against the published mouse spectral library (110,126 precursors, 10,542 protein groups), applying 1% FDR at both peptide precursor and protein levels to derive peptide quantification. Then apply the paper's peptide inclusion criteria — consistency across all files, sequence-length constraints, no missed cleavages, and exclusion of reactive-residue and tryptic-inhibitory peptides — before reaching the Skyline stage. For each surviving precursor, build/import its transitions in Skyline and manually inspect transition peak shapes, retaining only precursors with Skyline-inspectable, acceptable transitions; this is the decisive curation step because automated QC metrics alone cannot guarantee per-transition peak-shape quality. Finally compute physicochemical properties (GRAVY hydropathy, sequence-length distribution) of the retained set and verify the panel matches the expected profile: exactly 934 precursors, GRAVY in [-2.5, 2.3], and >=90% of peptides 7-15 amino acids long. Reject the exported panel (peptide_precursor_panel.tsv) if any of these criteria fail.

## Related tools

- **Skyline** (Manual inspection of transition peak shapes to decide which peptide precursors are retained in the high-quality panel)
- **DIA-NN** (Analyzes the DIA files (v1.7.15) against the spectral library to derive the peptide quantification that supplies candidate precursors for Skyline inspection)

## Evaluation signals

- Exported peptide_precursor_panel.tsv contains exactly 934 precursor rows
- All GRAVY values in the panel fall within [-2.5, 2.3]
- At least 90% of retained peptides are 7-15 amino acids long
- Every retained precursor passed the upstream inclusion criteria (cross-file consistency, no missed cleavages, no reactive-residue/tryptic-inhibitory peptides) before Skyline inspection
- Every retained precursor has transitions that were manually inspected for peak shape in Skyline; precursors without inspectable transitions must be absent from the panel

## Limitations

- The check is manual and expert-dependent; per-transition peak-shape judgments are not automated, limiting throughput and reproducibility
- Applies to targeted-style curation of DIA data with a spectral library; SRM/targeted QC tooling cannot be directly applied to large-scale quantitative proteomics experiments concerning thousands of peptide precursors
- Only transitions with inspectable peak shapes are retained, so the panel is bounded by what Skyline can visualize, potentially excluding otherwise-qualified precursors

## Evidence

- [other] retention of only Skyline-inspectable transitions: [evidence span withheld — anchor status: not_found]
- [other] the panel was narrowed to 934 high-quality peptide precursors: [evidence span withheld — anchor status: not_found]

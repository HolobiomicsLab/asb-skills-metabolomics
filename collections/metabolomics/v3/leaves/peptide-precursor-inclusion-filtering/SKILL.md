---
name: peptide-precursor-inclusion-filtering
description: Use when a large DIA-quantified precursor list (e.g., from a spectral
  library like 110,126 precursors) must be filtered down to a small, manually verifiable
  panel of high-quality peptide precursors for targeted assay development.
when_to_use_negative:
- When the goal is QC assessment of the mass spectrometer itself rather than curation
  of a peptide assay panel (iDIA-QC's 15 metrics target instrument LC/MS quality,
  not precursor selection)
- When input data are DDA runs — the criteria assume DIA-NN quantification of DIA
  files against a library-built precursor list
- 'When a discovery-level precursor list is desired: the criteria deliberately discard
  inconsistent, long, missed-cleaved, or chemically problematic peptides'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: DIA-NN
  role: Analyzes the DIA raw files (v1.7.15) against the spectral library with 1%
    FDR at precursor and protein levels to produce the peptide quantification that
    the inclusion criteria are applied to
- name: Skyline
  role: Used for manual inspection of transition peak shapes of candidate precursors;
    only Skyline-inspectable precursors with acceptable peak shapes are retained
- name: iDIA-QC
  role: Companion QC software built on DIA-NN algorithms for evaluating the quality
    of the DIA files feeding the analysis
  repo: github.com/guomics-lab/iDIA-QC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:12ad91e0083f129b51f2f9d6c537599e575734ff972562b34801646e31965973
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# peptide-precursor-inclusion-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Narrows DIA-NN-derived peptide quantification from retrospective mouse liver digest DIA files to a high-quality targeted panel by applying inclusion criteria (cross-file consistency, sequence-length constraints, no missed cleavages, exclusion of reactive-residue/tryptic-inhibitory peptides, and Skyline transition inspection), yielding a curated precursor set with characterized physicochemical properties.

## When to use

Use when a large DIA-quantified precursor list (e.g., from a spectral library like 110,126 precursors) must be filtered down to a small, manually verifiable panel of high-quality peptide precursors for targeted assay development.

## When NOT to use

- When the goal is QC assessment of the mass spectrometer itself rather than curation of a peptide assay panel (iDIA-QC's 15 metrics target instrument LC/MS quality, not precursor selection)
- When input data are DDA runs — the criteria assume DIA-NN quantification of DIA files against a library-built precursor list
- When a discovery-level precursor list is desired: the criteria deliberately discard inconsistent, long, missed-cleaved, or chemically problematic peptides

## Inputs

- Peptide quantification table from DIA-NN analysis of DIA raw files (e.g., 221 retrospective mouse liver digest DIA runs)
- Published mouse spectral library (110,126 precursors, 10,542 protein groups)
- Peptide sequences of candidate precursors (for length, missed-cleavage, reactive-residue, and GRAVY checks)
- Skyline transition lists for manual peak-shape inspection

## Outputs

- Filtered peptide precursor panel (e.g., peptide_precursor_panel.tsv with 934 precursor rows)
- Physicochemical characterization: GRAVY score distribution (−2.5 to 2.3) and sequence-length distribution (≥90% of peptides 7–15 aa)

## How to apply

First quantify peptides from the DIA raw files (e.g., 221 retrospective mouse liver digest files) using DIA-NN v1.7.15 against the published mouse spectral library, applying 1% FDR filtration at both peptide precursor and protein levels. Then apply the inclusion criteria sequentially: retain only precursors quantified consistently across all files, enforce sequence-length constraints (final panel is 90% 7–15 amino acids), exclude peptides with missed cleavages, and drop reactive-residue and tryptic-inhibitory peptides. Next, import the surviving precursors into Skyline and manually inspect transition peak shapes, discarding precursors with unusable transitions. Finally, compute physicochemical properties (GRAVY hydropathy, length distribution) on the retained panel and export it as a TSV; in the source study this converged on exactly 934 precursors with GRAVY ranging from −2.5 to 2.3.

## Related tools

- **DIA-NN** (Analyzes the DIA raw files (v1.7.15) against the spectral library with 1% FDR at precursor and protein levels to produce the peptide quantification that the inclusion criteria are applied to)
- **Skyline** (Used for manual inspection of transition peak shapes of candidate precursors; only Skyline-inspectable precursors with acceptable peak shapes are retained)
- **iDIA-QC** (Companion QC software built on DIA-NN algorithms for evaluating the quality of the DIA files feeding the analysis) — github.com/guomics-lab/iDIA-QC

## Examples

```
diann --f *.raw --lib mouse_speclib.tsv --fdr-pep 0.01 --fdr-protein 0.01 --out quant.tsv  # then filter quant.tsv by consistency, length 7-15 aa, no missed cleavages, and inspect transitions in Skyline
```

## Evaluation signals

- Exported panel contains exactly 934 precursor rows (row_count == 934)
- All GRAVY values in the panel fall within [-2.5, 2.3]
- At least 90% of retained peptides are 7–15 amino acids long
- No retained peptide contains a missed cleavage, a reactive residue, or a tryptic-inhibitory sequence
- Every retained precursor is quantified consistently across all 221 DIA files and has Skyline-inspectable transitions

## Limitations

- The panel size (934) is specific to the 221 retrospective mouse liver digest DIA files and the mouse spectral library; reapplying to other datasets will yield different counts
- The Skyline manual inspection step is subjective and labor-intensive and cannot be fully automated
- The DIA files analyzed here are those deposited in PRIDE by the iDIA-QC study; no accession number is given in the article text, so retrieval may require supplemental lookup

## Evidence

- [results] After applying the inclusion criteria and manual Skyline inspection of transition peak shapes, the panel was narrowed to 934 high-quality peptide precursors: [evidence span withheld — anchor status: not_found]

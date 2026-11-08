---
name: peptide-physicochemical-property-calculation
description: Use when a curated peptide precursor panel has been filtered by inclusion
  criteria (1% FDR, consistency, no missed cleavage, length 7–15 aa, Skyline transition
  inspection) and needs GRAVY/length characterization.
when_to_use_negative:
- The goal is quality assessment of the DIA files themselves (LC/MS status prediction)
  rather than characterizing the peptide panel — that is iDIA-QC's QC-metric and machine-learning
  pipeline, not this skill.
- The precursor list has not yet passed the inclusion criteria (consistency across
  files, length, missed-cleavage, reactive-residue/tryptic-inhibitory exclusion, Skyline
  transition inspection), so properties would be computed on an unvetted panel.
- Input is raw vendor data (.d/.raw/.wiff) or mzXML rather than a quantified, filtered
  precursor table.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: DIA-NN
  role: Produces the precursor-level quantification (v1.7.15, 1% FDR at precursor
    and protein levels) that supplies the precursor panel on which physicochemical
    properties are computed
- name: Skyline
  role: 'Upstream gate: manual inspection of transition peak shapes determines which
    precursors are retained before property calculation'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-54871-1
    title: 'iDIA-QC: AI-empowered data-independent acquisition mass spectrometry-based
      quality control'
  source_content_hash: sha256:50c927d3cb27f00446a3cf1b8562f87eec980c7b3b9837d0e6af7ee4b19fb08e
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
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# peptide-physicochemical-property-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes physicochemical descriptors (GRAVY hydropathy scores and sequence-length distribution) for a filtered DIA peptide precursor panel, characterizing the final curated set of high-quality precursors (e.g. 934 precursors, 90% 7–15 aa, GRAVY −2.5 to 2.3 from retrospective mouse liver digest DIA data).

## When to use

Use when a curated peptide precursor panel has been filtered by inclusion criteria (1% FDR, consistency, no missed cleavage, length 7–15 aa, Skyline transition inspection) and needs GRAVY/length characterization.

## When NOT to use

- The goal is quality assessment of the DIA files themselves (LC/MS status prediction) rather than characterizing the peptide panel — that is iDIA-QC's QC-metric and machine-learning pipeline, not this skill.
- The precursor list has not yet passed the inclusion criteria (consistency across files, length, missed-cleavage, reactive-residue/tryptic-inhibitory exclusion, Skyline transition inspection), so properties would be computed on an unvetted panel.
- Input is raw vendor data (.d/.raw/.wiff) or mzXML rather than a quantified, filtered precursor table.

## Inputs

- DIA-NN precursor-level quantification table (1% FDR at peptide precursor and protein levels) from 221 retrospective mouse liver digest DIA files searched against the 110,126-precursor mouse spectral library
- Filtered peptide precursor list passing inclusion criteria (consistency, 7–15 aa, no missed cleavage, no reactive-residue/tryptic-inhibitory peptides, Skyline-inspectable transitions)
- Peptide amino-acid sequences for each retained precursor

## Outputs

- peptide_precursor_panel.tsv with 934 precursor rows including GRAVY hydropathy scores and sequence lengths
- Sequence-length distribution summary (≥90% of peptides 7–15 aa)
- GRAVY range summary (−2.5 to 2.3)

## How to apply

Start from the precursor-level quantification table produced by DIA-NN (v1.7.15) against the published mouse spectral library (110,126 precursors, 10,542 protein groups), already filtered at 1% FDR at both peptide precursor and protein levels. Confirm the upstream inclusion criteria have been applied: consistency across all 221 retrospective mouse-liver-digest DIA files, sequence-length constraints, no missed cleavages, exclusion of reactive-residue and tryptic-inhibitory peptides, and retention of only Skyline-inspectable transitions. For each retained precursor, compute the GRAVY hydropathy score from its amino-acid sequence (Kyte–Doolittle scale, e.g. via a standard ProtParam-style gravy() call) and record the sequence length in amino acids. Verify the resulting panel contains exactly 934 precursor rows, that all GRAVY values fall within [−2.5, 2.3], and that ≥90% of peptides are 7–15 amino acids long; reject the panel if any criterion fails. Export the verified descriptors as peptide_precursor_panel.tsv.

## Related tools

- **DIA-NN** (Produces the precursor-level quantification (v1.7.15, 1% FDR at precursor and protein levels) that supplies the precursor panel on which physicochemical properties are computed)
- **Skyline** (Upstream gate: manual inspection of transition peak shapes determines which precursors are retained before property calculation)

## Examples

```
from Bio.SeqUtils.ProtParam import ProteinAnalysis; gravy = ProteinAnalysis(seq).gravy()
```

## Evaluation signals

- Exported panel has row_count equal to exactly 934 precursor rows
- All GRAVY values fall within [−2.5, 2.3]
- At least 90% of peptides in the panel are 7–15 amino acids long
- Every retained precursor passed the upstream 1% FDR (peptide precursor and protein) and no-missed-cleavage checks
- Panel is restricted to precursors with Skyline-inspectable transitions from the 221 DIA files

## Limitations

- The panel and its property ranges (934 precursors, GRAVY −2.5 to 2.3, 7–15 aa) are specific to the mouse liver digest DIA dataset and the 110,126-precursor mouse spectral library; properties may shift for other sample types or libraries.
- The length-window constraint (7–15 aa) applied during filtering shapes the resulting property distribution, so the computed descriptors reflect the curation criteria, not the full library.
- Property computation depends on upstream DIA-NN quantification quality; changes to the spectral library or DIA-NN version change which precursors survive filtering.

## Evidence

- [other] the panel was narrowed to 934 high-quality peptide precursors, 90% of which are 7 to 15 amino acids long, with GRAVY values ranging from −2.5 to 2.3.: [evidence span withheld — anchor status: not_found]

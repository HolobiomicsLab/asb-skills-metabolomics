---
name: peptide-fragment-ion-annotation
description: Use when a molecular family of tandem mass spectra contains a shared
  core scaffold with variable modifications — e.
when_to_use_negative:
- The spectra are not from a molecular family with a shared, sequenceable scaffold
  — e.g. small-molecule families without predictable b/y fragment series, where consensus
  peaks cannot be assigned to N- or C-terminal fragment types.
- You need fully automatic, exhaustive fragment annotation — the greedy multiple alignment
  misses smaller alignments such as the y2 peaks in the albicidin family, so annotation
  is incomplete without manual inspection.
- Only two spectra are available — MMSA is designed for multiple-spectrum alignment
  across a molecular family; pairwise tools like Metabolomics Spectrum Resolver or
  ModiFinder may be more appropriate for single comparisons.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: MMSA
  role: Computes the multiple-spectrum alignment that identifies aligned consensus
    peak sets across the molecular family; its output is the input for fragment-ion
    annotation
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: Python Dash
  role: Hosts the interactive Plotly visualization where aligned peaks are inspected,
    clicked for highlighting across spectra, and labeled as b/y fragment types
  repo: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- name: Plotly
  role: Renders the interactive alignment figure with spectra ordered by precursor
    m/z for fragment-series inspection
- name: GNPS2
  role: Source of the molecular family MS/MS spectra via task IDs and Universal Spectrum
    Identifiers
- name: ModiFinder
  role: Complementary tool for structural characterization that this annotation skill
    can feed into
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:e68f8d7a77d556abf6dbf72a2e04b607b22964df1bf46bbff18138b82fd182e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
  tools:
  - Flask
  - GNPS2
  - MMSA
  - ModiFinder
  - NetworkX
  - Plotly
  - Pyteomics
  - Python
  - Python Dash
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Peptide fragment ion annotation

## Summary

Annotate aligned consensus peaks across an MS/MS molecular family as shared b- and y-type fragment ions and use their alignment boundaries to localize chemical modification sites, as demonstrated for the albicidin family where b4/b5 and y2/y3 align across all derivatives and y5 localizes central-residue modifications.

## When to use

Use when a molecular family of tandem mass spectra contains a shared core scaffold with variable modifications — e.g. a linear peptide family like albicidin — and you need to assign aligned fragment peaks as b/y-type ions to pinpoint where carbamoylation, N-terminal hydroxy methylation, or methoxylation occur relative to the precursor series.

## When NOT to use

- The spectra are not from a molecular family with a shared, sequenceable scaffold — e.g. small-molecule families without predictable b/y fragment series, where consensus peaks cannot be assigned to N- or C-terminal fragment types.
- You need fully automatic, exhaustive fragment annotation — the greedy multiple alignment misses smaller alignments such as the y2 peaks in the albicidin family, so annotation is incomplete without manual inspection.
- Only two spectra are available — MMSA is designed for multiple-spectrum alignment across a molecular family; pairwise tools like Metabolomics Spectrum Resolver or ModiFinder may be more appropriate for single comparisons.

## Inputs

- Molecular family MS/MS spectra (MGF format, accessible via USIs of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>)
- MMSA alignment output JSON written to SETS_TEMP_PATH (./temp/sets)
- Precursor m/z values and known or hypothesized peptide scaffold sequence for fragment-mass assignment
- Spectra preprocessed with top-6 peak selection per ±25 m/z window, ±17 m/z precursor exclusion, square-root intensity transformation, and normalization

## Outputs

- Annotated alignment visualization with b/y-type fragment-ion labels on aligned consensus peak sets
- Modification-site localization: assignment of which residues carry carbamoylation, N-terminal hydroxy methylation, or methoxylation based on which fragment series align
- High-resolution SVG export of the annotated alignment figure
- Set information panels reporting m/z values and intensity statistics for each fragment-ion group

## How to apply

After MMSA has computed the transitive alignment and non-transitively aligned consensus peak sets for the family, inspect the interactive Plotly visualization ordered by precursor m/z. Assign each aligned consensus peak set to b-type ions (N-terminal side, e.g. b4, b5) or y-type ions (C-terminal side, e.g. y2, y3, y5) by comparing the peak m/z against expected fragment masses of the linear peptide scaffold. Where b-fragments but not y-fragments align across all spectra, the modification lies on the C-terminal side of the b-ion and vice versa; the article uses this logic to localize modification sites such as methoxylation of the central cyano-alanine residue of Albicidin. Cross-check that high-intensity consensus sets are captured — smaller alignments such as the y2 peaks are known to be missed by the greedy multiple alignment, so lower-intensity diagnostic ions may require manual verification. Export the annotated figure as a high-resolution SVG to document the modification-localization argument.

## Related tools

- **MMSA** (Computes the multiple-spectrum alignment that identifies aligned consensus peak sets across the molecular family; its output is the input for fragment-ion annotation) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **Python Dash** (Hosts the interactive Plotly visualization where aligned peaks are inspected, clicked for highlighting across spectra, and labeled as b/y fragment types) — https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website
- **Plotly** (Renders the interactive alignment figure with spectra ordered by precursor m/z for fragment-series inspection)
- **GNPS2** (Source of the molecular family MS/MS spectra via task IDs and Universal Spectrum Identifiers)
- **ModiFinder** (Complementary tool for structural characterization that this annotation skill can feed into)

## Examples

```
git clone https://github.com/Wang-Bioinformatics-Lab/NetworkFamily_MultipleAlignment_Website; docker-compose up; navigate to http://localhost:5000/setscreation/ and enter mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number> USIs for the albicidin family, then click 'Process and Save JSON'.
```

## Evaluation signals

- In the albicidin-family reference case, b4 and b5 fragment consensus sets must align across ALL derivatives, and y2 and y3 across all derivatives — verify this pattern reproduces before trusting your own annotations.
- y5 fragment consensus sets should align for most (not necessarily all) albicidin derivatives; if y5 alignment is missing for a derivative, check whether the greedy alignment dropped it or the fragment is genuinely absent.
- Confirmed modification-site assignments should match the article's reported pattern: carbamoylation, N-terminal hydroxy methylation, and methoxylation of the central cyano-alanine residue of Albicidin localized via differential b/y alignment boundaries.
- Check the Top-10 Intensity Analysis panel: fragment-ion sets contributing to alignment should rank in the top 10 intensities for most spectra in the set.
- Export the figure as SVG and confirm that clicking a peak highlights all matching peaks across spectra — an invariant of the Interactive Peak Alignment feature.

## Limitations

- Smaller peak alignments such as the y2 peaks in the albicidin family are not fully captured by the greedy multiple alignment, so some low-intensity or narrow-window fragment ions will be missing from the consensus sets and require manual annotation.
- In the rhamnolipid family, MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162, [M+Na]+ of C12H24O3) or rhamnose bound to C12 (m/z 385.219, [M+Na]+ of C18H34O7) — modification-shifted fragments may be dropped entirely.
- Fragment-ion assignment depends on a known or hypothesized scaffold sequence; MMSA aligns peaks but does not perform de novo sequence inference, so b/y-type labels require prior biochemical knowledge of the peptide series.
- Peak filtering (top-6 peaks per ±25 m/z window, ±17 m/z precursor exclusion, square-root transformation) may remove diagnostic low-intensity fragment ions needed for unambiguous annotation.

## Evidence

- [results] In the albicidin family, MMSA aligned prominent b4 and b5 fragments and C-terminal y2 and y3 fragments across all derivatives, and aligned y5 fragments for most derivatives, enabling rapid assignment of modification locations such as carbamoylation, N-terminal hydroxy methylation, and methoxylation of the central cyano-alanine residue of Albicidin.: [evidence span withheld — anchor status: not_found]

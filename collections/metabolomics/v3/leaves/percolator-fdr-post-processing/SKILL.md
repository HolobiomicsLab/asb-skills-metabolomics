---
name: percolator-fdr-post-processing
description: Use when database-search engine output (Tide, Comet, MSGF+ PSMs) must
  be converted into an FDR-controlled unique peptide list — e.g. to benchmark de novo
  sequencing results like Casanovo's against a reference search at matched confidence.
when_to_use_negative:
- The spectra have no usable protein sequence database (contaminants, sequence variants,
  unexpected PTMs, foreign peptides) — this is exactly the regime the article says
  motivates de novo sequencing instead of database search plus Percolator
- You only have de novo predictions (e.g. Casanovo mzTab output) with no target/decoy
  database-search PSMs to rescore — Percolator requires database-search scores as
  input
- Enzymatic tryptic digest data where an enzymatic Tide search plus standard FDR estimation
  is already sufficient and the no-enzyme search would only inflate the search space
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Tide
  role: Non-enzymatic (no-enzyme digest) database search engine whose PSMs are the
    input to Percolator; produced the reference 1544 unique peptides at 1% FDR
  repo: http://cruxtoolkit.sourceforge.net
- name: Percolator
  role: Semi-supervised post-processing tool that rescores target/decoy PSMs and enforces
    the 1% FDR cutoff on the unique peptide set
- name: SearchGUI
  role: Alternative common interface for configuring and running search engines including
    Tide, if Tide is driven through a GUI/pipeline wrapper
  repo: https://github.com/CompOmics/searchgui
- name: Casanovo
  role: De novo sequencing counterpart whose unique peptide predictions are compared
    against the Percolator-filtered Tide peptide set at 1% FDR
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:5b1cae05859772505415d007c293b2817c54f4cd9e0006ce3cb242d5ddc4eb73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - Percolator
  - SearchGUI
  - Tide
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# percolator-fdr-post-processing

## Summary

Post-process database-search PSMs (e.g., from Tide) with Percolator's semi-supervised rescoring to establish peptide-level FDR control at a fixed threshold (1% in the MDA-MB-231 immunopeptidomics benchmark), yielding a trustworthy unique peptide set for comparison against de novo predictions.

## When to use

Use when database-search engine output (Tide, Comet, MSGF+ PSMs) must be converted into an FDR-controlled unique peptide list — e.g. to benchmark de novo sequencing results like Casanovo's against a reference search at matched confidence.

## When NOT to use

- The spectra have no usable protein sequence database (contaminants, sequence variants, unexpected PTMs, foreign peptides) — this is exactly the regime the article says motivates de novo sequencing instead of database search plus Percolator
- You only have de novo predictions (e.g. Casanovo mzTab output) with no target/decoy database-search PSMs to rescore — Percolator requires database-search scores as input
- Enzymatic tryptic digest data where an enzymatic Tide search plus standard FDR estimation is already sufficient and the no-enzyme search would only inflate the search space

## Inputs

- Tide (or other database search engine) PSM output with target and decoy scores from the five MDA-MB-231 MHC class I immunopeptidomics MS/MS runs
- Human proteome FASTA database used as the non-enzymatic (no-enzyme digest) search target
- FDR threshold specification (1%)

## Outputs

- Percolator-rescored PSM list with q-values at 1% FDR
- Set of 1544 unique database-search peptide sequences (benchmark reference count) for overlap computation against the Casanovo de novo set (26,377 unique predictions, 2459 matching the human proteome)
- Peptide counts feeding immunopeptidomics_peptide_counts.csv

## How to apply

Run the database search engine first with settings matched to the biology — for the MHC class I immunopeptidomics comparison, Tide is run with non-specific (no-enzyme) digest settings because MHC-presented peptides are not tryptic. Feed the Tide PSMs (with target and decoy scores and features) into Percolator, which learns a semi-supervised score separating targets from decoys and reports PSMs at the desired FDR threshold — 1% FDR for this benchmark. Collapse the accepted PSMs to unique peptide sequences (the reference reproduces 1544 unique peptides for five MDA-MB-231 runs at 1% FDR). The rationale for post-processing is that raw search-engine scores are not calibrated: Percolator's learned discriminant gives a rigorous FDR estimate needed for a fair head-to-head overlap computation with a de novo tool's peptide set, and NetMHCpan-4.1 binder-fraction scoring downstream.

## Related tools

- **Tide** (Non-enzymatic (no-enzyme digest) database search engine whose PSMs are the input to Percolator; produced the reference 1544 unique peptides at 1% FDR) — http://cruxtoolkit.sourceforge.net
- **Percolator** (Semi-supervised post-processing tool that rescores target/decoy PSMs and enforces the 1% FDR cutoff on the unique peptide set)
- **SearchGUI** (Alternative common interface for configuring and running search engines including Tide, if Tide is driven through a GUI/pipeline wrapper) — https://github.com/CompOmics/searchgui
- **Casanovo** (De novo sequencing counterpart whose unique peptide predictions are compared against the Percolator-filtered Tide peptide set at 1% FDR) — https://github.com/Noble-Lab/casanovo

## Evaluation signals

- The filtered peptide set at 1% FDR contains 1544 unique peptides for the five MDA-MB-231 MHC class I runs (reference reproduction count)
- All accepted PSMs satisfy the 1% peptide-level FDR threshold estimated from target-decoy competition after Percolator rescoring
- Tide search used non-specific (no-enzyme) digest settings, consistent with the non-tryptic nature of the MHC I immunopeptidome
- The overlap with the Casanovo peptide set (26,377 unique, 2459 proteome-matching) is computed at matched 1% FDR, not raw score cutoffs
- Peptide counts are assembled reproducibly into immunopeptidomics_peptide_counts.csv for downstream NetMHCpan-4.1 binder-fraction scoring (87%/86%/50%)

## Limitations

- Database searching requires the set of proteins in the sample to be known in advance, preventing detection of unexpected peptides (contaminants, sequence variants, unexpected PTMs, foreign peptides) — the filtered peptide set is bounded by the FASTA supplied
- No-enzyme digest dramatically enlarges the search space, which affects the statistical power of the FDR-controlled comparison
- The article notes that whether, for a given DDA dataset, the de novo tool outperforms a standard database search in statistical power to detect peptides remains an open question, so the 1% FDR peptide set is a baseline, not a ground truth

## Evidence

- [other] Run Tide with non-specific (no-enzyme) digest settings, post-process with Percolator at 1% FDR, and verify 1544 unique database-search peptides.: [evidence span withheld — anchor status: not_found]
- [intro] Database search requires known proteins in advance: "a fundamental requirement for sequence database searching is that the set of proteins that may be present in the sample is known in advance."
- [discussion] Open question: database search vs de novo statistical power: "an open question is whether, for a given data-dependent acquisition dataset, Casanovo outperforms a standard database search procedure in terms of statistical power to detect peptides."
- [readme] SearchGUI supports Tide among search engines: [evidence span withheld — anchor status: external_doc]
- [intro] Dozens of database search engines widely deployed: "Pioneered by the SEQUEST algorithm2, dozens of database search engines have been subsequently developed and are very widely deployed3."

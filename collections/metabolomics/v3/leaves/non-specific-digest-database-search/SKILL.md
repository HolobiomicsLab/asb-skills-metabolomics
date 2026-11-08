---
name: non-specific-digest-database-search
description: Use when analyzing MHC class I immunopeptidomics (non-enzymatic) DDA
  spectra and you need a database-search reference peptide set at 1% FDR to compare
  against de novo sequencing predictions.
when_to_use_negative:
- The sample contains an enzymatic (e.g., tryptic) digest, where a standard enzyme-constrained
  database search is the appropriate baseline rather than a no-enzyme digest
- The set of proteins possibly present in the sample is unknown, since database searching
  cannot detect unexpected peptides (contaminants, sequence variants, unexpected PTMs,
  foreign peptides)
- The analysis goal is identifying peptides absent from any reference proteome, which
  requires de novo sequencing instead
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Tide
  role: Non-enzymatic (no-enzyme digest) database search engine used to generate the
    reference peptide identifications from MHC immunopeptidomics spectra
  repo: http://cruxtoolkit.sourceforge.net
- name: Percolator
  role: Post-processing rescoring of Tide PSMs to establish the 1% FDR peptide identification
    threshold
- name: SearchGUI
  role: Alternative common interface for configuring and running Tide database searches
    from the command line or GUI
  repo: https://github.com/CompOmics/searchgui
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:cfcab6d6ce816a1b4f643bc0425344098c4aaa90e6a9bfa2e94c082d04be4d71
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - Percolator
  - SearchGUI
  - Tide
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# non-specific-digest-database-search

## Summary

Establishes a non-enzymatic sequence-database-search baseline for MHC class I immunopeptidomics by running Tide with no-enzyme digest settings against the human proteome, then rescoring PSMs with Percolator at 1% FDR. It is used to benchmark de novo sequencing (Casanovo) peptide yield and overlap on the same spectra.

## When to use

Use when analyzing MHC class I immunopeptidomics (non-enzymatic) DDA spectra and you need a database-search reference peptide set at 1% FDR to compare against de novo sequencing predictions.

## When NOT to use

- The sample contains an enzymatic (e.g., tryptic) digest, where a standard enzyme-constrained database search is the appropriate baseline rather than a no-enzyme digest
- The set of proteins possibly present in the sample is unknown, since database searching cannot detect unexpected peptides (contaminants, sequence variants, unexpected PTMs, foreign peptides)
- The analysis goal is identifying peptides absent from any reference proteome, which requires de novo sequencing instead

## Inputs

- MHC class I immunopeptidomics DDA tandem mass spectrometry runs (e.g., five MDA-MB-231 spectra files)
- Human proteome reference FASTA database

## Outputs

- Set of unique database-search peptides accepted at 1% FDR (1544 unique peptides in the reference MDA-MB-231 analysis)
- Percolator-rescored PSM table for the no-enzyme Tide search
- Peptide overlap statistics between the database-search set and the de novo (Casanovo) peptide set

## How to apply

Configure Tide (part of the Crux toolkit, also supported as a search engine through SearchGUI) to search the MHC immunopeptidomics runs against a human proteome FASTA using non-specific (no-enzyme) digest settings, since MHC ligands are not tryptic. Post-process the Tide PSMs with Percolator, controlling peptide-spectrum matches at 1% FDR, and deduplicate the accepted matches to unique peptide sequences. In the reference reproduction on five MDA-MB-231 MHC class I runs this yields 1544 unique peptides, which is then compared to the 26,377 unique de novo predictions from Casanovo (of which 2459 match the human proteome); a majority of the human-matched de novo peptides overlap the Tide set. Finally, the overlap of the two peptide sets at 1% FDR is computed as the head-to-head comparison statistic.

## Related tools

- **Tide** (Non-enzymatic (no-enzyme digest) database search engine used to generate the reference peptide identifications from MHC immunopeptidomics spectra) — http://cruxtoolkit.sourceforge.net
- **Percolator** (Post-processing rescoring of Tide PSMs to establish the 1% FDR peptide identification threshold)
- **SearchGUI** (Alternative common interface for configuring and running Tide database searches from the command line or GUI) — https://github.com/CompOmics/searchgui

## Examples

```
docker run quay.io/biocontainers/searchgui:X.Y.Z--1 searchgui eu.isas.searchgui.cmd.IdentificationParametersCLI -out params -db /myresources/uniprot-human-reviewed.fasta
```

## Evaluation signals

- Deduplicated Tide peptide set at 1% FDR matches the expected count of 1544 unique peptides on the five MDA-MB-231 MHC class I runs
- Percolator-rescored PSMs satisfy the 1% FDR threshold before deduplication
- Tide digest configuration is confirmed to be non-specific (no-enzyme) rather than tryptic
- Overlap with the Casanovo de novo set is computed: a majority of the 2459 human-matched de novo peptides overlap the 1544 database-search peptides
- Human-matching rate of de novo predictions (2459 of 26,377) is consistent with the non-enzymatic nature of the immunopeptidome

## Limitations

- Database searching requires the set of proteins in the sample to be known in advance, preventing detection of unexpected peptides such as contaminants, sequence variants, unexpected PTMs, or foreign peptides
- It remains an open question whether, for a given DDA dataset, a database search outperforms de novo sequencing in statistical power to detect peptides
- No-enzyme digests greatly inflate the search space, so FDR control (here 1% via Percolator) is critical to keep the non-specific search peptide set reliable

## Evidence

- [intro] Database searching requires known proteome: "a fundamental requirement for sequence database searching is that the set of proteins that may be present in the sample is known in advance."
- [discussion] Open question: database search vs de novo power: "an open question is whether, for a given data-dependent acquisition dataset, Casanovo outperforms a standard database search procedure in terms of statistical power to detect peptides."
- [intro] Database search engines widely deployed: "Pioneered by the SEQUEST algorithm2, dozens of database search engines have been subsequently developed and are very widely deployed3."
- [readme] SearchGUI supports Tide search engine: [evidence span withheld — anchor status: external_doc]

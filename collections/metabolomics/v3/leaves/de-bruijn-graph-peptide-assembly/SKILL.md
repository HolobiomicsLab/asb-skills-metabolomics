---
name: de-bruijn-graph-peptide-assembly
description: Use when a set of de novo MS/MS peptide predictions (multi-enzymatic
  antibody digests) must be stitched into full-length light/heavy chain contigs without
  a reference database.
when_to_use_negative:
- When a reference database is available and database search (e.g. spectrum graph
  or database algorithms) can identify peptides — the article positions ALPS assembly
  as the database-independent reconstruction route, not as a substitute for DB search
- When isoleucine/leucine or other isobaric residue distinctions are required — MS/MS
  cannot discriminate I & L (nor Q & AG or deamidated N & D), so assembled contigs
  cannot resolve them
- When high-quality full-length heavy-chain assembly is the goal — the article reports
  low heavy-chain coverage/accuracy, showing complete de novo protein sequencing remains
  challenging
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: ALPS
  role: de Bruijn graph assembler that stitches confidence-filtered de novo peptide
    predictions into antibody chain contigs (k-mer size 7)
- name: Casanovo
  role: deep-learning de novo peptide sequencer whose confidence-filtered predictions
    serve as the primary input reads for ALPS assembly
- name: BLAST
  role: aligns assembled contigs to ground-truth antibody sequences to derive per-chain
    sequence coverage and accuracy
- name: Novor
  role: one of the evaluated de novo sequencing tools whose predictions can alternatively
    feed the ALPS assembly workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:85f55140b6116c473e0d83864c547a7fe696d9fe0fb24958a092d525129a0c9b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALPS
  - BLAST
  - Casanovo
  - Novor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# de-bruijn-graph-peptide-assembly

## Summary

Assembles confidence-filtered de novo peptide predictions (e.g. from Casanovo) into longer antibody chain contigs using the de Bruijn graph assembler ALPS, then aligns contigs to ground-truth sequences with BLAST to compute per-chain coverage and accuracy.

## When to use

Use when a set of de novo MS/MS peptide predictions (multi-enzymatic antibody digests) must be stitched into full-length light/heavy chain contigs without a reference database.

## When NOT to use

- When a reference database is available and database search (e.g. spectrum graph or database algorithms) can identify peptides — the article positions ALPS assembly as the database-independent reconstruction route, not as a substitute for DB search
- When isoleucine/leucine or other isobaric residue distinctions are required — MS/MS cannot discriminate I & L (nor Q & AG or deamidated N & D), so assembled contigs cannot resolve them
- When high-quality full-length heavy-chain assembly is the goal — the article reports low heavy-chain coverage/accuracy, showing complete de novo protein sequencing remains challenging

## Inputs

- Confidence-filtered de novo peptide predictions from a de novo sequencing tool (e.g. Casanovo, Novor, pNovo 3, DeepNovo, SMSNet, PointNovo)
- Ground-truth antibody reference sequences (for coverage/accuracy assessment via BLAST)
- Natural Antibody database reference CDR sequences (for variable-region/CDR assembly evaluation)

## Outputs

- Assembled antibody-chain contigs (longest contig per chain; light chains ~210–219 AAs, heavy chains >440 AAs)
- Per-chain protein sequence coverage and accuracy values derived from BLAST alignments
- Variable-region/CDR assembly coverage statistics

## How to apply

Load the confidence-score-filtered peptide predictions — in the evaluated pipeline the authors used Casanovo predictions after applying each tool's confidence-score threshold where amino-acid-level precision fell below 50% — and feed them into the de Bruijn assembler ALPS with a k-mer size of 7, chosen to balance sequence coverage against contig repetitiveness. ALPS stitches the short peptide reads into contigs, which for antibody light chains can reach 210–219 amino acids and for heavy chains exceed 440 amino acids. Align the assembled target contigs to ground-truth antibody sequences using BLAST, treating a target (sub)sequence as covered when a contig aligns to it. From the BLAST alignments compute per-chain protein sequence coverage and accuracy, recording the longest constructed contig for each chain. Also evaluate assembly of the functionally important variable regions and CDRs, using the Natural Antibody database to identify the reference CDR sequences for each antibody. Validation targets from the article: light-chain coverage 97.69–99.53% with 94.47–95.26% accuracy using Casanovo predictions; heavy-chain coverage reproduces the lower reported range (76.39–93.72%).

## Related tools

- **ALPS** (de Bruijn graph assembler that stitches confidence-filtered de novo peptide predictions into antibody chain contigs (k-mer size 7))
- **Casanovo** (deep-learning de novo peptide sequencer whose confidence-filtered predictions serve as the primary input reads for ALPS assembly)
- **BLAST** (aligns assembled contigs to ground-truth antibody sequences to derive per-chain sequence coverage and accuracy)
- **Novor** (one of the evaluated de novo sequencing tools whose predictions can alternatively feed the ALPS assembly workflow)

## Examples

```
alps -k 7 -i casanovo_predictions.pep -o assembled_contigs.fa
```

## Evaluation signals

- Light-chain coverage using Casanovo predictions falls within the reported 97.69–99.53% range with 94.47–95.26% accuracy (matches Table 4)
- Heavy-chain coverage falls within the lower reported range (76.39–93.72% with Casanovo)
- Longest constructed contig per chain is within the expected size window (light chains 210–219 AAs; heavy chains >440 AAs)
- Coverage of the variable regions and CDRs (identified via the Natural Antibody database) is reported alongside whole-chain coverage
- Every BLAST alignment covering a target (sub)sequence is counted as covered — verify the coverage definition is applied consistently when computing coverage from BLAST output

## Limitations

- Low sequence coverage and accuracy on heavy chains demonstrate that complete de novo protein sequencing remains a challenging issue in proteomics
- Particular amino acids cannot be resolved even with higher-resolution spectra and tools (e.g. I & L; Q & AG; deamidated N & D)
- Database-independent full-length protein assembly is limited even when using multiple contigs and different de novo sequencing tools
- The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools
- Advanced post-processing steps (e.g. precursor mass filter) are necessary to improve accuracy of newly developed de novo sequencing tools

## Evidence

- [abstract] Light-chain coverage and accuracy with Casanovo: "We achieved a sequence coverage of 97.69-99.53% on the light chains of three different antibody data sets using the de Bruijn assembler ALPS and the predictions from Casanovo."
- [abstract] Heavy-chain challenge: "However, low sequence coverage and accuracy on the heavy chains demonstrate that complete de novo protein sequencing remains a challenging issue in proteomics"
- [discussion] Unresolvable isomeric residues: "However, particular amino acids cannot be resolved even with spectra and tools with higher resolution (e.g. I & L; Q & AG; deamidated N & D)"
- [discussion] Assembly limited even with multiple contigs: "the ability of database-independent approaches of full-length protein assembly is limited even when using multiple contigs and different de novo sequencing tools (Table 4)"
- [discussion] Post-processing needed: "While newly developed tools demonstrate the potential of de novo sequencing, advanced post-processing steps are necessary to improve their accuracy"

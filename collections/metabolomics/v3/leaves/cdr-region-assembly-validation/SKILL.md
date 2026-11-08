---
name: cdr-region-assembly-validation
description: Use when de novo peptide predictions have been assembled into antibody
  contigs (e.g. with ALPS) and you must verify that CDR and variable-region sequences
  were recovered, not just overall chain coverage.
when_to_use_negative:
- When only whole-chain coverage is needed and CDR-level resolution is not required
  — plain BLAST-based coverage against the full chain suffices.
- When the goal is database-search-based sequence confirmation (e.g. Supernovo-style
  workflows with in silico genetic recombination) rather than database-independent
  de novo assembly validation.
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_0121
tools:
- name: ALPS
  role: de Bruijn assembler (k-mer 7) producing the contigs whose CDR/variable-region
    coverage is validated
- name: Casanovo
  role: deep-learning de novo sequencer whose confidence-filtered peptide predictions
    feed the assembly
- name: BLAST
  role: aligns assembled contigs to ground-truth chains and CDR references to score
    coverage
- name: Natural Antibody database
  role: source of reference CDR sequences for each evaluated antibody
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:c651840bde2fb8ac6392e8451b7310f3dbc069e977c69c33f9569e1a653243e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALPS
  - BLAST
  - Casanovo
  - Natural Antibody database
  - Novor
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cdr-region-assembly-validation

## Summary

Validation step for de novo antibody assembly that assesses whether assembled contigs reconstruct the functionally important variable regions and complementarity-determining regions (CDRs), using the Natural Antibody database to retrieve reference CDR sequences for each antibody.

## When to use

Use when de novo peptide predictions have been assembled into antibody contigs (e.g. with ALPS) and you must verify that CDR and variable-region sequences were recovered, not just overall chain coverage.

## When NOT to use

- When only whole-chain coverage is needed and CDR-level resolution is not required — plain BLAST-based coverage against the full chain suffices.
- When the goal is database-search-based sequence confirmation (e.g. Supernovo-style workflows with in silico genetic recombination) rather than database-independent de novo assembly validation.

## Inputs

- Assembled antibody contigs from ALPS (k-mer size 7) built from confidence-filtered de novo peptide predictions
- Ground-truth antibody light- and heavy-chain protein sequences
- Reference CDR sequences per antibody retrieved from the Natural Antibody database

## Outputs

- Per-CDR and per-variable-region sequence coverage (fraction of reference CDR residues covered by aligned contigs)
- Longest constructed contig per chain (light chains 210-219 AAs; heavy chains >440 AAs)
- BLAST alignments of contigs against ground-truth chain and CDR sequences

## How to apply

After assembling confidence-filtered de novo peptide predictions (e.g. Casanovo output filtered at each tool's precision threshold) with the de Bruijn assembler ALPS at k-mer size 7, restrict evaluation to the variable regions and CDRs. Identify the reference CDR sequences for each antibody using the Natural Antibody database, then check BLAST alignments of the assembled target contigs against the ground-truth chain sequences to determine whether each CDR (sub)sequence is covered. Record per-CDR and per-variable-region coverage alongside whole-chain coverage (light chains 210-219 AAs, heavy chains >440 AAs) so that functional-binding-critical positions are not masked by good constant-region recovery. Interpret results in light of the paper's finding that heavy chains show much lower coverage (76.39-93.72% with Casanovo) than light chains (97.69-99.53%), meaning CDR3 of heavy chains is a likely failure point.

## Related tools

- **ALPS** (de Bruijn assembler (k-mer 7) producing the contigs whose CDR/variable-region coverage is validated)
- **Casanovo** (deep-learning de novo sequencer whose confidence-filtered peptide predictions feed the assembly)
- **BLAST** (aligns assembled contigs to ground-truth chains and CDR references to score coverage)
- **Natural Antibody database** (source of reference CDR sequences for each evaluated antibody)

## Examples

```
blastp -query alps_contigs.fasta -subject antibody_light_chain_gt.fasta -outfmt 6; # then map aligned residues onto NAbDb CDR annotations per chain
```

## Evaluation signals

- Light-chain CDR/variable-region coverage is consistent with the reported whole-chain coverage of 97.69-99.53% (Casanovo + ALPS).
- Heavy-chain coverage falls in the lower reported range of 76.39-93.72% with Casanovo; substantially higher values suggest spurious contig alignment.
- Every reported covered CDR is backed by a BLAST alignment of at least one assembled contig to the reference CDR sequence.
- Per-chain longest contig lengths are plausible: light chains 210-219 AAs, heavy chains >440 AAs.
- Residues reported as covered are counted only where a contig aligns in the BLAST hit, matching the paper's coverage definition.

## Limitations

- Isoleucine/leucine and other isomeric or mass-degenerate residues (I & L; Q & AG; deamidated N & D) cannot be resolved via MS/MS, so CDR 'accuracy' is ambiguous at these positions.
- Heavy-chain (especially CDR-containing variable region) coverage is substantially lower than light-chain coverage; database-independent full-length assembly remains limited even with multiple contigs and tools.
- CDR reference identity depends on the Natural Antibody database annotations; misassigned CDR boundaries propagate into the validation.
- Missing fragmentation sites, noisy spectra and long peptides limit all de novo tools, so some CDR segments may simply have no supporting predictions.

## Evidence

- [other] Evaluate assembly of the functionally important variable regions and CDRs, using the Natural Antibody database to identify the reference CDR sequences for each antibody.: [evidence span withheld — anchor status: not_found]
- [abstract] We achieved a sequence coverage of 97.69-99.53% on the light chains of three different antibody data sets using the de Bruijn assembler ALPS and the predictions from Casanovo.: "We achieved a sequence coverage of 97.69-99.53% on the light chains of three different antibody data sets using the de Bruijn assembler ALPS and the predictions from Casanovo."
- [abstract] However, low sequence coverage and accuracy on the heavy chains demonstrate that complete de novo protein sequencing remains a challenging issue in proteomics.: "However, low sequence coverage and accuracy on the heavy chains demonstrate that complete de novo protein sequencing remains a challenging issue in proteomics"
- [discussion] Discrimination of the isomeric residues isoleucine and leucine cannot be achieved via MS/MS but require MS3 fragmentation.: "Discrimination of the isomeric residues isoleucine and leucine cannot be achieved via MS/MS but require MS3 fragmentation."
- [discussion] the ability of database-independent approaches of full-length protein assembly is limited even when using multiple contigs and different de novo sequencing tools (Table 4): "the ability of database-independent approaches of full-length protein assembly is limited even when using multiple contigs and different de novo sequencing tools (Table 4)"

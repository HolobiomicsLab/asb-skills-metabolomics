---
name: metagenomic-protein-database-construction
description: Use when analyzing metaproteomics tandem MS/MS spectra against metagenome-derived
  protein databases (metapeptide or non-redundant environmental FASTA), or when benchmarking
  database search vs de novo peptide-detection counts.
when_to_use_negative:
- The sample's protein complement is known in advance (e.g. a single organism) — sequence
  database searching against a standard reference database suffices and metagenome-informed
  database construction is unnecessary
- The goal is assigning de novo peptides to foreign spectra beyond target-decoy matching
  — the target-decoy matching approach 'misses out on the power of de novo sequencing
  to assign peptides to foreign spectra'
- The data are not bottom-up DDA tandem MS/MS spectra suitable for Casanovo or Tide
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: Tide
  role: Database search engine run against the metapeptide and environmental protein
    databases to produce the standard-search PSM identifications
  repo: http://cruxtoolkit.sourceforge.net
- name: Percolator
  role: Re-scores Tide PSMs to obtain confident identifications at 1% FDR
- name: Casanovo
  role: De novo sequencing of the same spectra; predictions are matched against target
    and decoy peptide databases for confidence estimation and comparison with database-search
    counts
  repo: https://github.com/Noble-Lab/casanovo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:58672772c98d9df9e7876223126b9105c0930ec1a069dfe98c6acc967d9a4e19
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - Percolator
  - Tide
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# metagenomic-protein-database-construction

## Summary

Constructs and applies sample-specific protein search databases for metaproteomics — a metagenome-derived metapeptide database and a non-redundant environmental protein database — so that database-search (Tide+Percolator) and de novo (Casanovo) peptide detection can be compared against the same matched peptide space.

## When to use

Use when analyzing metaproteomics tandem MS/MS spectra against metagenome-derived protein databases (metapeptide or non-redundant environmental FASTA), or when benchmarking database search vs de novo peptide-detection counts.

## When NOT to use

- The sample's protein complement is known in advance (e.g. a single organism) — sequence database searching against a standard reference database suffices and metagenome-informed database construction is unnecessary
- The goal is assigning de novo peptides to foreign spectra beyond target-decoy matching — the target-decoy matching approach 'misses out on the power of de novo sequencing to assign peptides to foreign spectra'
- The data are not bottom-up DDA tandem MS/MS spectra suitable for Casanovo or Tide

## Inputs

- Metaproteomics tandem MS/MS spectra files (DDA, from MassIVE accession MSV000094709)
- Metapeptide database (metagenome-assembled sample-specific peptide/protein database)
- Non-redundant environmental protein database
- Corresponding decoy peptide database for target-decoy confidence estimation

## Outputs

- peptide_counts.csv with per-tool, per-database unique peptide counts
- Tide+Percolator PSM identifications filtered at 1% FDR
- Casanovo de novo peptide predictions matched to target/decoy databases at a 1% random-matching criterion

## How to apply

Download the tandem MS spectra (e.g. MassIVE accession MSV000094709, Bering Strait and Chukchi Sea samples) together with the pre-built metapeptide database and the non-redundant environmental protein database. Run Tide against each database and re-score PSMs with Percolator, filtering at 1% FDR, to obtain the standard database-search identifications. Run Casanovo de novo sequencing on the same spectra and match predicted peptides against a target and corresponding decoy peptide database, applying a target-decoy-style 1% random-matching filter for confidence. Count unique peptides passing each filter per tool and per database (metapeptide vs environmental), writing the per-tool, per-database counts to peptide_counts.csv for comparison.

## Related tools

- **Tide** (Database search engine run against the metapeptide and environmental protein databases to produce the standard-search PSM identifications) — http://cruxtoolkit.sourceforge.net
- **Percolator** (Re-scores Tide PSMs to obtain confident identifications at 1% FDR)
- **Casanovo** (De novo sequencing of the same spectra; predictions are matched against target and decoy peptide databases for confidence estimation and comparison with database-search counts) — https://github.com/Noble-Lab/casanovo

## Examples

```
casanovo db_search
```

## Evaluation signals

- Reproduced Tide counts must match reference values for Fig. 4a-b: 5623 metapeptide / 2460 environmental peptides at 1% FDR (1364/682 with full-length matching)
- Reproduced Casanovo counts must match reference values: 8277 metapeptide / 3532 environmental peptides at 1% random matching (3425/1612 full-length), i.e. ~47% and ~44% increases over Tide+Percolator
- Both tools are filtered at the same 1% confidence level (FDR for Tide+Percolator, random-matching criterion for Casanovo) before counting
- Per-tool, per-database unique peptide counts are written to peptide_counts.csv; discrepancies from reference values are flagged for expert review

## Limitations

- Matching Casanovo predictions to a target/decoy peptide database misses the power of de novo sequencing to assign peptides to foreign spectra
- Sequence database searching requires the set of proteins present in the sample to be known in advance, preventing detection of unexpected peptides (contaminants, sequence variants, unexpected PTMs, foreign peptides)
- No rigorous method for confidence estimation for de novo sequencing exists; the target-decoy-style random-matching filter is an approximation

## Evidence

- [other] Download the metaproteomics tandem MS spectra files from the MassIVE accession MSV000094709 together with the metapeptide and non-redundant environmental protein databases.: [evidence span withheld — anchor status: not_found]
- [discussion] Target-decoy matching limits de novo power: "we have matched Casanovo predictions to a target and corresponding decoy peptide database, but such an approach misses out on the power of de novo sequencing to assign peptides to foreign spectra."
- [intro] Database search requires known proteins: "a fundamental requirement for sequence database searching is that the set of proteins that may be present in the sample is known in advance."
- [abstract] Casanovo benefits metaproteomics: "Casanovo’s superior performance improves the analysis of immunopeptidomics and metaproteomics experiments and allows us to delve deeper into the dark proteome"
- [readme] Casanovo is intended for peptides outside known databases: "particularly useful for immunopeptidomics, metaproteomics, paleoproteomics, venomics, or any setting in which you are interested in identifying peptides that may not be in your protein database."

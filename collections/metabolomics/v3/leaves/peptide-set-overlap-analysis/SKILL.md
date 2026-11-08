---
name: peptide-set-overlap-analysis
description: Use when you have two peptide identification result sets — e.g. de novo
  mzTab predictions and a database-search PSM list at a stated FDR — and must quantify
  their overlap and per-set properties.
when_to_use_negative:
- Input spectra come from DIA acquisition — Casanovo and this comparison target data-dependent
  acquisition, bottom-up tandem MS datasets
- The sample's protein set is fully known and the goal is standard identification
  — a plain database search suffices, and the overlap analysis adds no information
  about unexpected peptides
- Spectra have fewer peaks than the configured minimum (min_peaks) or predictions
  fail the minimum peptide length filter — these low-quality entries would distort
  set counts
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Casanovo
  role: Generates the de novo peptide predictions (beam search decoding, non-enzymatic
    immunopeptidomics model) exported as mzTab for one side of the overlap
  repo: https://github.com/Noble-Lab/casanovo
- name: Tide
  role: Non-enzymatic (no-enzyme digest) database search producing the reference peptide
    set
- name: Percolator
  role: Post-processes Tide results to control peptide FDR at 1% before deduplication
- name: NetMHCpan-4.1
  role: Scores peptide sets for MHC class I binding to compute binder fractions per
    peptide category
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:11d9afb68ed2f794a7f8a2be50afc7d8ddc702ec1f1caef3acd6ceddc28b0911
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - NetMHCpan-4.1
  - Percolator
  - Tide
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# peptide-set-overlap-analysis

## Summary

Compares de novo peptide predictions (Casanovo, exported as mzTab) against a non-enzymatic database-search peptide set (Tide + Percolator at 1% FDR) to quantify set overlap and binding-predicted fractions in MHC class I immunopeptidomics data.

## When to use

Use when you have two peptide identification result sets — e.g. de novo mzTab predictions and a database-search PSM list at a stated FDR — and must quantify their overlap and per-set properties.

## When NOT to use

- Input spectra come from DIA acquisition — Casanovo and this comparison target data-dependent acquisition, bottom-up tandem MS datasets
- The sample's protein set is fully known and the goal is standard identification — a plain database search suffices, and the overlap analysis adds no information about unexpected peptides
- Spectra have fewer peaks than the configured minimum (min_peaks) or predictions fail the minimum peptide length filter — these low-quality entries would distort set counts

## Inputs

- Casanovo de novo peptide predictions exported as mzTab files (one per MS/MS run)
- Tide + Percolator database-search PSM list filtered at 1% FDR (non-specific/no-enzyme digest)
- Human proteome FASTA database for mapping de novo peptide sequences
- MHC class I immunopeptidomics tandem MS runs (e.g. five MDA-MB-231 runs)

## Outputs

- Deduplicated unique-peptide sets per method (e.g. 26,377 Casanovo; 2459 human-matching; 1544 Tide at 1% FDR)
- Overlap counts between the de novo and database-search peptide sets
- NetMHCpan-4.1 binder fractions per peptide category
- Summary table such as immunopeptidomics_peptide_counts.csv

## How to apply

Run Casanovo (non-enzymatic immunopeptidomics model with beam search decoding) over each MHC class I immunopeptidomics run and export de novo predictions as mzTab files, then deduplicate predicted sequences to a unique-peptide set and map them against the human proteome FASTA to keep genome-matching peptides. In parallel, run Tide with non-specific (no-enzyme) digest settings and post-process with Percolator at 1% FDR to obtain the database-search unique-peptide set (e.g. 1544 unique peptides from five MDA-MB-231 runs). Compute the set intersection between the de novo set (26,377 unique predictions, 2459 human-matching) and the database-search set, then score each peptide category with NetMHCpan-4.1 to compute binder fractions (87%/86%/50% in Fig. 3c-d). Rationale: de novo sequencing detects peptides outside the searched database, so overlap analysis directly tests whether the transformer model recovers database-searchable peptides while additionally surfacing non-database peptides.

## Related tools

- **Casanovo** (Generates the de novo peptide predictions (beam search decoding, non-enzymatic immunopeptidomics model) exported as mzTab for one side of the overlap) — https://github.com/Noble-Lab/casanovo
- **Tide** (Non-enzymatic (no-enzyme digest) database search producing the reference peptide set)
- **Percolator** (Post-processes Tide results to control peptide FDR at 1% before deduplication)
- **NetMHCpan-4.1** (Scores peptide sets for MHC class I binding to compute binder fractions per peptide category)

## Evaluation signals

- Deduplicated Casanovo predictions total 26,377 unique peptides
- Exactly 2459 Casanovo peptides map to the human proteome FASTA
- Tide + Percolator yields 1544 unique peptides at 1% FDR
- A majority of the 2459 human-matching de novo peptides overlap the 1544 database-search peptides
- NetMHCpan-4.1 binder fractions reproduce 87%/86%/50% across the three peptide categories of Fig. 3c-d, and all counts assemble into immunopeptidomics_peptide_counts.csv

## Limitations

- No rigorous method for confidence estimation for de novo sequencing exists — a key bottleneck — so set membership at a nominal FDR is not directly comparable to database-search FDR
- Target-decoy matching of de novo predictions to a peptide database misses the power of de novo sequencing to assign peptides to foreign spectra, biasing the human-matching subset
- Whether, for a given DDA dataset, Casanovo outperforms a standard database search in statistical power to detect peptides remains an open question

## Evidence

- [abstract] Casanovo predictions to unique peptide sets and FDR-filtered database-search comparison: "We also develop a version of Casanovo that is fine-tuned for non-enzymatic peptides"
- [methods] mzTab export of de novo predictions: "Export spectrum predictions to mzTab files."
- [discussion] Rationale for comparing de novo vs database-search statistical power: [evidence span withheld — anchor status: not_found]
- [readme] Scope of Casanovo outputs usable in set-overlap analysis: "Casanovo can be used to find unexpected peptide sequences in any data-dependent acquisition, bottom-up tandem mass spectrometry dataset"

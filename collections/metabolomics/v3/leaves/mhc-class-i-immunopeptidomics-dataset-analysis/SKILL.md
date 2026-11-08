---
name: mhc-class-i-immunopeptidomics-dataset-analysis
description: Use when analyzing non-enzymatic MHC class I immunopeptidomics MS/MS
  runs where peptides may lie outside a known protein database, or when comparing
  de novo predictions against a no-enzyme database search at fixed FDR.
when_to_use_negative:
- The sample is a standard enzymatic (tryptic) digest whose peptides are fully representable
  in a known protein database — a conventional database search such as Tide/Percolator
  alone is the appropriate baseline.
- The data were acquired in DIA mode — Casanovo targets DDA bottom-up tandem MS datasets;
  use a DIA-specific de novo framework instead.
- You need calibrated per-spectrum PEP/q-values for de novo predictions — the field
  lacks a rigorous confidence-estimation method for de novo sequencing, so treat de
  novo peptide counts as uncalibrated.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: Casanovo
  role: Transformer-based de novo sequencing of immunopeptidomics spectra using the
    non-enzymatic fine-tuned model with beam search decoding; exports mzTab predictions
  repo: https://github.com/Noble-Lab/casanovo
- name: Tide
  role: Database-search baseline run with non-specific (no-enzyme) digest settings
    on the same spectra
  repo: http://cruxtoolkit.sourceforge.net
- name: Percolator
  role: Post-processing of Tide results to control peptide FDR at 1%
- name: NetMHCpan-4.1
  role: Scoring predicted peptides for MHC class I binding to compute binder fractions
    per peptide category
- name: PDV
  role: Graphical/command-line inspection of peptide identification results from the
    search engines
  repo: https://github.com/wenbostar/PDV/
- name: Limelight converter
  role: Optional conversion of Casanovo mzTab results to Limelight XML for web-based
    review
  repo: https://github.com/yeastrc/limelight-import-casanovo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:cbe9f9513049024cff070e1034fe21caf6a30b36fc5cc21c60fb78dbe7df7af5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - Limelight converter
  - NetMHCpan-4.1
  - PDV
  - Percolator
  - Tide
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mhc-class-i-immunopeptidomics-dataset-analysis

## Summary

End-to-end analysis of MHC class I immunopeptidomics DDA runs by de novo sequencing with Casanovo's non-enzymatic fine-tuned model, benchmarked against a no-enzyme Tide database search post-processed with Percolator at 1% FDR, with NetMHCpan-4.1 binder scoring.

## When to use

Use when analyzing non-enzymatic MHC class I immunopeptidomics MS/MS runs where peptides may lie outside a known protein database, or when comparing de novo predictions against a no-enzyme database search at fixed FDR.

## When NOT to use

- The sample is a standard enzymatic (tryptic) digest whose peptides are fully representable in a known protein database — a conventional database search such as Tide/Percolator alone is the appropriate baseline.
- The data were acquired in DIA mode — Casanovo targets DDA bottom-up tandem MS datasets; use a DIA-specific de novo framework instead.
- You need calibrated per-spectrum PEP/q-values for de novo predictions — the field lacks a rigorous confidence-estimation method for de novo sequencing, so treat de novo peptide counts as uncalibrated.

## Inputs

- MHC class I immunopeptidomics DDA MS/MS runs (raw/MGF/mzML; e.g., five MDA-MB-231 runs from MassIVE)
- Human proteome FASTA database
- Non-enzymatic fine-tuned Casanovo model checkpoint

## Outputs

- Per-run Casanovo de novo peptide predictions in mzTab format
- Deduplicated unique peptide sets (Casanovo and Tide/Percolator at 1% FDR) with overlap counts
- NetMHCpan-4.1 binder fractions per peptide category
- Summary table of reproduced counts and binder fractions (immunopeptidomics_peptide_counts.csv)

## How to apply

Download the MHC class I immunopeptidomics runs (e.g., the five MDA-MB-231 breast cancer cell runs), convert spectra to a Casanovo-compatible format with msconvert, and run the non-enzymatic immunopeptidomics-fine-tuned Casanovo model with beam search decoding, exporting predictions as mzTab files. Deduplicate the Casanovo predictions to unique peptide sequences and map them against the human proteome FASTA to count database-matching peptides. As a database-search baseline, run Tide with non-specific (no-enzyme) digest settings, post-process with Percolator at 1% FDR, and compute the overlap between the two peptide sets. Finally, score the peptide sets (Casanovo-only, Tide-only, and shared) with NetMHCpan-4.1 and compute binder fractions per category; in the reference analysis this yielded 26,377 unique Casanovo peptides, 2459 matching the human proteome, 1544 Tide peptides at 1% FDR, and 87%/86%/50% binder fractions (Fig. 3c-d). Assemble the reproduced counts and binder fractions into a table such as immunopeptidomics_peptide_counts.csv.

## Related tools

- **Casanovo** (Transformer-based de novo sequencing of immunopeptidomics spectra using the non-enzymatic fine-tuned model with beam search decoding; exports mzTab predictions) — https://github.com/Noble-Lab/casanovo
- **Tide** (Database-search baseline run with non-specific (no-enzyme) digest settings on the same spectra) — http://cruxtoolkit.sourceforge.net
- **Percolator** (Post-processing of Tide results to control peptide FDR at 1%)
- **NetMHCpan-4.1** (Scoring predicted peptides for MHC class I binding to compute binder fractions per peptide category)
- **PDV** (Graphical/command-line inspection of peptide identification results from the search engines) — https://github.com/wenbostar/PDV/
- **Limelight converter** (Optional conversion of Casanovo mzTab results to Limelight XML for web-based review) — https://github.com/yeastrc/limelight-import-casanovo

## Examples

```
casanovo sequence mda-mb-231_runs/*.mgf --model casanovo_nonenzymatic.ckpt -o casanovo_predictions.mztab
```

## Evaluation signals

- After deduplication, the Casanovo unique peptide count reproduces the reported total of 26,377 unique peptides across the five runs.
- Exactly 2459 of the Casanovo peptides match the human proteome FASTA under the mapping procedure.
- Tide with no-enzyme digest plus Percolator at 1% FDR yields 1544 unique peptides, and the Casanovo/Tide overlap is reproduced.
- NetMHCpan-4.1 binder fractions across the three peptide categories match the Fig. 3c-d values of 87%/86%/50%.
- All exported mzTab files parse cleanly and contain predictions covering every input run.

## Limitations

- There is no rigorous method for confidence estimation for de novo sequencing — 'one of the key bottlenecks in the field' — so de novo peptide counts lack calibrated FDR.
- It remains an open question whether, for a given DDA dataset, Casanovo outperforms a standard database search in statistical power to detect peptides.
- Target-decoy matching of Casanovo predictions misses the power of de novo sequencing to assign peptides to foreign spectra (e.g., contaminants or sequence variants).

## Evidence

- [abstract] Casanovo’s superior performance improves the analysis of immunopeptidomics and metaproteomics experiments: "Casanovo’s superior performance improves the analysis of immunopeptidomics and metaproteomics experiments"
- [intro] Additionally, we fine-tune a non-enzymatic version of Casanovo for tasks such as immunopeptidomics.: "Additionally, we fine-tune a non-enzymatic version of Casanovo for tasks such as immunopeptidomics."
- [methods] Export spectrum predictions to mzTab files.: "Export spectrum predictions to mzTab files."
- [methods] Casanovo-DB mode (`casanovo db_search`) to use Casanovo as a learned score function for sequence database searching: "Casanovo-DB mode (`casanovo db_search`) to use Casanovo as a learned score function for sequence database searching"
- [readme] Casanovo can be used to find unexpected peptide sequences in any data-dependent acquisition, bottom-up tandem mass spectrometry dataset: "Casanovo can be used to find unexpected peptide sequences in any data-dependent acquisition, bottom-up tandem mass spectrometry dataset"
- [discussion] at this stage one of the key bottlenecks in the field is the absence of a rigorous method for confidence estimation for de novo sequencing: "at this stage one of the key bottlenecks in the field is the absence of a rigorous method for confidence estimation for de novo sequencing"

---
name: netmhcpan-binder-fraction-estimation
description: Use when you have deduplicated peptide sequence sets from MHC class I
  immunopeptidomics (e.g. Casanovo de novo vs Tide at 1% FDR) and need binder fractions
  to compare their immunological plausibility.
when_to_use_negative:
- Input is raw MS/MS spectra rather than already-identified peptide sequences — de
  novo sequencing (Casanovo) or database searching (Tide) must run first
- The goal is peptide identification or FDR estimation, not biological validation
  of an already-identified peptide set
- The data are MHC class II ligands, which require an MHC class II prediction tool
  rather than NetMHCpan
edam_operation: http://edamontology.org/operation_0426
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: NetMHCpan-4.1
  role: Predicts MHC class I binding for each peptide set; binder fractions are computed
    from its predictions
- name: Casanovo
  role: Non-enzymatic fine-tuned model producing the de novo peptide predictions (mzTab
    exports) that are one scored category
  repo: https://github.com/Noble-Lab/casanovo
- name: Tide
  role: Non-specific-digest database search whose Percolator-processed 1% FDR peptides
    form the comparison category
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49731-x
    title: Sequence-to-sequence translation from mass spectra to peptides with a transformer
      model
  source_content_hash: sha256:435e47b25612226d7f58102dc6d7d59d478b676008fa62568dd85ebdd37a1339
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Noble-Lab/casanovo
  tools:
  - Casanovo
  - NetMHCpan-4.1
  - Tide
  edam_operation: http://edamontology.org/operation_0426
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# netmhcpan-binder-fraction-estimation

## Summary

Score peptide sets (e.g. de novo Casanovo predictions vs Tide/Percolator database-search identifications from MHC class I immunopeptidomics) with NetMHCpan-4.1 and compute the fraction of predicted MHC binders in each set, as a biological validation of peptide identifications.

## When to use

Use when you have deduplicated peptide sequence sets from MHC class I immunopeptidomics (e.g. Casanovo de novo vs Tide at 1% FDR) and need binder fractions to compare their immunological plausibility.

## When NOT to use

- Input is raw MS/MS spectra rather than already-identified peptide sequences — de novo sequencing (Casanovo) or database searching (Tide) must run first
- The goal is peptide identification or FDR estimation, not biological validation of an already-identified peptide set
- The data are MHC class II ligands, which require an MHC class II prediction tool rather than NetMHCpan

## Inputs

- Deduplicated peptide sequence sets (one per category, e.g. Casanovo de novo peptides, proteome-matched subset, Tide/Percolator 1% FDR peptides)
- NetMHCpan-4.1 (installed binary)
- Human proteome FASTA (for mapping the 2459 proteome-matching Casanovo peptides before scoring)

## Outputs

- NetMHCpan-4.1 binding predictions per peptide set
- Binder fraction per peptide category (e.g. 87%/86%/50% for Fig. 3c-d categories)
- Summary table such as immunopeptidomics_peptide_counts.csv with reproduced counts and binder fractions

## How to apply

First assemble the distinct peptide categories to compare — in the MDA-MB-231 immunopeptidomics comparison these were the 26,377 unique Casanovo de novo peptides, the 2459 Casanovo peptides matching the human proteome, and the 1544 unique peptides from a non-enzymatic Tide search post-processed with Percolator at 1% FDR. Run NetMHCpan-4.1 on each peptide set to obtain predicted MHC class I binding scores, then compute the binder fraction (fraction of peptides predicted as binders) within each category. Compare the fractions across categories as an orthogonal biological check on identification quality; the reference analysis reproduced 87%/86%/50% binder fractions across the three peptide categories reported in Fig. 3c-d. Record the counts and binder fractions in a table (e.g. immunopeptidomics_peptide_counts.csv) so the identification overlap and binder statistics can be verified together.

## Related tools

- **NetMHCpan-4.1** (Predicts MHC class I binding for each peptide set; binder fractions are computed from its predictions)
- **Casanovo** (Non-enzymatic fine-tuned model producing the de novo peptide predictions (mzTab exports) that are one scored category) — https://github.com/Noble-Lab/casanovo
- **Tide** (Non-specific-digest database search whose Percolator-processed 1% FDR peptides form the comparison category)

## Evaluation signals

- Binder fractions fall in [0, 1] and are computed over the full deduplicated peptide set for each category
- Reference values reproduce: 87%/86%/50% binder fractions across the three peptide categories of Fig. 3c-d
- Upstream counts verify before scoring: 26,377 unique Casanovo peptides, 2459 human-proteome matches, 1544 unique Tide peptides at 1% FDR
- Each peptide category is scored with the same NetMHCpan-4.1 settings so fractions are directly comparable
- Reproduced counts and binder fractions assemble cleanly into immunopeptidomics_peptide_counts.csv without missing categories

## Limitations

- De novo peptides outside the human proteome (e.g. the majority of the 26,377 Casanovo predictions) cannot be anchored to known proteins, so binder prediction is the main biological evidence available for them
- A majority-overlap comparison depends on the FDR threshold chosen for the database search (1% FDR via Percolator here); changing it changes the overlap and scored peptide set
- The field lacks a rigorous confidence-estimation method for de novo sequencing, so binder-fraction agreement is a validation proxy rather than a per-peptide confidence measure

## Evidence

- [abstract] fine-tuned non-enzymatic Casanovo for immunopeptidomics: "We also develop a version of Casanovo that is fine-tuned for non-enzymatic peptides"
- [other] task workflow defining the NetMHCpan-4.1 step and reference binder fractions: [evidence span withheld — anchor status: not_found]
- [other] source-card finding fixing the peptide counts per category: [evidence span withheld — anchor status: not_found]
- [readme] README scope statement for the peptide-set origin: "Casanovo can be used to find unexpected peptide sequences in any data-dependent acquisition, bottom-up tandem mass spectrometry dataset, and is particularly useful for immunopeptidomics"

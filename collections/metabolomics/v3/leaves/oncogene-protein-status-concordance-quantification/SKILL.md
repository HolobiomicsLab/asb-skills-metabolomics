---
name: oncogene-protein-status-concordance-quantification
description: Use when SNP-array copy-number calls (e.g. 10q/PTEN loss, 8q/MYC gain)
  must be cross-tabulated against protein-level statuses (phosphoAKT1, MYC from immunoblot
  densitometry) to compute per-subgroup concordance and residual discordance fractions.
when_to_use_negative:
- The cohort lacks both SNP-array copy-number calls and immunoblot-derived protein-status
  subgroups, so no concordance cross-tabulation is possible
- The goal is pathway-level metabolomics interpretation (e.g. KEGG single-sample GSEA
  enrichment of glycolysis or lipid-metabolism sets) rather than genomic–proteomic
  concordance accounting
- Protein statuses are continuous immunoblot densitometry values that have not yet
  been discretized into subgroups via k-means clustering
tools:
- name: Single Nucleotide Polymorphisms (SNP) arrays
  role: Source of the genomic-alteration calls (10q/PTEN loss, 8q/MYC gain) that are
    cross-tabulated against protein status
- name: Immunoblotting
  role: Produces the phosphoAKT1 and MYC protein measurements whose high/low discretized
    states define the subgroup denominators
- name: K-means clustering
  role: Segregates phosphoAKT1/MYC densitometric values into the 4 subgroups whose
    tallies (6/21/9/24) supply the denominators for concordance fractions
- name: pandas
  role: Holds the concordance counts, subgroup denominators, and cohort-level alteration
    frequencies as a data frame and emits the CSV output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:556bdd05469f3e1cfee02da75fb3589c8e5ff51798891f77b55180f013ed752a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Immunoblotting
  - K-means clustering
  - Single Nucleotide Polymorphisms (SNP) arrays
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# oncogene-protein-status-concordance-quantification

## Summary

Quantifies how often SNP-array copy-number alterations (10q/PTEN loss, 8q/MYC gain) coincide with high oncoprotein status (phosphoAKT1, MYC by immunoblotting) in a tumor cohort, reporting concordance fractions and residual-discordance fractions. It exposes whether genomic alterations account for protein activation, which in the Priolo et al. 2014 prostate cohort they did only partially (26% and 13%).

## When to use

Use when SNP-array copy-number calls (e.g. 10q/PTEN loss, 8q/MYC gain) must be cross-tabulated against protein-level statuses (phosphoAKT1, MYC from immunoblot densitometry) to compute per-subgroup concordance and residual discordance fractions.

## When NOT to use

- The cohort lacks both SNP-array copy-number calls and immunoblot-derived protein-status subgroups, so no concordance cross-tabulation is possible
- The goal is pathway-level metabolomics interpretation (e.g. KEGG single-sample GSEA enrichment of glycolysis or lipid-metabolism sets) rather than genomic–proteomic concordance accounting
- Protein statuses are continuous immunoblot densitometry values that have not yet been discretized into subgroups via k-means clustering

## Inputs

- Concordance counts from Results text (altered cases per protein-status subgroup: 7/27 phosphoAKT1-high; 2/15 MYC-high)
- K-means subgroup tallies of 60 T2 prostate tumors (phosphoAKT1-high/MYC-high 6, phosphoAKT1-high/MYC-low 21, phosphoAKT1-low/MYC-high 9, phosphoAKT1-low/MYC-low 24)
- Cohort-level SNP-array alteration frequencies (20% 10q loss, 18% 8q gain, 3% PTEN-loss/MYC-gain co-occurrence)

## Outputs

- snp_ihc_concordance_table.csv with columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance, and context rows for cohort-level alteration frequencies
- Recomputed concordance fractions (26% phosphoAKT1-high, 13% MYC-high) and residual-discordance fractions (≈0.74, ≈0.87)

## How to apply

Transcribe the stated concordance counts from the Results text into a pandas data frame — here 7/27 phosphoAKT1-high tumors and 2/15 MYC-high tumors carrying 10q/PTEN loss or 8q/MYC gain — along with cohort-level alteration frequencies (20% 10q loss, 18% 8q gain, 3% PTEN-loss/MYC-gain co-occurrence). Reconcile subgroup denominators against the k-means tallies (6 phosphoAKT1-high/MYC-high + 21 phosphoAKT1-high/MYC-low = 27 phosphoAKT1-high; 6 + 9 = 15 MYC-high of 60 T2 tumors) to catch transcription errors. Compute concordance fractions as altered_cases/subgroup_total (7/27 ≈ 25.9%, reported as 26%; 2/15 ≈ 13.3%, reported as 13%) using integer-percent rounding, and derive residual-discordance fractions as 1 − concordance (20/27 ≈ 0.74 for phosphoAKT1-high; 13/15 ≈ 0.87 for MYC-high). Write the result as snp_ihc_concordance_table.csv with columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance, plus context rows for the cohort-level alteration frequencies. The rationale is that discordance between genomics and protein status flags post-copy-number activation mechanisms that copy-number data alone cannot explain.

## Related tools

- **Single Nucleotide Polymorphisms (SNP) arrays** (Source of the genomic-alteration calls (10q/PTEN loss, 8q/MYC gain) that are cross-tabulated against protein status)
- **Immunoblotting** (Produces the phosphoAKT1 and MYC protein measurements whose high/low discretized states define the subgroup denominators)
- **K-means clustering** (Segregates phosphoAKT1/MYC densitometric values into the 4 subgroups whose tallies (6/21/9/24) supply the denominators for concordance fractions)
- **pandas** (Holds the concordance counts, subgroup denominators, and cohort-level alteration frequencies as a data frame and emits the CSV output)

## Examples

```
python -c "print(round(7/27*100), round(2/15*100), round(1-7/27,2), round(1-2/15,2))"
```

## Evaluation signals

- Recomputed concordance fractions equal the reported values on integer-percent rounding: 7/27 → 26% and 2/15 → 13%
- Each subgroup denominator reconciles with the k-means tallies: 6+21 = 27 phosphoAKT1-high and 6+9 = 15 MYC-high within 60 T2 tumors
- Residual discordance equals 1 − concordance exactly: 20/27 ≈ 0.74 and 13/15 ≈ 0.87
- Output CSV contains all required columns (subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance) and context rows for 20% 10q loss, 18% 8q gain, and 3% PTEN-loss/MYC-gain co-occurrence
- altered_cases ≤ subgroup_total for every row and subgroup totals sum consistently with the 60-tumor cohort

## Limitations

- Genomic alterations explain only a minority of protein activation cases (26% of phosphoAKT1-high, 13% of MYC-high), so high concordance should not be expected and residual discordance may reflect post-copy-number activation mechanisms
- Concordance estimates rest on a single 60-tumor T2 cohort (22 high Gleason, 38 low Gleason); small altered-case counts (7 and 2) make fractions sensitive to a single case
- The AKT1/MYC status was independent of Gleason grade and pathologic staging, so concordance fractions should not be interpreted as grade- or stage-associated

## Evidence

- [results] Genomic alterations accounted for only 26% (7/27) of phosphoAKT1-high tumors and 13% (2/15) of MYC-high tumors: [evidence span withheld — anchor status: not_found]

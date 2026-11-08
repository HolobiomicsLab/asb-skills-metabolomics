---
name: tumor-subgroup-denominator-reconciliation
description: Use when an analysis needs to test whether genomic alteration calls (e.g.
  10q/PTEN loss, 8q/MYC gain) predict or account for protein-level oncoprotein status
  (phosphoAKT1, MYC) in a cohort of tumors, requiring subgroup denominators to be
  reconciled before concordance fractions are reported.
when_to_use_negative:
- When the analysis goal is metabolomic pathway enrichment (single-sample GSEA over
  >50 KEGG metabolite sets) rather than genotype–protein-status concordance counting
- When subgroup denominators are derived from a clustering method other than k-means
  on phosphoAKT1/MYC immunoblot densitometric values — the reconciliation step (6+21=27,
  6+9=15) is specific to these reported tallies and will not transfer
- When cohort sizes or subgroup totals differ from the reported 60 T2 tumors; the
  integer-percent validation (26%, 13%) is cohort-specific
edam_operation: http://edamontology.org/operation_3659
tools:
- name: Single Nucleotide Polymorphisms (SNP) arrays
  role: Provide the genomic alteration calls (10q/PTEN loss, 8q/MYC gain) whose counts
    form the numerators of the concordance fractions
- name: Immunoblotting
  role: Generates the phosphoAKT1 and MYC expression measurements on human tumors
    whose densitometric values define the oncoprotein-high subgroups (denominators)
- name: K-means clustering
  role: Segregates the 60 T2 tumors into the 4 phosphoAKT1/MYC subgroups whose tallies
    (6, 21, 9, 24) are reconciled to set subgroup denominators of 27 and 15
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:e11e838525fd2af39027e2759ddfa2f5c51d68b24f0f81cc06002433adfacfc0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Immunoblotting
  - K-means clustering
  - Single Nucleotide Polymorphisms (SNP) arrays
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tumor-subgroup-denominator-reconciliation

## Summary

A cross-modality concordance skill that quantifies how often SNP-array genomic alterations (10q/PTEN loss, 8q/MYC gain) account for oncoprotein-high tumor subgroups, and derives the residual discordance left unexplained. It reconciles subgroup tallies from k-means clustering of phosphoAKT1/MYC immunoblot densitometry against alteration call counts before computing fractions.

## When to use

Use when an analysis needs to test whether genomic alteration calls (e.g. 10q/PTEN loss, 8q/MYC gain) predict or account for protein-level oncoprotein status (phosphoAKT1, MYC) in a cohort of tumors, requiring subgroup denominators to be reconciled before concordance fractions are reported.

## When NOT to use

- When the analysis goal is metabolomic pathway enrichment (single-sample GSEA over >50 KEGG metabolite sets) rather than genotype–protein-status concordance counting
- When subgroup denominators are derived from a clustering method other than k-means on phosphoAKT1/MYC immunoblot densitometric values — the reconciliation step (6+21=27, 6+9=15) is specific to these reported tallies and will not transfer
- When cohort sizes or subgroup totals differ from the reported 60 T2 tumors; the integer-percent validation (26%, 13%) is cohort-specific

## Inputs

- Concordance counts from Results text (e.g. 7/27 phosphoAKT1-high; 2/15 MYC-high tumors carrying 10q/PTEN loss or 8q/MYC gain)
- K-means subgroup tallies of phosphoAKT1/MYC densitometric subgroups over 60 T2 prostate tumors (6/21/9/24 per phosphoAKT1-high/MYC-high, phosphoAKT1-high/MYC-low, phosphoAKT1-low/MYC-high, phosphoAKT1-low/MYC-low)
- Cohort-level SNP-array alteration frequencies (10q loss 20%, 8q gain 18%, co-occurrence 3%)

## Outputs

- snp_ihc_concordance_table.csv with columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance
- Context rows recording cohort-level alteration frequencies (10q loss 20%, 8q gain 18%, PTEN-loss/MYC-gain co-occurrence 3%)

## How to apply

First transcribe the stated concordance counts and cohort-level alteration frequencies (20% 10q loss, 18% 8q gain, 3% PTEN-loss/MYC-gain co-occurrence) into a pandas data frame. Second, reconcile denominators against the k-means subgroup tallies: 6 phosphoAKT1-high/MYC-high + 21 phosphoAKT1-high/MYC-low = 27 phosphoAKT1-high, and 6 + 9 = 15 MYC-high out of 60 T2 prostate tumors; abort or flag the analysis if these sums do not match the reported subgroup sizes. Third, compute concordance fractions as altered_cases/subgroup_total — 7/27 ≈ 25.9% (reported as 26%) for phosphoAKT1-high and 2/15 ≈ 13.3% (reported as 13%) for MYC-high — validating that integer-percent rounding reproduces the published values. Fourth, derive residual-discordance fractions as 1 − concordance (20/27 ≈ 0.74 and 13/15 ≈ 0.87). Finally, write snp_ihc_concordance_table.csv with columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance, plus context rows for the cohort-level alteration frequencies. The rationale is that genomic alterations explain only a minority of oncoprotein activation, so the residual fraction is the key interpretive output, not the concordance fraction alone.

## Related tools

- **Single Nucleotide Polymorphisms (SNP) arrays** (Provide the genomic alteration calls (10q/PTEN loss, 8q/MYC gain) whose counts form the numerators of the concordance fractions)
- **Immunoblotting** (Generates the phosphoAKT1 and MYC expression measurements on human tumors whose densitometric values define the oncoprotein-high subgroups (denominators))
- **K-means clustering** (Segregates the 60 T2 tumors into the 4 phosphoAKT1/MYC subgroups whose tallies (6, 21, 9, 24) are reconciled to set subgroup denominators of 27 and 15)

## Examples

```
import pandas as pd; df = pd.DataFrame({'subgroup':['phosphoAKT1-high','MYC-high'],'altered_cases':[7,2],'subgroup_total':[27,15]}); df['concordance_fraction']=df.altered_cases/df.subgroup_total; df['residual_discordance']=1-df.concordance_fraction; df.to_csv('snp_ihc_concordance_table.csv', index=False)
```

## Evaluation signals

- Recomputed concordance fractions equal the reported values on integer-percent rounding: 7/27 ≈ 26% and 2/15 ≈ 13%
- Each subgroup denominator sums to the reported totals: 6+21 = 27 phosphoAKT1-high and 6+9 = 15 MYC-high within 60 T2 tumors
- Residual-discordance fractions satisfy residual = 1 − concordance: 20/27 ≈ 0.74 and 13/15 ≈ 0.87
- snp_ihc_concordance_table.csv contains exactly the columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance
- Context rows for cohort-level frequencies match the stated 20% 10q loss, 18% 8q gain, and 3% co-occurrence

## Limitations

- Concordance is counted as binary presence/absence of alteration within oncoprotein-high subgroups; it does not model dosage or partial alteration
- Only a minority of oncoprotein activation is explained by these alterations (26% phosphoAKT1-high, 13% MYC-high), so the residual discordance dominates and should not be interpreted as discordance from measurement error alone
- Subgroup assignment depends on k-means clustering of phosphoAKT1/MYC densitometry on 60 T2 tumors; subgroup tallies are not associated with Gleason grade, so grade cannot be used as an external validation axis

## Evidence

- [intro] genomic alterations rationale: [evidence span withheld — anchor status: not_found]
- [results] concordance finding: [evidence span withheld — anchor status: not_found]

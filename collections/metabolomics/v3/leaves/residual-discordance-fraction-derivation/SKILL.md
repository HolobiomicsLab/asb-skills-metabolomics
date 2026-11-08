---
name: residual-discordance-fraction-derivation
description: Use when comparing binary molecular subgroups (e.g., phosphoAKT1-high
  or MYC-high from k-means clustering) against SNP-array alteration calls (10q/PTEN
  loss, 8q/MYC gain) and you need the fraction of altered cases within each subgroup
  denominator.
when_to_use_negative:
- Subgroup membership has not been defined (no k-means clustering of phosphoAKT1/MYC
  densitometric values and no validated denominator tallies) — computing residuals
  against unverified denominators produces spurious fractions
- Genomic alteration calls and protein-status measurements are not paired on the same
  tumor samples, so per-subgroup concordance counts cannot be reconciled with subgroup
  totals
- The question is continuous correlation between genomic copy number and protein level
  rather than binary altered/unexplained classification of pre-defined subgroups
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: SNP arrays
  role: Source of the 10q/PTEN loss and 8q/MYC gain alteration calls used as the altered_cases
    numerator
- name: Immunoblotting
  role: Measures phosphoAKT1 and MYC protein levels that define the phosphoAKT1-high
    and MYC-high subgroup denominators
- name: K-means clustering
  role: Segregates the 60 tumors into the 4 phosphoAKT1/MYC subgroups whose tallies
    (27 and 15) set the denominators
- name: pandas
  role: Holds the concordance counts, computes concordance_fraction and residual_discordance,
    and writes snp_ihc_concordance_table.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:45a6853f313273b1d45151edf896ce6dbe46fd533b3b202d4544422cff4e632d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Immunoblotting
  - K-means clustering
  - SNP arrays
  - Single Nucleotide Polymorphisms (SNP) arrays
  - pandas
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# residual-discordance-fraction-derivation

## Summary

Quantifies how much of an oncoprotein-active tumor subgroup is NOT explained by its canonical genomic alteration by computing 1 − concordance (e.g., 20/27 ≈ 0.74 residual discordance for phosphoAKT1-high tumors lacking 10q/PTEN loss). It exposes genomic-vs-protein status gaps that raw concordance percentages understate.

## When to use

Use when comparing binary molecular subgroups (e.g., phosphoAKT1-high or MYC-high from k-means clustering) against SNP-array alteration calls (10q/PTEN loss, 8q/MYC gain) and you need the fraction of altered cases within each subgroup denominator.

## When NOT to use

- Subgroup membership has not been defined (no k-means clustering of phosphoAKT1/MYC densitometric values and no validated denominator tallies) — computing residuals against unverified denominators produces spurious fractions
- Genomic alteration calls and protein-status measurements are not paired on the same tumor samples, so per-subgroup concordance counts cannot be reconciled with subgroup totals
- The question is continuous correlation between genomic copy number and protein level rather than binary altered/unexplained classification of pre-defined subgroups

## Inputs

- Reported concordance counts from Results text (altered cases per protein-high subgroup: 7/27 phosphoAKT1-high; 2/15 MYC-high)
- K-means subgroup tallies of 60 T2 prostate tumors (phosphoAKT1-high/MYC-high 6, phosphoAKT1-high/MYC-low 21, phosphoAKT1-low/MYC-high 9, phosphoAKT1-low/MYC-low 24)
- Cohort-level SNP-array alteration frequencies (20% 10q loss, 18% 8q gain, 3% co-occurrence)
- Immunoblot densitometric values for phosphoAKT1 and MYC used to define the subgroups

## Outputs

- snp_ihc_concordance_table.csv with columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance
- Cohort-level context rows for 10q/PTEN-loss and 8q/MYC-gain frequencies

## How to apply

First transcribe the concordance counts from the Results text (7/27 phosphoAKT1-high tumors with 10q/PTEN loss; 2/15 MYC-high tumors with 8q/MYC gain) into a pandas data frame, alongside cohort-level frequencies (20% 10q loss, 18% 8q gain, 3% PTEN-loss/MYC-gain co-occurrence). Reconcile each denominator against the k-means subgroup tallies — 6+21 = 27 phosphoAKT1-high and 6+9 = 15 MYC-high of the 60 T2 prostate tumors. Compute concordance fractions (7/27 ≈ 25.9%, reported as 26%; 2/15 ≈ 13.3%, reported as 13%), then derive residual discordance as 1 − concordance (≈0.74 and ≈0.87). Write the table as snp_ihc_concordance_table.csv with columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance, plus context rows for cohort alteration frequencies. The rationale is that protein activation explained by PTEN loss or MYC gain is a minority phenomenon, so the residual fraction quantifies alternative drivers of phosphoAKT1/MYC status.

## Related tools

- **SNP arrays** (Source of the 10q/PTEN loss and 8q/MYC gain alteration calls used as the altered_cases numerator)
- **Immunoblotting** (Measures phosphoAKT1 and MYC protein levels that define the phosphoAKT1-high and MYC-high subgroup denominators)
- **K-means clustering** (Segregates the 60 tumors into the 4 phosphoAKT1/MYC subgroups whose tallies (27 and 15) set the denominators)
- **pandas** (Holds the concordance counts, computes concordance_fraction and residual_discordance, and writes snp_ihc_concordance_table.csv)

## Examples

```
df = pd.DataFrame({'subgroup':['phosphoAKT1-high','MYC-high'],'altered_cases':[7,2],'subgroup_total':[27,15]}); df['residual_discordance']=1-df.altered_cases/df.subgroup_total; df.to_csv('snp_ihc_concordance_table.csv')
```

## Evaluation signals

- Recomputed concordance fractions match the reported values on integer-percent rounding: 7/27 ≈ 25.9% rounds to the reported 26%, and 2/15 ≈ 13.3% rounds to the reported 13%
- Each subgroup denominator reconciles with the k-means tallies: 6+21 = 27 for phosphoAKT1-high and 6+9 = 15 for MYC-high, summing within the 60-tumor T2 cohort
- Residual-discordance values equal 1 − concordance: 20/27 ≈ 0.74 for phosphoAKT1-high and 13/15 ≈ 0.87 for MYC-high, and concordance + residual = 1 for every row
- snp_ihc_concordance_table.csv contains exactly the columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance, with subgroup_total > 0 and altered_cases ≤ subgroup_total

## Limitations

- Subgroup counts are small (altered cases of 7 and 2), so the concordance and residual fractions carry wide, unstated uncertainty — no confidence intervals were computed in the source analysis
- Concordance is computed only for the two canonical alterations (10q/PTEN loss, 8q/MYC gain); other mechanisms of phosphoAKT1 or MYC activation are not enumerated, so the residual is an unexplained fraction, not a causal attribution
- The reported percentages (26%, 13%) are rounded from 25.9% and 13.3%; comparisons at finer precision will diverge from the published figures

## Evidence

- [results] concordance counts with reported fractions: [evidence span withheld — anchor status: not_found]
- [other] output schema and pipeline: [evidence span withheld — anchor status: not_found]

---
name: snp-array-genomic-alteration-tabulation
description: Use when SNP-array alteration calls (e.g. 10q/PTEN loss, 8q/MYC gain)
  must be reconciled against protein-level tumor subgroups (phosphoAKT1/MYC k-means
  classes) to compute per-subgroup concordance and residual discordance in a cohort.
when_to_use_negative:
- The input is raw SNP-array intensity/CN data not yet reduced to alteration calls
  (10q/PTEN loss, 8q/MYC gain) — segmentation and calling must happen first.
- 'Tumor subgroups have not been defined: this skill requires the k-means phosphoAKT1/MYC
  classification (6+21=27 phosphoAKT1-high; 6+9=15 MYC-high), not just raw densitometry.'
- The goal is metabolomic pathway enrichment (GSEA over KEGG metabolite sets) rather
  than genomic–proteomic concordance accounting.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: SNP arrays
  role: Source of per-tumor genomic alteration calls (10q/PTEN loss, 8q/MYC gain)
    that are tabulated against protein subgroups
- name: Immunoblotting
  role: Provides phosphorylated AKT1 and MYC expression levels used to define the
    high/low protein subgroups serving as denominators
- name: K-means clustering
  role: Segregates the 60 T2 tumors into 4 phosphoAKT1/MYC subgroups, whose tallies
    (6+21=27, 6+9=15) anchor the concordance denominators
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:5fec44a48d3d2b154fe37591033e7790643204e3520bd2e1dc28b71b030134ec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Immunoblotting
  - K-means clustering
  - SNP arrays
  - Single Nucleotide Polymorphisms (SNP) arrays
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# snp-array-genomic-alteration-tabulation

## Summary

Tabulates how often SNP-array genomic alterations (10q/PTEN loss, 8q/MYC gain) coincide with immunoblot-defined phosphoAKT1-high or MYC-high tumor subgroups, converting raw case counts into concordance fractions and residual-discordance fractions. It quantifies how much of oncoprotein activation the underlying copy-number changes can actually explain.

## When to use

Use when SNP-array alteration calls (e.g. 10q/PTEN loss, 8q/MYC gain) must be reconciled against protein-level tumor subgroups (phosphoAKT1/MYC k-means classes) to compute per-subgroup concordance and residual discordance in a cohort.

## When NOT to use

- The input is raw SNP-array intensity/CN data not yet reduced to alteration calls (10q/PTEN loss, 8q/MYC gain) — segmentation and calling must happen first.
- Tumor subgroups have not been defined: this skill requires the k-means phosphoAKT1/MYC classification (6+21=27 phosphoAKT1-high; 6+9=15 MYC-high), not just raw densitometry.
- The goal is metabolomic pathway enrichment (GSEA over KEGG metabolite sets) rather than genomic–proteomic concordance accounting.

## Inputs

- SNP-array genomic alteration calls per tumor (10q/PTEN loss status, 8q/MYC gain status, co-occurrence)
- Immunoblot phosphoAKT1 and MYC densitometric values with k-means subgroup assignments (phosphoAKT1-high/low, MYC-high/low) for the 60 T2 prostate tumors
- Reported concordance counts from the Results text (7/27 and 2/15)

## Outputs

- snp_ihc_concordance_table.csv with columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance
- Context rows recording cohort-level frequencies: 20% 10q loss, 18% 8q gain, 3% PTEN-loss/MYC-gain co-occurrence
- Concordance fractions (26% phosphoAKT1-high, 13% MYC-high) and residual-discordance fractions (0.74, 0.87)

## How to apply

Transcribe the stated concordance counts from the Results text — 7/27 phosphoAKT1-high tumors and 2/15 MYC-high tumors carrying 10q/PTEN loss or 8q/MYC gain — into a pandas data frame with columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance. Reconcile denominators against the k-means subgroup tallies (6+21=27 phosphoAKT1-high; 6+9=15 MYC-high of 60 T2 tumors) and record cohort-level alteration frequencies (20% 10q loss, 18% 8q gain, 3% co-occurrence) as context rows. Compute concordance fractions as altered_cases/subgroup_total (7/27 = 25.9% reported as 26%; 2/15 = 13.3% reported as 13%) and residual discordance as 1 − concordance (20/27 = 0.74; 13/15 = 0.87). The rationale is to test whether the most frequent prostate-cancer copy-number lesions account for the oncoprotein statuses defined by immunoblotting and k-means clustering of densitometric values; write the result as snp_ihc_concordance_table.csv.

## Related tools

- **SNP arrays** (Source of per-tumor genomic alteration calls (10q/PTEN loss, 8q/MYC gain) that are tabulated against protein subgroups)
- **Immunoblotting** (Provides phosphorylated AKT1 and MYC expression levels used to define the high/low protein subgroups serving as denominators)
- **K-means clustering** (Segregates the 60 T2 tumors into 4 phosphoAKT1/MYC subgroups, whose tallies (6+21=27, 6+9=15) anchor the concordance denominators)

## Examples

```
python -c "import pandas as pd; df=pd.DataFrame([{'subgroup':'phosphoAKT1-high','altered_cases':7,'subgroup_total':27},{'subgroup':'MYC-high','altered_cases':2,'subgroup_total':15}]); df['concordance_fraction']=df.altered_cases/df.subgroup_total; df['residual_discordance']=1-df.concordance_fraction; df.to_csv('snp_ihc_concordance_table.csv',index=False)"
```

## Evaluation signals

- Recomputed concordance fractions equal the reported values on integer-percent rounding: 7/27 ≈ 25.9% → 26% and 2/15 ≈ 13.3% → 13%.
- Each subgroup denominator sums to the reported totals: phosphoAKT1-high = 6+21 = 27 and MYC-high = 6+9 = 15 of 60 T2 tumors.
- Residual-discordance column equals 1 − concordance_fraction (20/27 ≈ 0.74; 13/15 ≈ 0.87) for every row.
- Output CSV contains exactly the schema columns subgroup, altered_cases, subgroup_total, concordance_fraction, residual_discordance, plus context rows for the cohort-level frequencies (20% 10q loss, 18% 8q gain, 3% co-occurrence).
- Interpretive check: genomic alterations account for a minority of oncoprotein-high cases (26% and 13%), so most discordant cases lack 10q/PTEN loss or 8q/MYC gain.

## Limitations

- Genomic alterations explain only a minority of oncoprotein activation: 26% (7/27) of phosphoAKT1-high and 13% (2/15) of MYC-high tumors carry 10q/PTEN loss or 8q/MYC gain, so most phosphoAKT1/MYC-high tumors lack the corresponding copy-number lesion.
- The PTEN-loss/MYC-gain co-occurrence is rare in this cohort (3%), limiting power to assess combined genotype–protein concordance.
- Concordance is computed on a single 60-tumor T2 cohort with small subgroup sizes (6–24 tumors per class), so fractions are sensitive to single-case changes.

## Evidence

- [results] Genomic alterations accounted for only 26% (7/27) of phosphoAKT1-high tumors and 13% (2/15) of MYC-high tumors: [evidence span withheld — anchor status: not_found]
- [other] Cohort alteration frequencies: [evidence span withheld — anchor status: not_found]

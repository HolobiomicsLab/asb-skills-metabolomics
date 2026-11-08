---
name: lipidomic-data-matrix-reshaping
description: Use when an uploaded lipidomic matrix is at molecular-species level (rows
  like PC, PE, TG species) but the target analysis is at the lipid class (LCL) or
  building block (LBB) structural level.
when_to_use_negative:
- The input matrix is already at the lipid-class (LCL) level — aggregation would double-count
  rows that are already class sums
- The matrix contains duplicate lipid names or all-zero/zero-variance rows within
  a group — LipidOne 2.0 rejects the upload with an error listing the offending names,
  so clean the data first
- Row names do not follow LIPID MAPS shorthand notation, so class prefixes cannot
  be reliably extracted
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: LipidOne 2.0
  role: Web platform that performs the class/species/building-block sub-matrix generation
    and upload validation automatically
  repo: https://lipidone.eu
- name: R
  role: Back-end language of LipidOne 2.0 and suitable for scripted reconstruction
    of the aggregation (pandas/R load, group, sum)
- name: Lipid One (desktop)
  role: Standalone Perl/Windows predecessor covering building-block analysis of lipidomic
    datasets
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:13aae9b21745e9a8b6922ea879db085190695b4e6739877351ff951e8c069467
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lipid One (desktop)
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lipidomic-data-matrix-reshaping

## Summary

Reshapes a molecular-species-level lipidomic data matrix into lipid-class (LCL) or building-block (LBB) sub-matrices by aggregating rows that share a LIPID MAPS shorthand class prefix, sample-by-sample, so that class- and block-level analyses can run without manual submatrix calculation.

## When to use

Use when an uploaded lipidomic matrix is at molecular-species level (rows like PC, PE, TG species) but the target analysis is at the lipid class (LCL) or building block (LBB) structural level.

## When NOT to use

- The input matrix is already at the lipid-class (LCL) level — aggregation would double-count rows that are already class sums
- The matrix contains duplicate lipid names or all-zero/zero-variance rows within a group — LipidOne 2.0 rejects the upload with an error listing the offending names, so clean the data first
- Row names do not follow LIPID MAPS shorthand notation, so class prefixes cannot be reliably extracted

## Inputs

- Molecular-species-level lipidomic data matrix (CSV/TXT) with lipid shorthand notation row names and sample columns
- Experimental group labels for sample columns (required label fields, valid group assignment per sample)
- Example datasets such as ERYTHROCYTES, FUNGUS, SCIATIC NERVE from LipidOne 2.0

## Outputs

- LCL sub-matrix (lcl_submatrix.csv): one row per lipid class, same sample columns, values = per-sample sums of constituent molecular species
- Structured validation report (validation_report.json) with per-check error lists for duplicates, zero/constant rows, and label violations

## How to apply

Load the CSV/TXT lipidomic matrix with lipid names as the row index and samples as columns. First validate the upload: scan the lipid-name column for names appearing more than once, flag rows that are all-zero or zero-variance within an experimental group, and check that group labels are present and well-formed; emit issues as a structured report (JSON with per-check error lists) rather than accepting the data. If validation passes, parse each row's LIPID MAPS shorthand notation to extract the lipid class prefix (e.g., PC, PE, TG), group rows by that label, and for each sample column sum the abundance values of all molecular species in the same class, yielding one aggregated value per class per sample. Assemble these into a new matrix with one row per lipid class and the same sample columns (the LCL sub-matrix), e.g. lcl_submatrix.csv. Rationale: the article notes that per-level statistical or metabolic analyses otherwise require prior time- and labor-intensive submatrix calculations, which LipidOne 2.0 automates across its three structural levels.

## Related tools

- **LipidOne 2.0** (Web platform that performs the class/species/building-block sub-matrix generation and upload validation automatically) — https://lipidone.eu
- **R** (Back-end language of LipidOne 2.0 and suitable for scripted reconstruction of the aggregation (pandas/R load, group, sum))
- **Lipid One (desktop)** (Standalone Perl/Windows predecessor covering building-block analysis of lipidomic datasets) — github.com/matteogiulietti/LipidOne

## Examples

```
df = pd.read_csv('lipid_matrix.csv', index_col=0); lcl = df.groupby(df.index.str.extract(r'^([A-Za-z0-9]+)', expand=False)).sum(); lcl.to_csv('lcl_submatrix.csv')
```

## Evaluation signals

- Output matrix has exactly one row per distinct lipid class present in the input
- Each aggregated class value equals the sum of its constituent molecular-species values for every sample column
- lcl_submatrix.csv exists and parses as a well-formed matrix with the same sample columns as the input
- Validation report contains one section per check (duplicates, zero/constant rows, label violations) and every flagged entry names the offending lipid name or sample/group label
- Duplicate lipid names are reported explicitly rather than silently accepted

## Limitations

- Duplicate lipid names cause upload rejection rather than automatic correction — the duplicated names must be resolved by the user
- The desktop LipidOne predecessor is Windows 64-bit only, written in Perl, and requires GNUPLOT installed separately
- Example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE) are only accessible through the web platform rather than a persistent repository accession

## Evidence

- [abstract] three structural levels: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [intro] prior submatrix calculations: [evidence span withheld — anchor status: not_found]
- [discussion] example datasets access: "freely available for download on the designated page accessible by selecting the “START NOW” button and then “Load Data Matrix.”"
- [readme] desktop tool scope: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."

---
name: lipid-structural-level-mapping-lcl-lms-lbb
description: Use when an uploaded lipidomic data matrix is annotated at molecular-species
  level (e.g., PC 34:1, TG 52:2 shorthand) and statistical or pathway analyses must
  be run per lipid class, per molecular species, or per acyl/alkyl building block.
when_to_use_negative:
- The input matrix is already aggregated at the lipid-class level — re-summing would
  double-count class contributions
- Row labels are not in LIPID MAPS shorthand notation, so class prefixes or chain
  compositions cannot be parsed reliably
- Only polar-metabolite (non-lipid) data is being analyzed — the article stresses
  lipidomics requires lipid-specific analytical approaches distinct from general metabolomics
  platforms
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3353
- http://edamontology.org/topic_0153
tools:
- name: Lipid One 2.0
  role: Web platform that automates the class/species/building-block submatrix generation
    and hosts the level-tailored statistical analyses
  repo: https://lipidone.eu
- name: R
  role: Alternative scripting environment for reproducing the per-sample lipid-class
    aggregation (group-by class prefix, sum per sample column)
- name: Lipid Maps database
  role: Reference classification and shorthand notation standard used to parse lipid
    class prefixes and structural levels from row labels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:0f2aa4ecd80cf53c6b3bfc01f1f02ffb9ce8de28258d03ea75e0d86ab295b8a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lipid Maps database
  - Lipid One 2.0
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3353
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# lipid-structural-level-mapping-lcl-lms-lbb

## Summary

Maps a molecular-species-level lipidomic data matrix onto LipidOne 2.0's three structural levels — lipid classes (LCL), molecular species (LMS), and lipid building blocks (LBB: acyl, alkyl, or alkene chains) — by parsing each row's LIPID MAPS shorthand notation so class-, species-, and block-level analyses no longer require manual submatrix construction.

## When to use

Use when an uploaded lipidomic data matrix is annotated at molecular-species level (e.g., PC 34:1, TG 52:2 shorthand) and statistical or pathway analyses must be run per lipid class, per molecular species, or per acyl/alkyl building block.

## When NOT to use

- The input matrix is already aggregated at the lipid-class level — re-summing would double-count class contributions
- Row labels are not in LIPID MAPS shorthand notation, so class prefixes or chain compositions cannot be parsed reliably
- Only polar-metabolite (non-lipid) data is being analyzed — the article stresses lipidomics requires lipid-specific analytical approaches distinct from general metabolomics platforms

## Inputs

- Molecular-species-level lipidomic data matrix with LIPID MAPS shorthand notation row labels (e.g., PC 34:1, TG 52:2) and sample columns of abundance values
- LipidOne 2.0 example dataset (ERYTHROCYTES, FUNGUS, or SCIATIC NERVE) downloaded via the 'START NOW' button and 'Load Data Matrix' page

## Outputs

- LCL sub-matrix: one aggregated row per distinct lipid class, same sample columns (e.g., lcl_submatrix.csv)
- LMS sub-matrix: the original molecular-species-level matrix
- LBB sub-matrix: rows per lipid building block (acyl, alkyl, or alkene chain) derived from species shorthand decomposition

## How to apply

Load the molecular-species-level matrix (e.g., a LipidOne 2.0 example dataset such as ERYTHROCYTES, FUNGUS, or SCIATIC NERVE) and parse each row's lipid shorthand notation to extract the lipid class prefix (PC, PE, TG) per LIPID MAPS conventions. To generate the LCL sub-matrix, group rows by the extracted class label and, for each sample column, sum the abundance values of all molecular species belonging to the same lipid class, yielding one aggregated row per class; write the result to lcl_submatrix.csv. The LMS level uses the input matrix as-is, while the LBB level requires decomposing each species shorthand into its constituent acyl/alkyl/alkene chains. LipidOne 2.0 automates these submatrix calculations, which the article describes as time- and labor-intensive when done by hand, and lets users include or exclude experimental groups and lipid classes at any stage before running the level-tailored mono- and multivariate statistics.

## Related tools

- **Lipid One 2.0** (Web platform that automates the class/species/building-block submatrix generation and hosts the level-tailored statistical analyses) — https://lipidone.eu
- **R** (Alternative scripting environment for reproducing the per-sample lipid-class aggregation (group-by class prefix, sum per sample column))
- **Lipid Maps database** (Reference classification and shorthand notation standard used to parse lipid class prefixes and structural levels from row labels)

## Examples

```
df.groupby(df.index.str.extract(r'^(PC|PE|TG|...)_')) [col].sum().to_csv('lcl_submatrix.csv')
```

## Evaluation signals

- Output LCL matrix has exactly one row per distinct lipid class present in the input shorthand labels
- For every sample column, each aggregated class value equals the sum of its constituent molecular-species abundance values
- lcl_submatrix.csv exists and parses as a well-formed matrix with the same sample columns as the input
- LBB decomposition reproduces the correct number of chains (acyl/alkyl/alkene) per species shorthand, consistent with the LIPID MAPS notation
- Row/column ordering and group inclusion/exclusion flags are preserved so downstream level-tailored statistics can be re-run reproducibly

## Limitations

- The desktop Lipid One precursor (Perl, Windows 64-bit only, GNU Plot/Prima GUI) is Windows-only and focused on building blocks; Lipid One 2.0 is web-only with no stated Docker/conda/source deployment and no stated software license
- Example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE) are only accessible through the web platform rather than a persistent repository accession
- Class-level aggregation by summation assumes abundance values are on an additive scale; contributions of species in the same class must all be present to avoid underestimating class totals

## Evidence

- [abstract] abstract: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [intro] intro: [evidence span withheld — anchor status: not_found]
- [discussion] discussion: [evidence span withheld — anchor status: not_found]
- [readme] readme: "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."

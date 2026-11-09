---
name: lipid-class-aggregation-from-molecular-species
description: Use when the input is a molecular-species-level lipidomic data matrix
  (rows = lipid species in LIPID MAPS shorthand notation, columns = samples) and a
  lipid-class-level (LCL) matrix is needed for downstream class-level statistics.
when_to_use_negative:
- The input matrix is already at the lipid-class level (one row per class) — aggregation
  would double-count nothing but adds no information and class-level statistics can
  proceed directly.
- The target is building-block (LBB) level analysis of acyl/alkyl/alkenyl chains —
  that requires a different decomposition of species into building blocks, not class-prefix
  summation.
- Lipid shorthand notation does not follow LIPID MAPS conventions (e.g., free-text
  or proprietary names), so class prefixes cannot be reliably extracted without a
  prior nomenclature mapping step (see the article's Support Protocol on lipid nomenclature).
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LipidOne 2.0
  role: Web tool whose R back-end performs the per-sample lipid-class aggregation
    to produce the LCL sub-matrix during dataset upload/mining
  repo: https://lipidone.eu
- name: R
  role: Language of the LipidOne 2.0 back-end scripts; used to parse shorthand notation,
    group species, and sum abundances per class
- name: LipidOne (desktop v1)
  role: Predecessor standalone Perl tool for building-block analysis; reference implementation
    context for the aggregation approach
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:e5eed55919dae626ebdf3f152a3430c591d3e4c6dc5ee1d45864a807d2885c10
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne (desktop v1)
  - LipidOne 2.0
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# lipid-class-aggregation-from-molecular-species

## Summary

Generates the lipid class (LCL) sub-matrix from a molecular-species-level lipidomic data matrix by summing, sample-by-sample, the abundances of all molecular species sharing the same LIPID MAPS shorthand lipid class prefix (e.g., PC, PE, TG). This is the aggregation step LipidOne 2.0 performs so users can analyze lipids at the class structural level without manual submatrix calculations.

## When to use

Use when the input is a molecular-species-level lipidomic data matrix (rows = lipid species in LIPID MAPS shorthand notation, columns = samples) and a lipid-class-level (LCL) matrix is needed for downstream class-level statistics.

## When NOT to use

- The input matrix is already at the lipid-class level (one row per class) — aggregation would double-count nothing but adds no information and class-level statistics can proceed directly.
- The target is building-block (LBB) level analysis of acyl/alkyl/alkenyl chains — that requires a different decomposition of species into building blocks, not class-prefix summation.
- Lipid shorthand notation does not follow LIPID MAPS conventions (e.g., free-text or proprietary names), so class prefixes cannot be reliably extracted without a prior nomenclature mapping step (see the article's Support Protocol on lipid nomenclature).

## Inputs

- Molecular-species-level lipidomic data matrix (CSV/tabular), rows in LIPID MAPS shorthand notation, columns = samples
- LipidOne 2.0 example datasets such as ERYTHROCYTES, FUNGUS, or SCIATIC NERVE (downloadable via the web tool's 'Load Data Matrix' page)

## Outputs

- lcl_submatrix.csv: lipid-class-level matrix with one aggregated row per lipid class and the same sample columns as the input

## How to apply

Load the species-level matrix (e.g., one of the LipidOne example datasets such as ERYTHROCYTES, FUNGUS, or SCIATIC NERVE) with pandas or R, and parse each row's lipid shorthand notation to extract the lipid class prefix (e.g., PC, PE, TG) according to LIPID MAPS shorthand notation. Group rows by the extracted class label, then for each sample column sum the abundance values of all molecular species belonging to the same class, yielding one aggregated value per class per sample. Assemble these into a new matrix with one row per lipid class and the same sample columns, and write it to lcl_submatrix.csv. The rationale (per the article) is that analyses at the class, molecular-species, and building-block levels each require a sub-matrix that is otherwise 'time- and labor-intensive' to compute manually; LipidOne 2.0's R back-end automates this so statistics can be run at each structural level.

## Related tools

- **LipidOne 2.0** (Web tool whose R back-end performs the per-sample lipid-class aggregation to produce the LCL sub-matrix during dataset upload/mining) — https://lipidone.eu
- **R** (Language of the LipidOne 2.0 back-end scripts; used to parse shorthand notation, group species, and sum abundances per class)
- **LipidOne (desktop v1)** (Predecessor standalone Perl tool for building-block analysis; reference implementation context for the aggregation approach) — github.com/matteogiulietti/LipidOne

## Examples

```
df<-read.csv("lipids.csv");df$class<-sub("\\(.*","",df$lipid);lcl<-aggregate(.~class,df[,-1],sum);write.csv(lcl,"lcl_submatrix.csv",row.names=FALSE)
```

## Evaluation signals

- The output lcl_submatrix.csv exists and parses as a well-formed matrix (readable by pandas/R without errors).
- The LCL matrix has exactly one row per distinct lipid class present in the input (row count equals number of unique class prefixes).
- Sample columns of the LCL matrix are identical to the sample columns of the input species-level matrix.
- For every sample, each aggregated class value equals the arithmetic sum of its constituent molecular-species values — spot-check, e.g., PC(34:1) + PC(36:2) equals the PC row value.
- No molecular species is lost or double-assigned: every input row maps to exactly one extracted class prefix.

## Limitations

- Class extraction depends on correctly parsing LIPID MAPS shorthand prefixes; malformed or non-standard lipid names will mis-group or drop species.
- LipidOne 2.0's example datasets are only accessible through the web platform ('START NOW' → 'Load Data Matrix'), not via a persistent repository accession.
- The article provides no installability/deployment information (Docker, conda, source install) for the web tool, and no explicit license statement for LipidOne 2.0; the public GitHub README documents only the older standalone Perl/Windows desktop version (LipidOne v1, Gnuplot-dependent), not the 2.0 back-end.
- The aggregation uses simple summation of abundances; the article does not describe normalization or transformation applied during LCL construction.

## Evidence

- [abstract] It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains).: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks"
- [intro] Existing platforms treat lipids only at the molecular-species level, and analyses at lipid class (LCL), molecular species (LMS), or building block (LBB) levels require prior time- and labor-intensive submatrix calculations: [evidence span withheld — anchor status: not_found]
- [discussion] The three examples of lipidomic datasets “ERYTHROCYTES,” “FUNGUS,” and “SCIATIC NERVE” are freely available for download: "freely available for download on the designated page accessible by selecting the “START NOW” button and then “Load Data Matrix.”"
- [readme] README: LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset.: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."

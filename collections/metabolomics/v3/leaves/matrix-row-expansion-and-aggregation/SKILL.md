---
name: matrix-row-expansion-and-aggregation
description: Use when the input is a molecular-species (LMS) intensity matrix whose
  row identifiers are LIPID MAPS shorthand names (e.g. 'PC 16:0_18:1') and a chain-level
  (LBB) matrix is required for downstream statistics or pathway analysis.
when_to_use_negative:
- The matrix is already at the lipid class (LCL) or building-block (LBB) level and
  only needs direct statistical analysis — no expansion/aggregation is required.
- Row identifiers do not encode chain composition in LIPID MAPS shorthand notation,
  so chains cannot be parsed from the row names.
- You need to preserve molecular-species-level statistics unchanged; this skill intentionally
  redistributes each species' signal across its chains.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: LipidOne 2.0
  role: Web tool that performs the automatic LBB sub-matrix generation from the uploaded
    molecular-species data matrix; R scripts in its back-end implement the expansion
    and aggregation.
  repo: https://lipidone.eu
- name: R
  role: Implementation environment for parsing row identifiers, expanding species
    rows into chain rows, aggregating by pseudo-lipid header, and writing lbb_matrix.csv.
- name: LipidOne (desktop)
  role: Predecessor desktop (Perl/Windows) tool for building-block-level analysis
    that this web-based reconstruction supersedes.
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:508e17b2ce71d088378ed88b36b3f747b6c4e2970b7d27a52dd6c3e353afa86a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne (desktop)
  - LipidOne 2.0
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# matrix-row-expansion-and-aggregation

## Summary

Reconstructs the lipid building block (LBB) sub-matrix from an LMS-level molecular-species intensity matrix by splitting each species row into its constituent chain rows, dividing intensities by the chain count, and summing duplicate pseudo-lipid rows. It automates the otherwise time- and labor-intensive submatrix calculations needed for chain-level lipidomics analysis in LipidOne 2.0.

## When to use

Use when the input is a molecular-species (LMS) intensity matrix whose row identifiers are LIPID MAPS shorthand names (e.g. 'PC 16:0_18:1') and a chain-level (LBB) matrix is required for downstream statistics or pathway analysis.

## When NOT to use

- The matrix is already at the lipid class (LCL) or building-block (LBB) level and only needs direct statistical analysis — no expansion/aggregation is required.
- Row identifiers do not encode chain composition in LIPID MAPS shorthand notation, so chains cannot be parsed from the row names.
- You need to preserve molecular-species-level statistics unchanged; this skill intentionally redistributes each species' signal across its chains.

## Inputs

- LMS-level molecular lipid species intensity matrix (rows = species in LIPID MAPS shorthand notation such as 'PC 16:0_18:1', columns = samples)
- Lipid class-to-chain-count mapping (e.g. lyso-PC = 1, PC = 2, TG = 3, CL = 4)

## Outputs

- Aggregated chain-level LBB sub-matrix (pseudo-lipid rows such as 'PC_16:0' x original sample columns)
- lbb_matrix.csv file containing the LBB sub-matrix

## How to apply

Load the LMS-level intensity matrix in R and parse each row identifier into a lipid class prefix and a chain list ('PC 16:0_18:1' -> class 'PC', chains ['16:0','18:1']). For each class, determine the number of constituent chains — 1 for lyso-PC, 2 for PC, 3 for TG, 4 for CL — and split the species row into that many chain rows, dividing every sample intensity value by the chain count so total signal is conserved. Build the pseudo-lipid row header by joining the class prefix and the chain annotation with an underscore (e.g. 'PC_16:0', 'PC_18:1'). Identify rows with identical pseudo-lipid headers — including species whose chains appear in reversed order — and sum their values sample-by-sample into a single row. Assemble the aggregated chain-level matrix (pseudo-lipid rows x original sample columns) and write it to lbb_matrix.csv.

## Related tools

- **LipidOne 2.0** (Web tool that performs the automatic LBB sub-matrix generation from the uploaded molecular-species data matrix; R scripts in its back-end implement the expansion and aggregation.) — https://lipidone.eu
- **R** (Implementation environment for parsing row identifiers, expanding species rows into chain rows, aggregating by pseudo-lipid header, and writing lbb_matrix.csv.)
- **LipidOne (desktop)** (Predecessor desktop (Perl/Windows) tool for building-block-level analysis that this web-based reconstruction supersedes.) — github.com/matteogiulietti/LipidOne

## Examples

```
R> expanded <- do.call(rbind, lapply(split(species, rownames(species)), expand_chains)); lbb <- aggregate(. ~ pseudo_lipid, expanded, sum); write.csv(lbb, "lbb_matrix.csv")
```

## Evaluation signals

- Signal conservation: the sum of a species' expanded chain-row values (after division by the chain count) equals the original species row value in every sample.
- Each class expands to exactly its expected number of chain rows: 1 for lyso-PC, 2 for PC, 3 for TG, 4 for CL.
- Row headers of the output follow the pseudo-lipid format class_underscore_chain (e.g. 'PC_16:0'), with all duplicate headers (including reversed chain orders) collapsed to a single summed row.
- The number of columns is unchanged from the input matrix; only rows are expanded and re-aggregated.
- The final object is written as lbb_matrix.csv with pseudo-lipid rows x original sample columns.

## Limitations

- Row identifiers must follow LIPID MAPS shorthand notation with class prefix and underscore-separated chains; non-conforming names cannot be parsed.
- Species with the same chains in reversed order (e.g. 16:0_18:1 vs 18:1_16:0) produce identical pseudo-lipid headers and must be summed — if chain order is biologically meaningful (sn-position), this aggregation loses that information.
- Existing platforms treated this step as time- and labor-intensive manual calculation; errors in the chain-count mapping (lyso-PC/PC/TG/CL) will silently mis-scale intensities.
- The GitHub README describes the older desktop LipidOne (Perl, Windows 64-bit, Gnuplot), not LipidOne 2.0's web implementation of this step.

## Evidence

- [abstract] Three structural levels of lipids: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [intro] Need for prior submatrix calculations: [evidence span withheld — anchor status: not_found]
- [readme] Desktop LipidOne purpose: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."

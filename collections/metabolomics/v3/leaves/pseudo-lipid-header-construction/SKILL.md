---
name: pseudo-lipid-header-construction
description: Use when an LMS-level (molecular species) intensity matrix must be decomposed
  into chain-level lipid building block rows for downstream LBB-level statistics or
  pathway analysis, preserving total sample signal.
when_to_use_negative:
- The input matrix is already at the LBB (building block/chain) or LCL (lipid class)
  level and needs no species-to-chain decomposition.
- Row identifiers cannot be parsed into a lipid class prefix plus chain list (non-standard
  lipid shorthand notation), since chain count and pseudo-lipid headers depend on
  this parsing.
- The goal is class-level or molecular-species-level analysis, where splitting into
  chains would discard species-level information.
tools:
- name: LipidOne 2.0
  role: Web tool whose back-end R scripts perform the LBB sub-matrix generation; the
    skill reconstructs its pseudo-lipid expansion logic
  repo: https://lipidone.eu
- name: R
  role: Environment for implementing the parsing, chain splitting, division by chain
    count, and row-sum aggregation of the LBB matrix
- name: Lipid One
  role: Precursor desktop (Perl/Prima) tool that first addressed building-block analysis;
    its repository documents the standalone LipidOne software
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:a929ea0dfb2973fbc298096bbdcb57f76074c61b44f33cd3dafd823620be93f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lipid One
  - LipidOne 2.0
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pseudo-lipid-header-construction

## Summary

Reconstructs LipidOne 2.0's automated generation of the lipid building block (LBB) sub-matrix: each molecular-species row (e.g., 'PC 16:0_18:1') is split into chain-level pseudo-lipid rows with class-underscore-chain headers, intensities divided by chain count to conserve signal, and duplicate pseudo-lipid rows summed.

## When to use

Use when an LMS-level (molecular species) intensity matrix must be decomposed into chain-level lipid building block rows for downstream LBB-level statistics or pathway analysis, preserving total sample signal.

## When NOT to use

- The input matrix is already at the LBB (building block/chain) or LCL (lipid class) level and needs no species-to-chain decomposition.
- Row identifiers cannot be parsed into a lipid class prefix plus chain list (non-standard lipid shorthand notation), since chain count and pseudo-lipid headers depend on this parsing.
- The goal is class-level or molecular-species-level analysis, where splitting into chains would discard species-level information.

## Inputs

- LMS-level molecular species intensity matrix (rows = lipid species such as 'PC 16:0_18:1', columns = samples), e.g. loaded in R
- Lipid class nomenclature mapping class prefix to number of chains (lyso-PC=1, PC=2, TG=3, CL=4)

## Outputs

- Aggregated chain-level LBB sub-matrix (pseudo-lipid rows such as 'PC_16:0' x original sample columns)
- lbb_matrix.csv file containing the pseudo-lipid matrix

## How to apply

Parse each LMS row identifier into a lipid class prefix and a chain list (e.g., 'PC 16:0_18:1' -> class 'PC', chains ['16:0','18:1']). Determine the number of constituent chains per class — 1 for lyso-PC, 2 for PC, 3 for TG, 4 for CL — and split each species row into that many chain rows, dividing every sample intensity value by the chain count so total signal is conserved. Build each pseudo-lipid header by joining the class prefix and the chain annotation with an underscore (e.g., 'PC_16:0', 'PC_18:1'). Rows with identical pseudo-lipid headers — arising when different species share a chain, including reversed chain orders — are summed sample-by-sample into a single row. Assemble the aggregated pseudo-lipid x sample matrix and write it to lbb_matrix.csv.

## Related tools

- **LipidOne 2.0** (Web tool whose back-end R scripts perform the LBB sub-matrix generation; the skill reconstructs its pseudo-lipid expansion logic) — https://lipidone.eu
- **R** (Environment for implementing the parsing, chain splitting, division by chain count, and row-sum aggregation of the LBB matrix)
- **Lipid One** (Precursor desktop (Perl/Prima) tool that first addressed building-block analysis; its repository documents the standalone LipidOne software) — github.com/matteogiulietti/LipidOne

## Examples

```
R> write.csv(lbb_matrix, file = "lbb_matrix.csv", row.names = TRUE)  # pseudo-lipid rows (e.g. PC_16:0) x original sample columns
```

## Evaluation signals

- Signal conservation: for every original species row, the sum of its expanded chain-row values (each divided by the chain count) equals the original sample intensity.
- Expanded row count per species matches the class chain count (1 for lyso-PC, 2 for PC, 3 for TG, 4 for CL).
- Every output row header is a pseudo-lipid of the form '<class>_<chain>' (e.g., 'PC_16:0', 'PC_18:1').
- No duplicate pseudo-lipid headers remain in the final matrix: rows sharing a header, including reversed chain-order variants, are merged by sample-wise summation.
- Sample columns of the LBB matrix correspond one-to-one with the original LMS matrix columns, and the output file lbb_matrix.csv is produced.

## Limitations

- Chain-count assignment is convention-based per class (e.g., 1 for lyso-PC, 4 for CL); species outside these conventions or with unparseable identifiers will be mis-expanded.
- Species sharing chains in reversed order (e.g., 'PC 16:0_18:1' vs 'PC 18:1_16:0') must be merged correctly or signal will be double-counted across pseudo-lipid rows.
- The LBB level requires prior sub-matrix calculations that the article characterizes as 'time- and labor-intensive' when done manually; the example datasets (ERYTHROCYTES, FUNGUS, SCIATIC NERVE) are only accessible through the web platform rather than a persistent repository accession.

## Evidence

- [abstract] three structural levels of lipids: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [intro] prior sub-matrix calculations burden: [evidence span withheld — anchor status: not_found]
- [readme] LipidOne building block purpose (README): "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."
- [readme] LipidOne desktop implementation (README): "LipidOne is a stand alone software entirely written in Perl and implemented for the Microsoft Windows operating system."

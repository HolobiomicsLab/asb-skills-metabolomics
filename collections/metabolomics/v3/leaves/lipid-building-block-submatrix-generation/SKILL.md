---
name: lipid-building-block-submatrix-generation
description: Use when an uploaded lipidomic data matrix is at the molecular-species
  level (e.g., 'PC 16:0_18:1') and a chain-level (lipid building block) sub-matrix
  is required for downstream analysis.
when_to_use_negative:
- The input matrix is already at the lipid class (LCL) level, so no chain decomposition
  is possible
- The input is already a chain-level/building-block matrix — re-splitting would divide
  values by chain counts a second time
- Row identifiers do not follow lipid shorthand notation with class prefix and colon-separated
  chain annotations, so chains cannot be parsed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidOne 2.0
  role: Web tool whose backend R scripts perform the automatic LBB sub-matrix generation
    from the uploaded molecular-species matrix
  repo: https://lipidone.eu
- name: R
  role: Language in which the LBB decomposition (parsing species IDs, splitting rows
    by chain count, aggregating duplicate pseudo-lipid rows) is implemented
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:f9386b0f5680c4b3872c36db253ccc64d3e047647761840ede698516e1a07a1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# lipid-building-block-submatrix-generation

## Summary

Automatically derives the lipid building block (LBB) sub-matrix from a molecular-species (LMS-level) intensity matrix by splitting each lipid species row into its constituent acyl/alkyl/alkenyl chain rows, conserving total signal, and aggregating chains shared across species into pseudo-lipid rows.

## When to use

Use when an uploaded lipidomic data matrix is at the molecular-species level (e.g., 'PC 16:0_18:1') and a chain-level (lipid building block) sub-matrix is required for downstream analysis.

## When NOT to use

- The input matrix is already at the lipid class (LCL) level, so no chain decomposition is possible
- The input is already a chain-level/building-block matrix — re-splitting would divide values by chain counts a second time
- Row identifiers do not follow lipid shorthand notation with class prefix and colon-separated chain annotations, so chains cannot be parsed

## Inputs

- Molecular-species (LMS-level) lipidomic intensity matrix with row identifiers in shorthand notation (e.g., 'PC 16:0_18:1') and one column per sample

## Outputs

- Aggregated chain-level LBB sub-matrix written to lbb_matrix.csv, with pseudo-lipid rows ('<class>_<chain>', e.g., 'PC_16:0') × original sample columns
- Pseudo-lipid row headers displaying the lipid class name together with the individual chain name

## How to apply

Load the LMS-level intensity matrix in R and parse each row identifier into a lipid class prefix plus a chain list (e.g., 'PC 16:0_18:1' → class 'PC', chains ['16:0','18:1']). For each class, determine the number of constituent chains — 1 for lyso-PC, 2 for PC, 3 for TG, 4 for CL — and split the species row into that many chain rows, dividing every sample intensity value by the chain count so total signal is conserved. Build each pseudo-lipid header by joining the class prefix and the chain annotation with an underscore (e.g., 'PC_16:0', 'PC_18:1'). Identify rows with identical pseudo-lipid headers (they arise when different molecular species share a chain, including reversed chain orders) and sum their values sample-by-sample into a single row. Assemble the aggregated chain-level matrix (pseudo-lipid rows × original sample columns) and write it to lbb_matrix.csv.

## Related tools

- **LipidOne 2.0** (Web tool whose backend R scripts perform the automatic LBB sub-matrix generation from the uploaded molecular-species matrix) — https://lipidone.eu
- **R** (Language in which the LBB decomposition (parsing species IDs, splitting rows by chain count, aggregating duplicate pseudo-lipid rows) is implemented) — github.com/matteogiulietti/LipidOne

## Evaluation signals

- Each species row is expanded into exactly its class-specific chain count: 1 row for lyso-PC, 2 for PC, 3 for TG, 4 for CL
- Signal conservation: for each sample, the sum of the expanded chain-row values equals the original species row value (values divided by 1, 2, 3, or 4 as appropriate)
- Row headers are pseudo-lipids in '<class>_<chain>' format (e.g., 'PC_16:0'), with chains shared across molecular species (including reversed chain orders) collapsed into a single summed row
- The output matrix has the same sample columns as the input, and is persisted as lbb_matrix.csv

## Limitations

- Signal is apportioned evenly across chains (divided by chain count), so identical chains shared by different species are aggregated by summation rather than resolved individually
- Parsing depends on lipid shorthand notation; species whose identifiers cannot be split into a class prefix and chain list cannot be decomposed

## Evidence

- [abstract] Three structural levels support: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [intro] Prior submatrix calculation burden: [evidence span withheld — anchor status: not_found]
- [readme] README purpose statement: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."

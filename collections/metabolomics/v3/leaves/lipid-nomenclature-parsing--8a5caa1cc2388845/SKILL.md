---
name: lipid-nomenclature-parsing
description: Use when an uploaded lipidomic data matrix is keyed by molecular-species
  identifiers (e.g. 'PC 16:0_18:1') and chain-level lipid building block sub-matrices
  must be generated for downstream class/species/building-block analyses.
when_to_use_negative:
- The analysis targets the lipid class (LCL) or molecular species (LMS) level only
  — no chain decomposition is needed and existing sub-matrices can be used directly
- Row identifiers lack parsable chain annotations (e.g. only class names or non-shorthand
  labels), so class prefix and chain list cannot be split reliably
- The matrix is already a chain-level/pseudo-lipid feature table, and re-splitting
  would double-divide intensities
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidOne 2.0
  role: Web tool whose upload pipeline performs the LBB sub-matrix generation from
    the uploaded molecular-species data matrix
  repo: https://lipidone.eu
- name: R
  role: Back-end scripting language in which the LBB matrix construction and pseudo-lipid
    aggregation are implemented
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:074fa450d8df34f888690ad23643a59b596cf3a1b8c0a4fc45e736ebc429cf43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidOne 2.0
  - R
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# lipid-nomenclature-parsing

## Summary

Derives the lipid building block (LBB) sub-matrix from a molecular-species (LMS) intensity matrix by parsing each species identifier into a lipid class prefix and chain list, splitting rows into per-chain pseudo-lipids with signal conservation, and aggregating duplicate chain rows.

## When to use

Use when an uploaded lipidomic data matrix is keyed by molecular-species identifiers (e.g. 'PC 16:0_18:1') and chain-level lipid building block sub-matrices must be generated for downstream class/species/building-block analyses.

## When NOT to use

- The analysis targets the lipid class (LCL) or molecular species (LMS) level only — no chain decomposition is needed and existing sub-matrices can be used directly
- Row identifiers lack parsable chain annotations (e.g. only class names or non-shorthand labels), so class prefix and chain list cannot be split reliably
- The matrix is already a chain-level/pseudo-lipid feature table, and re-splitting would double-divide intensities

## Inputs

- Molecular-species (LMS-level) lipidomic intensity matrix with row identifiers in shorthand notation (e.g. 'PC 16:0_18:1', 'TG 16:0_18:1_18:2') and one column per sample
- Lipid class-to-chain-count mapping (1 = lyso-PC, 2 = PC, 3 = TG, 4 = CL)

## Outputs

- LBB sub-matrix written to lbb_matrix.csv with pseudo-lipid row headers (e.g. 'PC_16:0') x original sample columns
- Row-per-chain expansion where each species intensity is divided by the lipid's chain count

## How to apply

Load the molecular-species (LMS-level) intensity matrix in R and parse each row identifier into a lipid class prefix and a chain list, e.g. 'PC 16:0_18:1' yields class 'PC' and chains ['16:0','18:1']. For each lipid class, determine the number of constituent chains (1 for lyso-PC, 2 for PC, 3 for TG, 4 for CL) and split the species row into that many chain rows, dividing every sample intensity value by the chain count so total signal is conserved across the expansion. Build each pseudo-lipid row header by joining the class prefix and chain annotation with an underscore (e.g. 'PC_16:0', 'PC_18:1'). Identify rows with identical pseudo-lipid headers — which arise when different molecular species share a chain, including reversed chain orders — and sum their values sample-by-sample into a single row. Assemble the aggregated chain-level matrix (pseudo-lipid rows x original sample columns) and write it out as lbb_matrix.csv.

## Related tools

- **LipidOne 2.0** (Web tool whose upload pipeline performs the LBB sub-matrix generation from the uploaded molecular-species data matrix) — https://lipidone.eu
- **R** (Back-end scripting language in which the LBB matrix construction and pseudo-lipid aggregation are implemented) — github.com/matteogiulietti/LipidOne

## Evaluation signals

- Column-sum invariant: after chain-count division and aggregation, each sample column sum in lbb_matrix.csv equals the corresponding LMS column sum (total signal conserved)
- Chain-count correctness: each class contributes exactly its expected number of chain rows (lyso-PC = 1, PC = 2, TG = 3, CL = 4)
- Header format check: every row header matches the pseudo-lipid pattern CLASS_chain with an underscore separator (e.g. 'PC_16:0')
- Aggregation completeness: no duplicate pseudo-lipid headers remain in the output, including species whose chain lists are reversed orderings of each other
- Row/column sanity: number of output rows equals the number of distinct class-chain combinations present in the input identifiers; columns unchanged from the input matrix

## Limitations

- Identifiers with reversed chain orders (e.g. 'PC 18:1_16:0' vs 'PC 16:0_18:1') must be recognized and merged, otherwise duplicate pseudo-lipid rows inflate the matrix
- The desktop LipidOne README describes a Perl/Windows standalone tool with Gnuplot/Prima dependencies; LipidOne 2.0's LBB generation runs via its R back-end on the web platform, so local reproduction requires re-implementing the R workflow
- Parsing depends on consistent LIPID MAPS-style shorthand notation; non-standard row labels will fail the class-prefix/chain-list split

## Evidence

- [abstract] three structural levels including LBB: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [intro] motivation for sub-matrix generation: [evidence span withheld — anchor status: not_found]
- [readme] README scope of building block analysis: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."

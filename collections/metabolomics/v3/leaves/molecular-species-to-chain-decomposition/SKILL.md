---
name: molecular-species-to-chain-decomposition
description: Use when a molecular-species intensity matrix (row IDs like 'PC 16:0_18:1')
  must be converted to chain-level building-block rows for statistical or pathway
  analysis at the LBB level.
when_to_use_negative:
- The input is already a chain-level or lipid-class (LCL) sub-matrix rather than a
  molecular-species matrix
- Row identifiers lack parseable lipid class prefixes and chain annotations (e.g.,
  trivial names or non-shorthand labels), so rows cannot be split into class + chain
  components
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: LipidOne 2.0
  role: Web tool that performs the LBB sub-matrix generation automatically from the
    uploaded molecular-species data matrix
  repo: https://lipidone.eu
- name: R
  role: Back-end language in which the LBB sub-matrix computation is implemented;
    used to load, parse, split, divide, and sum the species rows
- name: Lipid One
  role: Standalone Perl desktop predecessor whose building-block analysis this decomposition
    reproduces at the LBB level
  repo: github.com/matteogiulietti/LipidOne
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cpz1.70009
    title: 'LipidOne 2.0: A Web Tool for Discovering Biological Meanings Hidden in
      Lipidomic Data'
  source_content_hash: sha256:4c8d36d75a3ce22dcaf34e8b4292aed21c28c0d0d9af7cf61ccde73d99fbe66d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Lipid One
  - LipidOne 2.0
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# molecular-species-to-chain-decomposition

## Summary

Decomposes an LMS-level (molecular species) lipid intensity matrix into a lipid building block (LBB) sub-matrix, splitting each species row into its constituent acyl/alkyl/alkenyl chains as 'pseudo-lipid' rows while conserving total signal. This supports the third structural level (classes, molecular species, building blocks) explored by LipidOne 2.0.

## When to use

Use when a molecular-species intensity matrix (row IDs like 'PC 16:0_18:1') must be converted to chain-level building-block rows for statistical or pathway analysis at the LBB level.

## When NOT to use

- The input is already a chain-level or lipid-class (LCL) sub-matrix rather than a molecular-species matrix
- Row identifiers lack parseable lipid class prefixes and chain annotations (e.g., trivial names or non-shorthand labels), so rows cannot be split into class + chain components

## Inputs

- Molecular species (LMS-level) intensity matrix with row identifiers in shorthand notation (e.g., 'PC 16:0_18:1') and one column per sample
- Lipid class-to-chain-count mapping (1 for lyso-PC, 2 for PC, 3 for TG, 4 for CL)

## Outputs

- Aggregated chain-level LBB sub-matrix (pseudo-lipid rows such as 'PC_16:0' x original sample columns)
- lbb_matrix.csv file containing the LBB sub-matrix

## How to apply

Load the LMS-level intensity matrix in R and parse each row identifier into a lipid class prefix and a chain list (e.g., 'PC 16:0_18:1' -> class 'PC', chains ['16:0','18:1']). For each class, determine the number of constituent chains (1 for lyso-PC, 2 for PC, 3 for TG, 4 for CL) and split the species row into that many chain rows, dividing every sample intensity value by the chain count so total signal is conserved. Build the pseudo-lipid header for each expanded row by joining the class prefix and chain annotation with an underscore (e.g., 'PC_16:0', 'PC_18:1'). Sum rows with identical pseudo-lipid headers — which arise when different molecular species share a chain, including reversed chain orders — sample-by-sample into a single row, then assemble the aggregated chain-level matrix (pseudo-lipid rows x original sample columns) and write it to lbb_matrix.csv.

## Related tools

- **LipidOne 2.0** (Web tool that performs the LBB sub-matrix generation automatically from the uploaded molecular-species data matrix) — https://lipidone.eu
- **R** (Back-end language in which the LBB sub-matrix computation is implemented; used to load, parse, split, divide, and sum the species rows)
- **Lipid One** (Standalone Perl desktop predecessor whose building-block analysis this decomposition reproduces at the LBB level) — github.com/matteogiulietti/LipidOne

## Examples

```
R> write.csv(lbb_matrix, "lbb_matrix.csv", row.names = TRUE)
```

## Evaluation signals

- Every sample intensity in each expanded chain row equals the original species value divided by the correct chain count (1–4), so the sum over a species' chain rows equals the original species row
- Each expanded row header is a pseudo-lipid formed by joining the lipid class name and chain name with an underscore (e.g., 'PC_16:0', 'PC_18:1')
- Species pairs sharing a chain in reversed order (e.g., 'PC 16:0_18:1' and 'PC 18:1_16:0') contribute to the same summed pseudo-lipid row
- Number of chain rows per species matches the class-specific chain count (lyso-PC=1, PC=2, TG=3, CL=4)
- Output matrix columns are identical to the original sample columns, and the file is written as lbb_matrix.csv

## Limitations

- Row identifiers must follow the parseable shorthand notation (class prefix + chain list); unparseable or non-standard names will break the class/chain splitting
- The chain-count mapping (1–4) must be known per class; classes outside this mapping are not handled by the described procedure
- The desktop LipidOne predecessor only ran on Microsoft Windows 64-bit with GNUPLOT; the chain decomposition itself is implemented in the R back-end of LipidOne 2.0

## Evidence

- [abstract] It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains).: "It facilitates the exploration of the three structural levels of lipids: classes, molecular species, and lipid building blocks (acyl, alkyl, or alkenes chains)."
- [intro] conducting statistical or metabolic analyses for each of these three levels requires the prior application of time- and labor-intensive calculations: [evidence span withheld — anchor status: not_found]
- [readme] LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset.: "LipidOne is a freely available and easy-to-use software to discover the deeper features of building blocks in a lipidomic dataset."

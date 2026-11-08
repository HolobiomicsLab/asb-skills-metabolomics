---
name: smiles-tree-node-constraint-validation
description: Use when decoding or simulating fragment trees from candidate SMILES
  or MSn peak lists and you must enforce TeFT's MW, element-set, and SMILES-tree node-count
  limits before similarity scoring.
when_to_use_negative:
- Input spectra come from instruments with high mass resolution and accuracy (e.g.
  Orbitrap) where the TeFT low-resolution constraint set does not apply.
- The molecule under test contains elements outside C,H,O,N,P,S,Cl,Br,I,F or exceeds
  500 Da — the constraint check would systematically exclude it rather than help.
- Only fingerprint prediction or formula search is needed, with no fragment tree construction
  — MetFrag-style database search or SIRIUS is a better fit.
edam_operation: http://edamontology.org/operation_0249
edam_topics:
- http://edamontology.org/topic_3421
- http://edamontology.org/topic_0092
tools:
- name: TeFT
  role: Produces the simulated SMILES fragment trees from Transformer-decoded candidates
    and defines the node/MW/element constraints that this validation enforces.
  repo: thumingo/TeFT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:2490a38a44b0f003cc444271b31f0c4e25f9b4a2566acb21fd0b143dd505532f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - TeFT
  edam_operation: http://edamontology.org/operation_0249
  edam_topics:
  - http://edamontology.org/topic_3421
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# smiles-tree-node-constraint-validation

## Summary

Validates that simulated or predicted SMILES fragment trees in the TeFT de novo structure pipeline respect the stated chemical constraints (MW < 500 Da, only C,H,O,N,P,S,Cl,Br,I,F, at most 1500 tree nodes) before downstream fragment-tree similarity scoring.

## When to use

Use when decoding or simulating fragment trees from candidate SMILES or MSn peak lists and you must enforce TeFT's MW, element-set, and SMILES-tree node-count limits before similarity scoring.

## When NOT to use

- Input spectra come from instruments with high mass resolution and accuracy (e.g. Orbitrap) where the TeFT low-resolution constraint set does not apply.
- The molecule under test contains elements outside C,H,O,N,P,S,Cl,Br,I,F or exceeds 500 Da — the constraint check would systematically exclude it rather than help.
- Only fingerprint prediction or formula search is needed, with no fragment tree construction — MetFrag-style database search or SIRIUS is a better fit.

## Inputs

- Candidate SMILES fragment trees from TeFT Transformer decoding (RECAP-based)
- Low-resolution MSn peak list from the miniaturized mass spectrometer (top 5-6 peaks + 5 nearby points, ≤100 peaks by intensity)
- Molecular formulas inferred from the MSn data

## Outputs

- Filtered/validated SMILES fragment trees (MW < 500 Da, ≤1500 nodes, allowed element set)
- Flagged or rejected candidates violating the constraints
- Validated fragment trees ready for the similarity scoring table (similarity_scores.csv)

## How to apply

After TeFT's Transformer decodes candidate molecules and they are simulated-fragmented into RECAP-based SMILES fragment trees, check each tree against the paper's constraints: total molecular weight under 500 Da, only the ten elements C, H, O, N, P, S, Cl, Br, I, and F present, and a total SMILES tree node count of no more than 1500. Similarly, when building the experimental fragment tree from the low-resolution MSn peak list (top 5-6 fragmentation peaks plus their five nearby points, max 100 peaks by intensity), verify the extracted molecular formulas fall within the same element kinds and quantity restrictions used to narrow potential molecular formulas. Trees that breach any limit are excluded or truncated before the fragment-tree similarity scoring (matched +6/+3/+0 parent, +5/+2/+0 fragment, +4/+1.5/+0 neutral loss tiers). The rationale is that the constraints keep the fragment-tree search space tractable for on-site miniaturized mass spectrometry where computing resources are limited.

## Related tools

- **TeFT** (Produces the simulated SMILES fragment trees from Transformer-decoded candidates and defines the node/MW/element constraints that this validation enforces.) — thumingo/TeFT

## Evaluation signals

- No validated tree contains an element outside the allowed ten-element set.
- Every validated tree's molecular weight is strictly below 500 Da.
- Total SMILES tree node count per candidate is ≤ 1500.
- Experimental fragment tree was built from ≤100 peaks by intensity, using only the top 5-6 fragmentation peaks plus their five nearby points.
- Post-filter, the ranked similarity scores match the reported top-1 accuracy on the 16 flavonoid alcohols (complete prediction for 8 substances).

## Limitations

- TeFT was only trained on positive-ion-mode (H+) spectra, so negative-mode fragment trees fall outside the validated constraint set.
- The constraint set may exclude valid structures with heavier MW or rare elements, reducing recall on such substances.
- Quantitative comparison with SIRIUS4, MSNovelist, or MassGenie on these constraints was not provided.
- Fragmentation rule library (Supplementary Table S1) availability and versioning is not specified.

## Evidence

- [discussion] Constraints: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] Element set: "permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F"
- [discussion] Node limit: "we have imposed a limitation of 1500 nodes for the total count of SMILES tree nodes"
- [discussion] Peak selection: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] Ranked scoring: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"

---
name: protein-structure-superposition
description: Use when you have computed NMR structure ensembles (from NMRFx torsion-angle
  calculation on NEF constraint files) that need to be aligned and validated against
  experimentally confirmed PDB structures, or when aggregating per-protein quality
  statistics for comparison against published tables.
when_to_use_negative:
- When the input is a single unvalidated structure with no corresponding experimentally
  confirmed PDB reference for alignment
- When the goal is purely de novo structure prediction without constraint files (no
  NEF input for NMRFx calculation)
- When working with RNA or small-molecule structures rather than the protein NMR datasets
  the article's benchmarking covers
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: NMRFx
  role: Performs the torsion-angle structure calculation and provides the batch mode
    that generates and ranks the 200-structure ensemble from which the 10 lowest-cost
    conformers are extracted for superposition
  repo: github.com/nanalysis/nmrfx
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:95d988a8ba8217c7b36ff9d9633c1db2187ad758ef6c97b8702d546af912caf7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nanalysis/nmrfx
  tools:
  - NMRFx
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# protein-structure-superposition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Alignment and comparison of computed protein NMR structure ensembles against experimentally determined reference structures, using NMRFx's structure-calculation output (10-structure PDBx/mmCIF ensembles) superimposed onto deposited PDB coordinates to validate that the calculated structures agree with published assignments.

## When to use

Use when you have computed NMR structure ensembles (from NMRFx torsion-angle calculation on NEF constraint files) that need to be aligned and validated against experimentally confirmed PDB structures, or when aggregating per-protein quality statistics for comparison against published tables.

## When NOT to use

- When the input is a single unvalidated structure with no corresponding experimentally confirmed PDB reference for alignment
- When the goal is purely de novo structure prediction without constraint files (no NEF input for NMRFx calculation)
- When working with RNA or small-molecule structures rather than the protein NMR datasets the article's benchmarking covers

## Inputs

- NEF file containing NMR constraints for a target protein
- Experimentally confirmed protein structure from the PDB (reference for superposition)
- NMRFx structure-calculation output (10-structure PDBx/mmCIF ensemble)

## Outputs

- Aligned PDBx/mmCIF ensemble file per protein (10 lowest-cost conformers)
- Aggregated summary table of per-protein cost-function and quality statistics
- Reproduction comparison against published figures/tables (Fig. 3, SI Tables S3/S4)

## How to apply

For each protein in a benchmarking dataset (e.g., the 100-protein structure dataset in this article), run the NMRFx batch structure calculation to generate a 200-structure ensemble and retain the 10 lowest-cost conformers ranked by the cost function. Output these 10 structures to a PDBx/mmCIF file, then superimpose (align) the retained conformers onto the corresponding experimentally confirmed PDB structure. Aggregate per-protein quality metrics — including cost-function values of the retained conformers — into a summary table. Validate the pipeline by confirming that all proteins yield a complete 10-structure ensemble and that aggregated statistics match published figures and SI tables (e.g., Fig. 3 and Tables S3/S4), which serve as the ground truth for reproduction.

## Related tools

- **NMRFx** (Performs the torsion-angle structure calculation and provides the batch mode that generates and ranks the 200-structure ensemble from which the 10 lowest-cost conformers are extracted for superposition) — github.com/nanalysis/nmrfx

## Examples

```
nmrfxs batch -n 200 -k 10 -a <nef-file>
```

## Evaluation signals

- All 100 proteins in the dataset yield a valid 10-structure PDBx/mmCIF ensemble file
- Aggregated cost-function values of retained conformers match the values summarized in Fig. 3
- Per-protein quality statistics reproduce the values listed in SI Tables S3 and S4
- Superimposed ensembles show strong agreement with experimentally confirmed PDB structures (visual/quantitative check via Fig. 3)

## Limitations

- Evaluation depends on expert review of tabulated values against reported SI tables rather than automated thresholds
- The article does not report RMSD cutoffs or automated agreement metrics for the superposition; agreement is described qualitatively as 'strong agreement' via Fig. 3
- Structure-calculation validation relies on the availability of matching PDB entries for each protein in the dataset

## Evidence

- [discussion] The 10 lowest-cost structures from the 200-structure ensemble were output to a PDBx/mmCIF file, and these 10-structure ensembles were averaged and superimposed onto the corresponding experimentally confirmed structures from the PDB: [evidence span withheld — anchor status: not_found]
- [other] For each 100-protein dataset entry, the 10 lowest-cost structures from the 200-structure ensemble were output to a PDBx/mmCIF file: [evidence span withheld — anchor status: not_found]

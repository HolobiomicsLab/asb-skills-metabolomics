---
name: torsion-angle-ensemble-generation
description: Use when a NEF file containing NMR constraints (e.g. from BMRB) must
  be converted into a ranked PDBx/mmCIF structure ensemble via torsion-angle generation
  in NMRFx.
when_to_use_negative:
- The input is not a NEF constraint file (e.g. raw FIDs or processed spectra) — use
  NMRFx Processor for data processing instead.
- A single, fully converged atomic structure is needed rather than a ranked multi-conformer
  ensemble — this skill produces a 10-best-structure ensemble by cost-function ranking.
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: NMRFx
  role: Runs the torsion-angle structure calculation from NEF constraint files and
    exports the ranked ensemble to PDBx/mmCIF.
  repo: github.com/nanalysis/nmrfx
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:4f2d3bc9915a64f213249f73a067a35b561027618bff44deea6c2979888b4eeb
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
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# torsion-angle-ensemble-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a curated protein NMR structure ensemble by running NMRFx's torsion-angle structure generator on deposited NEF constraint files, retaining the lowest-cost conformers and aligning them for comparison with experimentally determined PDB structures.

## When to use

Use when a NEF file containing NMR constraints (e.g. from BMRB) must be converted into a ranked PDBx/mmCIF structure ensemble via torsion-angle generation in NMRFx.

## When NOT to use

- The input is not a NEF constraint file (e.g. raw FIDs or processed spectra) — use NMRFx Processor for data processing instead.
- A single, fully converged atomic structure is needed rather than a ranked multi-conformer ensemble — this skill produces a 10-best-structure ensemble by cost-function ranking.

## Inputs

- NEF file containing NMR constraints (dihedral restraints, distance restraints, etc.) for a single protein
- Experimentally determined PDB structure for superimposition/quality comparison

## Outputs

- 10-structure PDBx/mmCIF ensemble file per protein (aligned conformers)
- Per-protein cost-function/quality statistics aggregated across the dataset

## How to apply

Collect the NEF constraint file for the target protein (e.g. from the 100-protein structure dataset referenced by the article). Invoke the batch command `nmrfxs batch -n 200 -k 10 -a <nef-file>`: the torsion-angle generator samples 200 candidate structures and retains the 10 best structures as ranked by the cost function. Superimpose the retained conformers onto a representative conformer and export the aligned ensemble to PDBx/mmCIF. Accept the run only if all 10 structures are produced and the cost-function/quality statistics match the values summarized in Fig. 3 and SI Tables S3/S4.

## Related tools

- **NMRFx** (Runs the torsion-angle structure calculation from NEF constraint files and exports the ranked ensemble to PDBx/mmCIF.) — github.com/nanalysis/nmrfx

## Examples

```
nmrfxs batch -n 200 -k 10 -a <nef-file>
```

## Evaluation signals

- All 100 proteins (or the queried subset) yield a 10-structure PDBx/mmCIF ensemble file.
- The 10 retained structures are ranked by the lowest cost-function values from the 200-structure pool.
- Aggregated cost/quality statistics match the article's Fig. 3 and SI Tables S3/S4.
- Averaged ensemble superimposes onto the experimentally confirmed PDB structure with strong agreement.

## Limitations

- NMRFx is under active development, so exact behavior may change.
- Full Python processing scripts and detailed RMSD/MAE statistics are deferred to the SI and not reproduced inline.

## Evidence

- [discussion] these 10-structure ensembles were averaged and superimposed onto the corresponding experimentally confirmed structures from the PDB: [evidence span withheld — anchor status: not_found]

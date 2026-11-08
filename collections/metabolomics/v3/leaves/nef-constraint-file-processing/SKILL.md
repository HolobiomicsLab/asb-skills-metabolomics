---
name: nef-constraint-file-processing
description: Use when you have a NEF-format constraint file (deposited chemical shifts,
  distance/dihedral restraints) for a protein or nucleic acid and need to run or reproduce
  an NMR structure calculation in NMRFx without a pre-existing project.
when_to_use_negative:
- You already have an NMRFx (or NMRViewJ) project with constraints loaded and simply
  want interactive peak picking, assignment, or chemical shift prediction rather than
  a batch structure calculation.
- You only need spectrum processing (FID to processed NUS/non-NUS spectra); the NEF
  batch route addresses structure calculation, not data processing.
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_1317
tools:
- name: NMRFx
  role: Runs the batch torsion-angle structure calculation directly from the supplied
    NEF constraint file via the `nmrfxs batch` subcommand, ranking and retaining conformers
    by cost function.
  repo: github.com/nanalysis/nmrfx
- name: NMRFx Processor
  role: Predecessor component of NMRFx whose functionality is merged into the batch
    structure-calculation tooling used here.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:025206310a7211c0d3bba0f6ce6e2e9e5c9989eb0620c0a433c454e4685d9438
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
  - NMRFx Processor
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# nef-constraint-file-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load NMR constraint data in NEF (NMR Exchange Format) files into NMRFx's batch structure-calculation engine, so that torsion-angle geometry can be generated from deposited chemical shifts, NOEs, J-couplings, RDCs and other restraints rather than from native NMRFx project files.

## When to use

Use when you have a NEF-format constraint file (deposited chemical shifts, distance/dihedral restraints) for a protein or nucleic acid and need to run or reproduce an NMR structure calculation in NMRFx without a pre-existing project.

## When NOT to use

- You already have an NMRFx (or NMRViewJ) project with constraints loaded and simply want interactive peak picking, assignment, or chemical shift prediction rather than a batch structure calculation.
- You only need spectrum processing (FID to processed NUS/non-NUS spectra); the NEF batch route addresses structure calculation, not data processing.

## Inputs

- NEF (NMR Exchange Format) file containing NMR constraints (chemical shift assignments, distance and dihedral restraints) for a protein or RNA entry
- Optionally a reference PDB structure for superposition of the calculated ensemble

## Outputs

- Ensemble of 10 lowest-cost conformers written as a PDBx/mmCIF file
- Per-conformer cost-function (restraint violation) statistics for the retained structures

## How to apply

Obtain the NEF file containing the NMR constraints for your macromolecule, e.g. the per-protein NEF files used for the 100-protein structure dataset in the NMRFx paper. Invoke the batch structure-calculation command `nmrfxs batch -n 200 -k 10 -a <nef-file>`: the `-n 200` flag sets the size of the generated conformer ensemble, `-k 10` keeps only the 10 lowest-cost structures as ranked by NMRFx's cost function, and `-a` points to the NEF constraint file that supplies chemical shifts and restraints. NMRFx performs a torsion-angle structure calculation from the parsed restraints and writes the retained conformers as a PDBx/mmCIF ensemble file. For benchmarking or reproducibility work, superimpose the 10-structure ensemble onto the corresponding experimentally determined PDB structure and tabulate the cost-function values of the retained conformers against the article's reported statistics (Fig. 3 and SI Tables S3/S4) to confirm agreement.

## Related tools

- **NMRFx** (Runs the batch torsion-angle structure calculation directly from the supplied NEF constraint file via the `nmrfxs batch` subcommand, ranking and retaining conformers by cost function.) — github.com/nanalysis/nmrfx
- **NMRFx Processor** (Predecessor component of NMRFx whose functionality is merged into the batch structure-calculation tooling used here.)

## Examples

```
nmrfxs batch -n 200 -k 10 -a ubiquitin.nef
```

## Evaluation signals

- Each processed protein yields exactly 10 structures in the output PDBx/mmCIF ensemble (the `-k 10` retention count).
- Retained conformers show low and consistent cost-function values, comparable to those tabulated for the 100-protein dataset in SI Tables S3/S4.
- Averaged and superimposed 10-structure ensembles align well with the corresponding experimentally determined PDB structures, reproducing the strong agreement shown in Fig. 3.
- Command accepts the NEF file without parse errors and populates the expected restraint categories (distances, dihedrals, chemical shifts).

## Limitations

- Convergence and accuracy depend on the completeness/quality of the restraints in the NEF file; the article benchmarks against a curated 100-protein dataset rather than providing a universal error model.
- Only the 10 best structures of the 200-structure ensemble are retained, so downstream users should not treat the output as a full Boltzmann-like sampling of conformations.
- NMRFx is under active development, so flags and default behavior around `nmrfxs batch` may change between releases.

## Evidence

- [results] Ensemble retention and superposition finding: [evidence span withheld — anchor status: not_found]

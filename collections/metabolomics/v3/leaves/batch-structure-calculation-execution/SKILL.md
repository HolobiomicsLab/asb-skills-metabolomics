---
name: batch-structure-calculation-execution
description: Use when NEF files containing NMR constraints for proteins must be converted
  into ranked PDBx/mmCIF structure ensembles using NMRFx batch calculations.
when_to_use_negative:
- When the goal is NMR data processing (FID to processed spectra), peak picking, or
  chemical shift prediction — use other NMRFx workflow skills, since NMRFx handles
  these separately.
- When the input constraint data are not in NEF format, since the batch command takes
  NEF files directly.
- When only a single structure (not an ensemble) is required, as the batch mode generates
  and ranks 200 candidate structures.
edam_operation: http://edamontology.org/operation_0301
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3300
tools:
- name: NMRFx
  role: Performs the batch torsion-angle structure calculation, ranking, and superimposition
    of structures from NEF constraint files.
  repo: github.com/nanalysis/nmrfx
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:2148654b0d3f8aae2cfebbe255c65791b8ebf715015a5d671160239251888db1
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
  edam_operation: http://edamontology.org/operation_0301
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# batch-structure-calculation-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes NMRFx's torsion-angle molecular structure calculations in batch mode from NEF NMR constraint files, generating and ranking structure ensembles via a cost function, as demonstrated on the article's 100-protein structure dataset.

## When to use

Use when NEF files containing NMR constraints for proteins must be converted into ranked PDBx/mmCIF structure ensembles using NMRFx batch calculations.

## When NOT to use

- When the goal is NMR data processing (FID to processed spectra), peak picking, or chemical shift prediction — use other NMRFx workflow skills, since NMRFx handles these separately.
- When the input constraint data are not in NEF format, since the batch command takes NEF files directly.
- When only a single structure (not an ensemble) is required, as the batch mode generates and ranks 200 candidate structures.

## Inputs

- NEF files containing NMR constraints (per protein)
- Reference/representative conformer for alignment (e.g., from PDB)

## Outputs

- 10-structure ensemble PDBx/mmCIF file per protein (retained lowest-cost structures)
- Aligned (superimposed) PDBx/mmCIF ensemble file per protein
- Aggregated cost-function and quality statistics summary table across proteins

## How to apply

Collect NEF files with NMR constraints for each protein, then invoke 'nmrfxs batch -n 200 -k 10 -a <nef-file>' which performs torsion-angle structure calculations, generates a 200-structure ensemble, and retains the 10 best structures ranked by the cost function. Superimpose the 10 retained structures onto a representative conformer to produce an aligned PDBx/mmCIF ensemble file. Aggregate per-protein cost-function and quality statistics across all proteins into a summary table. Compare aggregated results against published figures and tables (e.g., Fig. 3 and SI Tables S3/S4) to confirm the run reproduces published outcomes.

## Related tools

- **NMRFx** (Performs the batch torsion-angle structure calculation, ranking, and superimposition of structures from NEF constraint files.) — github.com/nanalysis/nmrfx

## Examples

```
nmrfxs batch -n 200 -k 10 -a <nef-file>
```

## Evaluation signals

- All target proteins must yield a 10-structure PDBx/mmCIF ensemble file (e.g., 10 conformers retained per protein in the 100-protein dataset).
- Cost-function values of the retained conformers must be low/ranked within the top 10 of the 200-structure ensemble.
- Aligned ensemble must superimpose onto the experimentally confirmed PDB structures with strong agreement.
- Aggregated per-protein cost/quality statistics must match values reported in the article (Fig. 3, SI Tables S3/S4).
- Output must be in PDBx/mmCIF format, per the article's batch output specification.

## Limitations

- The article does not state the specific NMRFx software version used, so reproducibility against a fixed version may be uncertain.
- Batch output details such as exact cost-function formulation and full statistical output tables are deferred to SI tables (Tables S3/S4) and the SI, not the main text.
- The specific NEF file source location for the 100-protein dataset is referenced only via the article's dataset footnote (dataset65), not a direct link in the text.

## Evidence

- [discussion] Batch structure calculation command and parameters: "nmrfxs batch -n 200 -k 10 -a <nef-file>"
- [discussion] Batch workflow description: "For each protein in the 100-protein structure dataset65, a NEF file containing NMR constraints was processed by NMRFx"
- [abstract] Ensemble generation and ranking: "NMRFx facilitates data processing, peak picking and assignment, chemical shift prediction, molecular structure calculation, and beyond"
- [discussion] Reproducibility demonstration via figure comparison: [evidence span withheld — anchor status: not_found]
- [discussion] Input data format for structure calculation: [evidence span withheld — anchor status: not_found]

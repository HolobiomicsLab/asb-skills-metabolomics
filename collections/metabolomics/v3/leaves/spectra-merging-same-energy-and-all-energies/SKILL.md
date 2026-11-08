---
name: spectra-merging-same-energy-and-all-energies
description: Use when raw vendor/open-format MSn data has been denoised and annotated
  in mzmine and replicate fragmentation spectra must be merged into consensus library
  entries at the same collision energy or across energies, including pseudo-MS2 collapse
  of MSn trees.
when_to_use_negative:
- Before MSn trees are built and purity scores are computed — merging chimeric spectra
  defeats the purpose of the isolation-purity flagging.
- On spectra that do not share the same precursor; SAME_ENERGY and ALL_ENERGIES merge
  only replicate spectra of the same annotated compound.
- For single-stage MS2-only workflows without MSn tree data — ALL_MSN_TO_PSEUDO_MS2
  requires multi-stage MSn input.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzmine
  role: Performs the actual SAME_ENERGY, ALL_ENERGIES and ALL_MSN_TO_PSEUDO_MS2 merging
    of MSn spectra and the quality filtering before library export
  repo: https://github.com/mzmine/mzmine
- name: Orbitrap ID-X
  role: Instrument whose Xcalibur sequence produces the MSn data that are merged into
    consensus spectra
- name: Xcalibur
  role: Acquisition software whose sequences generate the raw spectra that mzmine
    merges
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:ec793f21ae19a04b534e21166f82e9b0803424f0f17add0b4d06e29620714e26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - Orbitrap ID-X
  - Xcalibur
  - mzmine
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectra-merging-same-energy-and-all-energies

## Summary

Builds consensus MSn spectral library entries in mzmine by merging replicate spectra with the SAME_ENERGY and ALL_ENERGIES strategies and collapsing multi-stage MSn trees into pseudo-MS2 spectra via ALL_MSN_TO_PSEUDO_MS2, so the >2 million MSnLib spectra of 30,008 unique small molecules become flat, machine-learning-ready library entries.

## When to use

Use when raw vendor/open-format MSn data has been denoised and annotated in mzmine and replicate fragmentation spectra must be merged into consensus library entries at the same collision energy or across energies, including pseudo-MS2 collapse of MSn trees.

## When NOT to use

- Before MSn trees are built and purity scores are computed — merging chimeric spectra defeats the purpose of the isolation-purity flagging.
- On spectra that do not share the same precursor; SAME_ENERGY and ALL_ENERGIES merge only replicate spectra of the same annotated compound.
- For single-stage MS2-only workflows without MSn tree data — ALL_MSN_TO_PSEUDO_MS2 requires multi-stage MSn input.

## Inputs

- vendor (Thermo .raw) or open-format (mzML) MSn raw files imported into mzmine
- Denoised, background-removed MSn peak lists with MSn trees built
- Metadata sheet with unique sample_id, plate_id and well location for adduct and in-source fragment annotation
- Precursor isolation purity scores used to exclude chimeric spectra

## Outputs

- Consensus spectra merged at same collision energy (SAME_ENERGY)
- Comprehensive consensus spectra across all energies (ALL_ENERGIES)
- Pseudo-MS2 spectra collapsed from MSn trees (ALL_MSN_TO_PSEUDO_MS2)
- Spectra passing the ≥2 signals above noise filter, ready for .json/.mgf/.msp export

## How to apply

In mzmine, after building MSn trees from the imported Orbitrap ID-X/Xcalibur data, first run SAME_ENERGY merging so that replicate spectra acquired at the same collision energy are combined into a single consensus spectrum per energy. Then run ALL_ENERGIES merging to pool consensus spectra across the whole energy range, producing a comprehensive spectrum per precursor. Optionally apply ALL_MSN_TO_PSEUDO_MS2 to collapse higher-order MSn (MS3, MS4, ...) trees into a single pseudo-MS2 spectrum when downstream tools expect flat MS2-style entries. Merge only spectra of the same precursor that passed denoising, background removal, and isolation-purity scoring (chimeric spectra are flagged during MSn tree building), and retain at least two signals above the noise threshold so low-quality entries do not enter the exported library.

## Related tools

- **mzmine** (Performs the actual SAME_ENERGY, ALL_ENERGIES and ALL_MSN_TO_PSEUDO_MS2 merging of MSn spectra and the quality filtering before library export) — https://github.com/mzmine/mzmine
- **Orbitrap ID-X** (Instrument whose Xcalibur sequence produces the MSn data that are merged into consensus spectra)
- **Xcalibur** (Acquisition software whose sequences generate the raw spectra that mzmine merges)

## Examples

```
mzmine module: MSn spectral library generation with merge strategies SAME_ENERGY, ALL_ENERGIES, ALL_MSN_TO_PSEUDO_MS2
```

## Evaluation signals

- Merged consensus spectrum contains at least two signals above the noise threshold, matching the quality filter of MSnLib.
- SAME_ENERGY output groups spectra by identical collision energy; ALL_ENERGIES output pools across the full energy range.
- Pseudo-MS2 spectra produced by ALL_MSN_TO_PSEUDO_MS2 remain representable as flat MS2 spectra without loss of the MSn tree structure needed for the >2 million spectrum library.
- Spectra flagged as chimeric by the isolation-purity score are excluded from merged entries.
- Final merged spectra export successfully to .json, .mgf and .msp via mzmine.

## Limitations

- Merging strategies require the mzmine module set — they are not available as standalone scripts.
- ALL_MSN_TO_PSEUDO_MS2 collapses tree information, losing the explicit MSn stage structure for machine-learning use cases that benefit from multi-stage trees.
- Merging does not substitute for denoising, background removal, adduct annotation or purity scoring; those must be done beforehand in the mzmine pipeline.
- Results depend on the quality of the input metadata — the unique sample_id and plate_id/well mapping must be correct for annotation to work before merging.

## Evidence

- [full_text] Merge strategies named by the workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Post-processing steps before merging: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
- [full_text] Filter criterion that merged spectra must satisfy: [evidence span withheld — anchor status: not_found]
- [abstract] Merge targets a >2 million spectra MSn tree library: "MS n Lib, a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"

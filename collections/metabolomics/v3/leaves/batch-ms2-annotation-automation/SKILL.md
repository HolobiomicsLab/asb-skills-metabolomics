---
name: batch-ms2-annotation-automation
description: Use when per-feature MS2 spectra exported by MZmine need formula, 2D
  structure, and natural-product class annotation at sample/feature-batch scale via
  SIRIUS/CSI:FingerID, CANOPUS, and NPClassifier.
when_to_use_negative:
- The input features lack an associated MS2 scan or fall outside the RT window 0.5–8.0
  min — the ENPKG feature filtering module removes these before SIRIUS export, so
  annotation would be wasted or fail.
- The goal is spectral library matching only (e.g., GNPS-style spectral similarity
  search against experimental libraries) rather than in-silico formula/structure and
  class prediction.
- Only a feature quantification table is available without the MZmine 'Export for
  SIRIUS' .mgf files — isotopic patterns and MS2 spectra are required inputs.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2258
tools:
- name: SIRIUS
  role: Computes molecular formula candidates with ZODIAC re-ranking and CSI:FingerID
    structure search on the exported MS1 isotopic patterns and MS2 spectra; run locally
    via install_sirius.sh with credentials in .env.
  repo: https://boecker-lab.github.io/docs.sirius.github.io/install/
- name: CANOPUS
  role: Predicts chemical classes from the feature spectra of both PI and NI modes,
    run as part of the same SIRIUS call (no additional parameters needed).
- name: NPClassifier
  role: API called with the anisomeric SMILES of candidate 2D structures to retrieve
    their natural-product taxonomy.
- name: ENPKG (04_enpkg_sirius_canopus)
  role: Automated script that collates the per-feature SIRIUS/CSI:FingerID and CANOPUS
    annotation records and persists versions/parameters as knowledge graph sub-properties.
  repo: https://github.com/enpkg/enpkg_full
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:b1c0be5ae0245023e53c7318d439faac91adb697808c02167c0820328d4e194b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - ENPKG
  - ENPKG (04_enpkg_sirius_canopus)
  - NPClassifier
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# batch-ms2-annotation-automation

## Summary

Batch-convert MZmine-exported MS2 spectra (.mgf with MS1 isotopic patterns) into per-feature structural annotations (SIRIUS + ZODIAC formula re-ranking + CSI:FingerID) and chemical class annotations (CANOPUS, NPClassifier), as implemented in the ENPKG workflow step 04_enpkg_sirius_canopus.

## When to use

Use when per-feature MS2 spectra exported by MZmine need formula, 2D structure, and natural-product class annotation at sample/feature-batch scale via SIRIUS/CSI:FingerID, CANOPUS, and NPClassifier.

## When NOT to use

- The input features lack an associated MS2 scan or fall outside the RT window 0.5–8.0 min — the ENPKG feature filtering module removes these before SIRIUS export, so annotation would be wasted or fail.
- The goal is spectral library matching only (e.g., GNPS-style spectral similarity search against experimental libraries) rather than in-silico formula/structure and class prediction.
- Only a feature quantification table is available without the MZmine 'Export for SIRIUS' .mgf files — isotopic patterns and MS2 spectra are required inputs.

## Inputs

- Per-sample 'Export for SIRIUS' files from MZmine containing feature MS1 isotopic patterns and MS2 spectra (.mgf)
- features_quanted_mgf files with per-feature metadata (retention time, ion mobility) per sample
- params/user.yml configuration parameters (commented, self-explanatory)
- .env file with PATH_TO_SIRIUS and SIRIUS_USERNAME/SIRIUS_PASSWORD credentials

## Outputs

- Per-feature SIRIUS formula candidate rankings with ZODIAC re-ranking and CSI:FingerID 2D structure candidates (anisomeric SMILES)
- CANOPUS chemical class predictions per feature per ionization mode
- NPClassifier natural-product taxonomy annotations per candidate structure
- RDF records under enpkg:has_sirius_annotation sub-properties carrying the versions/parameters used, ready for knowledge graph integration

## How to apply

Collect the per-sample 'Export for SIRIUS' .mgf files (feature MS1 isotopic patterns and MS2 spectra) produced by MZmine during feature detection, after the feature filtering module has kept only features with an associated MS2 scan. Run SIRIUS with ZODIAC formula re-ranking and CSI:FingerID structure search restricted to biological databases: PI mode with SIRIUS v4.9.12 and adducts [M + H]+, [M + NH4]+, [M + K]+, [M + Na]+; NI mode with SIRIUS v5.5.4 and adducts [M − H]−, [M + K-2H]−, [M + Na-2H]−; in both cases use 10 ppm tolerance, maximum m/z 800, and 10 candidates considered. Run CANOPUS on the feature spectra of both ionization modes to predict chemical classes (no extra parameters needed). Submit the anisomeric SMILES of candidate 2D structures to the NPClassifier API to retrieve natural-product taxonomy, then collate all per-feature records with the automated ENPKG script at 04_enpkg_sirius_canopus. Save the parameters used at each step so they become retrievable from the knowledge graph as sub-properties of enpkg:has_sirius_annotation (e.g. enpkg:has_sirius_annotation_00c339c3b183cb7fbf466b4d334dbef2), ensuring reproducibility.

## Related tools

- **SIRIUS** (Computes molecular formula candidates with ZODIAC re-ranking and CSI:FingerID structure search on the exported MS1 isotopic patterns and MS2 spectra; run locally via install_sirius.sh with credentials in .env.) — https://boecker-lab.github.io/docs.sirius.github.io/install/
- **CANOPUS** (Predicts chemical classes from the feature spectra of both PI and NI modes, run as part of the same SIRIUS call (no additional parameters needed).)
- **NPClassifier** (API called with the anisomeric SMILES of candidate 2D structures to retrieve their natural-product taxonomy.)
- **ENPKG (04_enpkg_sirius_canopus)** (Automated script that collates the per-feature SIRIUS/CSI:FingerID and CANOPUS annotation records and persists versions/parameters as knowledge graph sub-properties.) — https://github.com/enpkg/enpkg_full

## Examples

```
sh workflow/00_workflow_all.sh
```

## Evaluation signals

- Every feature with an associated MS2 scan carries formula/structure candidates within the 10 ppm parent-mass tolerance.
- Run versions and parameters are retrievable from the knowledge graph as sub-properties of enpkg:has_sirius_annotation (e.g., enpkg:has_sirius_annotation_00c339c3b183cb7fbf466b4d334dbef2).
- Structural annotations are compared with ISDB-LOTUS annotations, and agreement between the two independent approaches strengthens confidence in the spectral annotations.
- NI and PI mode runs used the correct SIRIUS versions (v4.9.12 PI, v5.5.4 NI) with the expected adduct lists and 10 candidates considered.
- The workflow completes on the small Zenodo test dataset (doi:10.5281/zenodo.10018590) in about 10 minutes.

## Limitations

- Requires a local SIRIUS installation (PATH_TO_SIRIUS) and a SIRIUS account (SIRIUS_USERNAME/SIRIUS_PASSWORD in .env), which are git-ignored secrets that must be configured per machine.
- Search space is constrained to maximum m/z 800 and 10 ppm parent-mass tolerance with only 10 candidates considered; features outside these bounds are not annotated.
- Different SIRIUS versions are used per ionization mode (v4.9.12 PI vs v5.5.4 NI), so outputs may not be strictly comparable across modes.
- The emikg successor pipeline warns it 'will likely not work if your dataset to be treated is not formatted in the exact same way as the example dataset', so deviating input formats may break annotation collation.

## Evidence

- [results] SIRIUS adducts and versions per mode: [evidence span withheld — anchor status: not_found]
- [results] Cross-validation with ISDB annotations: [evidence span withheld — anchor status: not_found]
- [results] Parameter persistence in the graph: [evidence span withheld — anchor status: not_found]
- [readme] README: two annotation methods including Sirius/CSI:FingerID: [evidence span withheld — anchor status: external_doc]
- [readme] README: local SIRIUS installation step: "Run `src/install_sirius.sh <install_dir>` to download and unpack the latest release for your platform"

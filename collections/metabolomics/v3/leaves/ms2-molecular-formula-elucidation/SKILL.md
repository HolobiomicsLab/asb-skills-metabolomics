---
name: ms2-molecular-formula-elucidation
description: Use when MZmine 'Export for SIRIUS' .mgf files (feature MS1 isotopic
  patterns + MS2 spectra) need per-feature molecular formula and 2D structure candidates
  within a 10 ppm parent-mass tolerance, e.g. to later cross-check against ISDB-LOTUS
  spectral annotations.
when_to_use_negative:
- Input spectra lack an associated MS2 scan or isotopic pattern (the upstream feature
  filtering module is what guarantees every kept feature carries an MS2 scan) — rerun
  feature detection/filtering first
- The goal is spectral library matching (e.g. GNPS/ISDB-LOTUS search) rather than
  in-silico formula/structure elucidation — ENPKG handles that in a separate ISDB
  step (03_enpkg_mn_isdb_isdb_taxo) and the two are meant to be compared, not substituted
- Features have m/z above 800 or adducts outside the configured lists ([M + H]+, [M
  + NH4]+, [M + K]+, [M + Na]+ in PI; [M – H]−, [M + K-2H]−, [M + Na-2H]− in NI) —
  they fall outside this SIRIUS configuration
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Predicts molecular formula candidates from MS1 isotopic patterns and MS2 spectra;
    formula candidates re-ranked with ZODIAC (v4.9.12 for PI, v5.5.4 for NI, 10 ppm,
    max m/z 800, 10 candidates)
  repo: https://boecker-lab.github.io/docs.sirius.github.io/install/
- name: CSI:FingerID
  role: Structure search run inside SIRIUS, restricted to biological databases, returning
    candidate 2D structures (anisomeric SMILES)
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: ZODIAC
  role: Re-ranks SIRIUS molecular formula candidates across features of the sample
    before structure search
- name: CANOPUS
  role: Predicts chemical classes from the feature MS2 spectra of both ionization
    modes (no parameters needed)
- name: NPClassifier
  role: API that assigns natural-product taxonomy to the anisomeric SMILES of the
    candidate 2D structures
- name: ENPKG (04_enpkg_sirius_canopus)
  role: Automated script that collates per-feature SIRIUS/CSI:FingerID and CANOPUS
    annotation records into the knowledge graph, storing parameters as enpkg:has_sirius_annotation
    sub-properties
  repo: https://github.com/enpkg/enpkg_full
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:71a8d47bbd9c0a433c8a318f7c8982637a19fdb48277bdb505ac0a5c0275f27d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - CSI:FingerID
  - ENPKG
  - ENPKG (04_enpkg_sirius_canopus)
  - NPClassifier
  - SIRIUS
  - ZODIAC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ms2-molecular-formula-elucidation

## Summary

In the ENPKG workflow, MZmine-exported per-feature MS1 isotopic patterns and MS2 spectra are elucidated into molecular formula and 2D structure candidates with SIRIUS (ZODIAC re-ranking) and CSI:FingerID, complemented by CANOPUS class predictions and NPClassifier natural-product taxonomy, as a per-sample step before knowledge-graph integration.

## When to use

Use when MZmine 'Export for SIRIUS' .mgf files (feature MS1 isotopic patterns + MS2 spectra) need per-feature molecular formula and 2D structure candidates within a 10 ppm parent-mass tolerance, e.g. to later cross-check against ISDB-LOTUS spectral annotations.

## When NOT to use

- Input spectra lack an associated MS2 scan or isotopic pattern (the upstream feature filtering module is what guarantees every kept feature carries an MS2 scan) — rerun feature detection/filtering first
- The goal is spectral library matching (e.g. GNPS/ISDB-LOTUS search) rather than in-silico formula/structure elucidation — ENPKG handles that in a separate ISDB step (03_enpkg_mn_isdb_isdb_taxo) and the two are meant to be compared, not substituted
- Features have m/z above 800 or adducts outside the configured lists ([M + H]+, [M + NH4]+, [M + K]+, [M + Na]+ in PI; [M – H]−, [M + K-2H]−, [M + Na-2H]− in NI) — they fall outside this SIRIUS configuration

## Inputs

- Per-sample MZmine 'Export for SIRIUS' files (.mgf) containing feature MS1 isotopic patterns and MS2 spectra
- Features that survived the ENPKG feature filtering (associated MS2 scan, RT between 0.5 and 8.0 min)
- .env file with PATH_TO_SIRIUS, SIRIUS_USERNAME, SIRIUS_PASSWORD
- params/user.yml configuration for the dataset to process

## Outputs

- Per-feature ranked molecular formula candidates (ZODIAC re-ranked) within 10 ppm parent-mass tolerance
- Per-feature CSI:FingerID candidate 2D structures (anisomeric SMILES) from biological databases
- CANOPUS chemical class predictions per feature spectrum (both ionization modes)
- NPClassifier natural-product taxonomy per candidate structure
- RDF annotation records enpkg:has_sirius_annotation_<hash> with embedded SIRIUS versions and parameters

## How to apply

Collect the per-sample 'Export for SIRIUS' files produced by MZmine feature detection (step 1 of Figure 2B) and run SIRIUS separately per ionization mode: PI mode with SIRIUS v4.9.12 and adducts [M + H]+, [M + NH4]+, [M + K]+, [M + Na]+; NI mode with SIRIUS v5.5.4 and adducts [M – H]−, [M + K-2H]−, [M + Na-2H]− — in both cases with 10 ppm tolerance, maximum m/z 800, and 10 candidates considered. Enable ZODIAC to re-rank formula candidates using fragmentation-tree agreement across features, and run CSI:FingerID structure search restricted to biological databases to obtain candidate 2D structures. Run CANOPUS on the same spectra (no extra parameters) to predict chemical classes, then submit the anisomeric SMILES of CSI:FingerID candidates to the NPClassifier API for natural-product taxonomy. Collate all per-feature records with the automated ENPKG script 04_enpkg_sirius_canopus, saving run versions and parameters so they become sub-properties of enpkg:has_sirius_annotation (e.g. enpkg:has_sirius_annotation_00c339c3b183cb7fbf466b4d334dbef2) in the RDF graph for reproducibility. Locally, SIRIUS is installed with the bundled script and located via the PATH_TO_SIRIUS variable plus SIRIUS_USERNAME/SIRIUS_PASSWORD credentials in .env.

## Related tools

- **SIRIUS** (Predicts molecular formula candidates from MS1 isotopic patterns and MS2 spectra; formula candidates re-ranked with ZODIAC (v4.9.12 for PI, v5.5.4 for NI, 10 ppm, max m/z 800, 10 candidates)) — https://boecker-lab.github.io/docs.sirius.github.io/install/
- **CSI:FingerID** (Structure search run inside SIRIUS, restricted to biological databases, returning candidate 2D structures (anisomeric SMILES)) — https://bio.informatik.uni-jena.de/software/sirius/
- **ZODIAC** (Re-ranks SIRIUS molecular formula candidates across features of the sample before structure search)
- **CANOPUS** (Predicts chemical classes from the feature MS2 spectra of both ionization modes (no parameters needed))
- **NPClassifier** (API that assigns natural-product taxonomy to the anisomeric SMILES of the candidate 2D structures)
- **ENPKG (04_enpkg_sirius_canopus)** (Automated script that collates per-feature SIRIUS/CSI:FingerID and CANOPUS annotation records into the knowledge graph, storing parameters as enpkg:has_sirius_annotation sub-properties) — https://github.com/enpkg/enpkg_full

## Examples

```
bash src/install_sirius.sh /opt/sirius && export PATH_TO_SIRIUS=/opt/sirius/sirius/bin/sirius
```

## Evaluation signals

- Every feature with an associated MS2 scan carries formula/structure candidates within the 10 ppm parent-mass tolerance
- Run versions and parameters are retrievable from the graph as sub-properties of enpkg:has_sirius_annotation (e.g. enpkg:has_sirius_annotation_00c339c3b183cb7fbf466b4d334dbef2)
- Both ionization modes were processed with the mode-specific adduct lists and SIRIUS versions (v4.9.12 PI, v5.5.4 NI)
- Each feature has up to 10 considered candidates, with formula ranking reflecting ZODIAC re-ranking
- Each candidate SMILES has a corresponding NPClassifier natural-product taxonomy assignment and a CANOPUS class prediction

## Limitations

- Fixed configuration bounds the search: maximum m/z 800, 10 ppm tolerance, 10 candidates considered, and a closed adduct list per ionization mode — features outside these bounds are not elucidated
- SIRIUS must be installed locally with account credentials (PATH_TO_SIRIUS, SIRIUS_USERNAME, SIRIUS_PASSWORD in .env), adding a machine-specific dependency
- Different SIRIUS versions are used per ionization mode (v4.9.12 for PI, v5.5.4 for NI), so results across modes are not strictly version-homogeneous
- Structural annotations from this step are in-silico predictions; the paper's validation strategy is agreement with independent ISDB-LOTUS annotations, not ground-truth confirmation

## Evidence

- [results] Comparing structural annotations results from two different approaches: [evidence span withheld — anchor status: not_found]
- [readme] SIRIUS installation: "Run `src/install_sirius.sh <install_dir>` to download and unpack the latest release for your platform"
- [readme] SIRIUS runtime configuration: "Runtime secrets and machine-specific paths (e.g. `PATH_TO_SIRIUS`, `SIRIUS_USERNAME`, `SIRIUS_PASSWORD`) live in a `.env` file"
- [readme] ENPKG annotation of features: [evidence span withheld — anchor status: external_doc]
- [methods] Upstream feature filtering: "Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min."

---
name: insilico-ms2-annotation-sirius-canopus
description: Use when you have MZmine-processed features with associated MS2 scans
  (mzML / mgf spectra) from LC-HRMS2 DDA data in positive and/or negative ionization
  mode and want per-feature in-silico candidate structures, formulas, and chemical
  classes to merge with ISDB spectral matches and taxonomic.
when_to_use_negative:
- When the input is already a feature table or mgf of MS2 spectra without the MZmine-processed
  per-sample context required by the ENPKG workflow (e.g. raw vendor files not yet
  converted).
- When you only need spectral-library matching to known reference spectra and do not
  want formula/class annotations — ISDB-LOTUS matching in 03_enpkg_mn_isdb_isdb_taxo
  alone may suffice.
- When features lack an associated MS2 scan or fall outside the RT 0.5–8.0 min window,
  as they are filtered out and will not be annotated.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: SIRIUS
  role: Computes molecular formulas and generates fragmentation trees from MS2 spectra;
    CSI:FingerID then proposes candidate structures for each feature.
  repo: https://boecker-lab.github.io/docs.sirius.github.io/install/
- name: CANOPUS
  role: Predicts compound substructure classes from MS2 spectra, complementing SIRIUS
    structural candidates.
  repo: https://boecker-lab.github.io/docs.sirius.github.io/install/
- name: ENPKG (04_enpkg_sirius_canopus)
  role: Wraps SIRIUS/CANOPUS invocation for per-sample processing and passes results
    forward to RDF formatting.
  repo: https://github.com/enpkg/enpkg_full
- name: MZmine
  role: Upstream feature detection that produces the per-feature MS2 scans and quantification
    data which are the input to SIRIUS/CANOPUS annotation.
  repo: https://github.com/enpkg/enpkg_full
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:32de40d101ed0b687f27609b974553c42185b57fb9ee8edf478ec45bf0e75074
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CANOPUS
  - ENPKG
  - ENPKG (04_enpkg_sirius_canopus)
  - MZmine
  - RDF
  - SIRIUS
  - matchms
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# insilico-ms2-annotation-sirius-canopus

## Summary

Submit exported MS2 spectra to SIRIUS with CSI:FingerID and CANOPUS to obtain in-silico structural (molecular formula and candidate structures) and substructure/compound-class annotations, which are then reranked and integrated alongside ISDB-LOTUS spectral matches into the ENPKG knowledge graph.

## When to use

Use when you have MZmine-processed features with associated MS2 scans (mzML / mgf spectra) from LC-HRMS2 DDA data in positive and/or negative ionization mode and want per-feature in-silico candidate structures, formulas, and chemical classes to merge with ISDB spectral matches and taxonomic reweighting.

## When NOT to use

- When the input is already a feature table or mgf of MS2 spectra without the MZmine-processed per-sample context required by the ENPKG workflow (e.g. raw vendor files not yet converted).
- When you only need spectral-library matching to known reference spectra and do not want formula/class annotations — ISDB-LOTUS matching in 03_enpkg_mn_isdb_isdb_taxo alone may suffice.
- When features lack an associated MS2 scan or fall outside the RT 0.5–8.0 min window, as they are filtered out and will not be annotated.

## Inputs

- Per-sample LC-MS/MS DDA mzML files (positive and/or negative ionization mode)
- MZmine feature lists with associated MS2 scans (features filtered to RT 0.5–8.0 min)
- Locally installed SIRIUS binary path in .env as PATH_TO_SIRIUS
- SIRIUS account credentials (SIRIUS_USERNAME, SIRIUS_PASSWORD)

## Outputs

- Per-feature molecular formula and ranked candidate structures (up to 10 candidates) from CSI:FingerID
- Per-feature substructure/compound-class annotations from CANOPUS
- SIRIUS/CANOPUS results per sample ready for RDF integration into the ENPKG knowledge graph via 06_enpkg_graph_builder

## How to apply

After MZmine feature detection and filtering (keep only features with an associated MS2 scan and RT between 0.5 and 8.0 min), submit the per-feature MS2 spectra to SIRIUS/CSI:FingerID and CANOPUS with ppm tolerance 10, maximum m/z 800, 10 candidates per feature, and ionization modes restricted to adducts [M+H]+, [M+NH4]+, [M+K]+, [M+Na]+ in PI and [M−H]−, [M+K-2H]−, [M+Na-2H]− in NI. Because samples are processed individually in the sample-centric ENPKG design, this step runs after 03_enpkg_mn_isdb_isdb_taxo (molecular networking and ISDB-LOTUS spectral matching with modified cosine score cutoff 0.7) and its per-sample results are subsequently formatted as RDF by the graph builder (06_enpkg_graph_builder). The key decision point is combining the two complementary annotation approaches — SIRIUS-based molecular fingerprinting and ISDB spectral matching with taxonomic distance and network propagation reranking — to strengthen confidence in spectral annotations rather than relying on a single method. For 1,600 plant extracts this yields in-silico annotations for 788,623 PI-mode and 364,967 NI-mode features. The workflow expects a locally installed Sirius binary whose path is stored in .env as PATH_TO_SIRIUS, with SIRIUS_USERNAME/SIRIUS_PASSWORD credentials.

## Related tools

- **SIRIUS** (Computes molecular formulas and generates fragmentation trees from MS2 spectra; CSI:FingerID then proposes candidate structures for each feature.) — https://boecker-lab.github.io/docs.sirius.github.io/install/
- **CANOPUS** (Predicts compound substructure classes from MS2 spectra, complementing SIRIUS structural candidates.) — https://boecker-lab.github.io/docs.sirius.github.io/install/
- **ENPKG (04_enpkg_sirius_canopus)** (Wraps SIRIUS/CANOPUS invocation for per-sample processing and passes results forward to RDF formatting.) — https://github.com/enpkg/enpkg_full
- **MZmine** (Upstream feature detection that produces the per-feature MS2 scans and quantification data which are the input to SIRIUS/CANOPUS annotation.) — https://github.com/enpkg/enpkg_full

## Examples

```
bash src/install_sirius.sh /opt/sirius; set -a; source .env; set +a; sh workflow/00_workflow_all.sh
```

## Evaluation signals

- All features with an associated MS2 scan and RT between 0.5 and 8.0 min should receive a SIRIUS molecular formula and ≤10 ranked candidate structures.
- Adduct lists must match the ionization mode: [M+H]+, [M+NH4]+, [M+K]+, [M+Na]+ for PI and [M−H]−, [M+K-2H]−, [M+Na-2H]− for NI.
- Per-sample SIRIUS/CANOPUS outputs must be generated for every sample in the batch before 06_enpkg_graph_builder is invoked.
- Combining SIRIUS annotations with ISDB-LOTUS spectral matches (cosine ≥ 0.7 edges, min cosine score 0.2, 6 minimal matching fragments) should raise annotation confidence relative to either method alone.
- Across the 1,600-extract dataset, the SIRIUS/CANOPUS step should produce annotations consistent with the reported scale of 788,623 PI-mode and 364,967 NI-mode features.

## Limitations

- SIRIUS/CANOPUS run with ppm tolerance 10 and maximum m/z 800 — compounds above m/z 800 are outside the configuration described here.
- The article's missing signals note that no statistical metrics (e.g. FDR, p-values, effect sizes, AUROC) are reported for compound annotations, so confidence is assessed only via the combined two-method comparison rather than formal statistics.
- A SIRIUS account (SIRIUS_USERNAME / SIRIUS_PASSWORD) and a locally installed Sirius binary are required; workflow fails if PATH_TO_SIRIUS or credentials are not set in .env.
- In-silico annotations are reranked against taxonomic distance and network propagation, so annotations on samples with poorly represented taxa may be less reliable.

## Evidence

- [other] Submit exported MS2 spectra to SIRIUS/CSI:FingerID and CANOPUS annotation (ppm tolerance 10, maximum m/z 800, 10 candidates; adducts [M+H]+, [M+NH4]+, [M+K]+, [M+Na]+ in PI and [M−H]−, [M+K-2H]−, [M+Na-2H]− in NI): [evidence span withheld — anchor status: not_found]
- [methods] Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min.: "Each feature list was filtered using the feature filtering module to keep only features with an associated MS2 scan and an RT between 0.5 and 8.0 min."
- [results] Comparing structural annotations results from two different approaches...allows the user to strengthen confidence in spectral annotation: [evidence span withheld — anchor status: not_found]
- [readme] Run `src/install_sirius.sh <install_dir>` to download and unpack the latest release for your platform: "Run `src/install_sirius.sh <install_dir>` to download and unpack the latest release for your platform"
- [readme] Runtime secrets and machine-specific paths (e.g., `PATH_TO_SIRIUS`, `SIRIUS_USERNAME`, `SIRIUS_PASSWORD`) live in a `.env` file that is ignored by git.: "Runtime secrets and machine-specific paths (e.g., `PATH_TO_SIRIUS`, `SIRIUS_USERNAME`, `SIRIUS_PASSWORD`) live in a `.env` file that is ignored by git."

---
name: in-silico-msms-annotation
description: Use when untargeted LC-MS/MS data in .mzML format contains features that
  cannot be annotated by spectral matching alone, and per-feature chemical structures,
  formulas, and chemical classes must be assigned and ranked.
when_to_use_negative:
- When all features are already covered by exact spectral database matches (e.g. the
  betaine case where public spectral databases suffice and are not standardized beyond
  their contents)
- When features belong to lipid or macrocyclic natural-product classes (e.g. Marchantin)
  where SIRIUS false positives and complex fragmentation make in silico ranking unreliable
- When input is not tandem MS2 data (MAW takes MS2 .mzML files as input in R)
edam_operation: http://edamontology.org/operation_3804
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MAW-R
  role: R segment performing MS2 preprocessing, spectral database dereplication via
    Spectra, and SIRIUS in silico annotation
  repo: https://github.com/zmahnoor14/MAW
- name: MAW-Py
  role: Python segment merging spectral and in silico results and selecting one final
    candidate per feature using RDKit
  repo: https://github.com/zmahnoor14/MAW
- name: Spectra
  role: R package integrating GNPS/HMDB/MassBank spectral databases with cosine similarity
    matching
  repo: https://github.com/zmahnoor14/MAW
- name: SIRIUS
  role: In silico candidate generation and CSI:FingerID compound database annotation
    against COCONUT
  repo: https://github.com/zmahnoor14/MAW
- name: MetFrag
  role: Alternative CLI 2.5.0 in silico candidate generation against a local structure
    database CSV (as evaluated in Table 4)
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: maw-r
  role: Docker image (zmahnoor/maw-r:1.0.8) providing the reproducible R execution
    environment
  repo: https://github.com/zmahnoor14/MAW
- name: maw-py
  role: Docker image (zmahnoor/maw-py:1.0.7) providing the reproducible Python execution
    environment
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:660fa73e9e8e796e001635ae61cd4e570167cfd07e94b9ddf026a4ba0c2e93ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - MAW-Py
  - MAW-R
  - MetFrag
  - SIRIUS
  - Spectra
  - maw-py
  - maw-r
  edam_operation: http://edamontology.org/operation_3804
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# in-silico-msms-annotation

## Summary

Annotate untargeted MS2 metabolomics features by combining spectral database dereplication (Spectra with GNPS/HMDB/MassBank), in silico candidate generation (SIRIUS/CSI:FingerID or MetFrag), and RDKit-based final candidate selection as implemented in the MAW workflow.

## When to use

Use when untargeted LC-MS/MS data in .mzML format contains features that cannot be annotated by spectral matching alone, and per-feature chemical structures, formulas, and chemical classes must be assigned and ranked.

## When NOT to use

- When all features are already covered by exact spectral database matches (e.g. the betaine case where public spectral databases suffice and are not standardized beyond their contents)
- When features belong to lipid or macrocyclic natural-product classes (e.g. Marchantin) where SIRIUS false positives and complex fragmentation make in silico ranking unreliable
- When input is not tandem MS2 data (MAW takes MS2 .mzML files as input in R)

## Inputs

- MS2 LC-MS/MS data in .mzML format
- Spectral reference databases gnps.rda, hmdb.rda, mbankNIST.rda (Zenodo record 7519270)
- Local structure database CSV with Identifier, InChI, SMILES, molecular_weight columns (e.g. COCONUT, Zenodo record 7704937)
- MS1DATA.csv feature table and spectral_results.csv from MAW-R dereplication
- SIRIUS .ms files (file/insilico/SIRIUS/no_isotope) and metparam_list.txt for MetFrag

## Outputs

- mergedResults-with-one-Candidates.csv with one final annotated candidate per feature
- Per-feature candidate lists scored by cosine similarity, MAW-defined metrics, and Tanimoto structural similarity
- Chemical structure assignments importable into a chemical structure similarity network

## How to apply

Run MAW-R inside the zmahnoor/maw-r:1.0.8 container on the .mzML input with 15 ppm precursor tolerance to dereplicate against the gnps.rda, hmdb.rda, and mbankNIST.rda spectral databases (Zenodo record 7519270). Generate in silico candidates either with SIRIUS 5 (ppm_max 5, ppm_max_ms2 15, 30 candidates, profile 'orbitrap', db 'coconut') on the .ms files under file/insilico/SIRIUS/no_isotope, or with MetFrag CLI 2.5.0 (metparam_list.txt + MetFragCommandLine-2.5.0.jar) against a structure database CSV containing Identifier, InChI, SMILES, and molecular_weight columns. Then run the MAW-Py post-processing script (Workflow_Python_Script_all_MetFrag.py or Workflow_Python_Script_all_SIRIUS.py) with spectral_results.csv, the GNPS/HMDB/MassBank directories, and MS1DATA.csv, applying a MetFrag score threshold of 0.75, to merge spectral and in silico results and select one final candidate per feature via RDKit. Larger compound databases such as PubChem push the true structure to lower ranks, so focused databases like COCONUT are preferred for natural products.

## Related tools

- **MAW-R** (R segment performing MS2 preprocessing, spectral database dereplication via Spectra, and SIRIUS in silico annotation) — https://github.com/zmahnoor14/MAW
- **MAW-Py** (Python segment merging spectral and in silico results and selecting one final candidate per feature using RDKit) — https://github.com/zmahnoor14/MAW
- **Spectra** (R package integrating GNPS/HMDB/MassBank spectral databases with cosine similarity matching) — https://github.com/zmahnoor14/MAW
- **SIRIUS** (In silico candidate generation and CSI:FingerID compound database annotation against COCONUT) — https://github.com/zmahnoor14/MAW
- **MetFrag** (Alternative CLI 2.5.0 in silico candidate generation against a local structure database CSV (as evaluated in Table 4)) — https://github.com/ipb-halle/MetFragRelaunched
- **maw-r** (Docker image (zmahnoor/maw-r:1.0.8) providing the reproducible R execution environment) — https://github.com/zmahnoor14/MAW
- **maw-py** (Docker image (zmahnoor/maw-py:1.0.7) providing the reproducible Python execution environment) — https://github.com/zmahnoor14/MAW

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- mergedResults-with-one-Candidates.csv exists and contains exactly one final candidate per feature
- MAW-defined metrics (matching score, etc.) each reach at least 0.5 in addition to a high cosine similarity score for retained candidates
- MetFrag candidates below the 0.75 score threshold are excluded from final selection
- On the 37-feature standards dataset, coverage should approximate Table 4: MAW structural 100%, chemical class 97.3%, formula 100%; Spectra 75.7%; SIRIUS 70.2%; MetFrag CLI 2.5.0 with PubChem 32.4%
- Intermediate provenance files (spectral_results.csv, SIRIUS .ms outputs) are retained so results are reproducible and traceable

## Limitations

- SIRIUS CLI provides no deterministic probability score, so candidate ranking thresholds were discarded as case-dependent
- MAW's 0.5 threshold on MAW-defined metrics can miss candidates with low cosine similarity as false negatives
- Mismatch between experimental spectra in spectral databases and structures in compound databases leads to a low number of annotated features
- Large databases such as PubChem can push the actual structure to a low rank (e.g. top 100th candidate); natural-product-focused databases like COCONUT and GNPS improve top-ranked recovery
- Only SIRIUS (CSI:FingerID) is integrated for compound databases in the first MAW version; lipid class annotation via SIRIUS can produce false positives

## Evidence

- [abstract] combining spectral and in silico annotation: "combining tandem mass spectrometry (MS 2 ) input data pre-processing, spectral and compound database matching with computational classification, and in silico annotation"
- [methods] SIRIUS 5 run parameters: [evidence span withheld — anchor status: not_found]
- [methods] MetFrag score threshold: "'--score_thresh', type=float, default=0.75, help='score threshold for MetFrag results (default: 0.75)'"
- [discussion] 0.5 MAW-defined metric threshold: "All of these MAW-defined metrics should at least have a 0.5 matching score in addition to a high cosine similarity score"
- [discussion] PubChem ranking caveat: "Larger databases such as PubChem can lead to a larger list of candidates with the actual structure being at a lower rank (e.g.: top 100th candidate)"
- [readme] README: workflow description: [evidence span withheld — anchor status: not_found]

---
name: ms-finder-spectral-library-format-preparation
description: Use when building an RT-annotated structure library for MS-FINDER ≥3.24
  to rank/exclude isomer candidates in untargeted MS/MS annotation, starting from
  SMILES sets from HMDB, DrugBank, FooDB, STOFF-IDENT and other Retiplib databases.
when_to_use_negative:
- The compounds are complex high-molecular-weight lipids (e.g. triacylglycerols or
  phosphatidylcholines) that were largely excluded from the Retip training sets —
  predictions outside the training chemical landscape may be more erroneous
- The goal is discriminating closely eluting compounds with narrow retention time
  search windows (e.g. 12 min UPLC methods), since ~1 min prediction errors are not
  adequate for that
- Retention times must be adjusted for small changes in a specific chromatographic
  method — this was beyond the scope of the Retip study
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3292
tools:
- name: Retip
  role: R package providing the pre-trained HILIC/RPLC Keras models, getCD() CDK descriptor
    calculation, and cesc() centering/scaling used to batch-predict retention times
    for library compounds
  repo: https://github.com/olobion/Retip
- name: Retiplib
  role: Companion R package bundling the biochemical databases (HMDB, DrugBank, FooDB,
    etc.) whose SMILES feed the RT prediction workflow
  repo: https://github.com/olobion/Retiplib
- name: MS-FINDER
  role: 'Consumer of the RT-annotated library: uses Retip-predicted retention times
    in isomer search to rank candidates and reduce false-positive structures (68%
    candidate reduction reported)'
- name: MS-DIAL
  role: Mass spectrometry software into which Retip RT predictions are likewise integrated
    for compound annotation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:47e4b0283b6ab96d350745e292a2114ba4cea5f15c72ca343e82271f5e70a633
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - MS-DIAL
  - MS-FINDER
  - R
  - Retip
  - Retiplib
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# ms-finder-spectral-library-format-preparation

## Summary

Prepares RT-annotated compound library files for MS-FINDER (v3.24+/3.30) by batch-predicting HILIC and reversed-phase LC retention times for large small-molecule structure sets (e.g. 320,919 compounds from Retip's bundled databases) with Retip's pre-trained Keras neural-network models, so predicted RTs can rank or exclude candidate structures during untargeted metabolomics annotation.

## When to use

Use when building an RT-annotated structure library for MS-FINDER ≥3.24 to rank/exclude isomer candidates in untargeted MS/MS annotation, starting from SMILES sets from HMDB, DrugBank, FooDB, STOFF-IDENT and other Retiplib databases.

## When NOT to use

- The compounds are complex high-molecular-weight lipids (e.g. triacylglycerols or phosphatidylcholines) that were largely excluded from the Retip training sets — predictions outside the training chemical landscape may be more erroneous
- The goal is discriminating closely eluting compounds with narrow retention time search windows (e.g. 12 min UPLC methods), since ~1 min prediction errors are not adequate for that
- Retention times must be adjusted for small changes in a specific chromatographic method — this was beyond the scope of the Retip study

## Inputs

- SMILES structure collections from the biochemical databases bundled with Retiplib (HMDB, KNApSAcK, ChEBI, DrugBank, SMPDB, YMDB, T3DB, FooDB, NANPDB, STOFF, BMDB, LipidMAPS, Urine, Saliva, Feces, ECMDB, CSF, Serum, PubChem.1, PlantCyc, UNPD, BLEXP, NPA, COCONUT)
- Pre-trained Retip Keras model artifacts for HILIC and reversed-phase (RPLC) chromatography
- Compound identifiers and SMILES strings per candidate structure

## Outputs

- Structured RT prediction table (e.g. retip_rt_predictions_320919.csv) with one row per compound: identifier, SMILES, HILIC-predicted RT, RPLC-predicted RT
- RT-annotated library files for deployment in MS-FINDER v3.24+ for ranking/excluding structures
- Out-of-domain flags for compounds outside the HILIC/RPLC training chemical space

## How to apply

Install the R packages via devtools::install_github('olobion/Retiplib') and devtools::install_github('olobion/Retip') (Retip 2.0 requires R 4.4.0 and python3 for the Keras/TensorFlow models). Load the pre-trained HILIC and RPLC Keras model artifacts, collect SMILES from the biochemical databases bundled in Retiplib, and compute the 286 CDK molecular descriptors for each structure with the Retip getCD() function using rJava/rcdk. Center and scale the descriptor matrix with cesc() as required for neural-network input, then predict retention times with the HILIC-trained and RPLC-trained Keras models (RT.spell()/getRT.smile()). Flag compounds outside the training chemical space — e.g. high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines, which were excluded from training — as out-of-domain. Write compound identifier, SMILES, HILIC-predicted RT, and RPLC-predicted RT per compound into a structured table that can be shipped as an RT-annotated library file consumed by MS-FINDER 3.24+/3.30, where predictions act as a method-specific filter alongside accurate mass precursors and MS/MS spectra.

## Related tools

- **Retip** (R package providing the pre-trained HILIC/RPLC Keras models, getCD() CDK descriptor calculation, and cesc() centering/scaling used to batch-predict retention times for library compounds) — https://github.com/olobion/Retip
- **Retiplib** (Companion R package bundling the biochemical databases (HMDB, DrugBank, FooDB, etc.) whose SMILES feed the RT prediction workflow) — https://github.com/olobion/Retiplib
- **MS-FINDER** (Consumer of the RT-annotated library: uses Retip-predicted retention times in isomer search to rank candidates and reduce false-positive structures (68% candidate reduction reported))
- **MS-DIAL** (Mass spectrometry software into which Retip RT predictions are likewise integrated for compound annotation)

## Examples

```
devtools::install_github('olobion/Retiplib'); devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Output contains exactly one prediction row per input compound with non-negative HILIC and RPLC RT values
- Spot-checked RT values agree with the RT values shipped in MS-FINDER 3.24+ within the reported Keras mean-absolute-error tolerances (0.78 min HILIC, 0.57 min RPLC)
- All descriptor matrices were centered and scaled with cesc() before Keras prediction (unscaled descriptors invalidate neural-network predictions)
- Triacylglycerol- and phosphatidylcholine-like compounds falling outside the HILIC/RPLC training chemical space are explicitly flagged as out-of-domain
- Downstream MS-FINDER isomer search on test data (e.g. mouse blood plasma) shows a large candidate-structure reduction, ~68% expected with 1 min RT exclusion windows

## Limitations

- 1 min RT exclusion windows excluded about 25% of true positives in the Retip models
- Predictions outside the chemical landscape of the training set may be more erroneous; accuracy is limited by the number and chemical diversity of training compounds
- None of the current models yield validation MAEs better than ±1 min, precluding discrimination of closely eluting compounds in typical 12 min UPLC methods
- RT predictions cannot yet be adjusted for small changes in chromatographic methods, and molecular descriptors may not fully explain LC buffer/microenvironment interactions

## Evidence

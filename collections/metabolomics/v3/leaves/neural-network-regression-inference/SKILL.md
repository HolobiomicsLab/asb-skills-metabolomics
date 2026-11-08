---
name: neural-network-regression-inference
description: Use when RT predictions are needed for large structure libraries (e.g.
  320,919 SMILES from public databases) using a pre-trained neural network on molecular
  descriptors, with centering/scaling before inference.
when_to_use_negative:
- The query compounds lie outside the training chemical landscape (e.g. complex high-molecular-weight
  lipids like triacylglycerols or phosphatidylcholines that were largely excluded
  from training) — predictions may be more erroneous
- Narrow retention-time search windows are required (e.g. resolving closely eluting
  compounds in typical 12 min UPLC methods) — 1 min prediction errors preclude this
- Chromatographic conditions changed from the training methods — the study did not
  test adjusting predictions for small changes in chromatographic methods
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: Retip
  role: R package providing pre-trained Keras models, getCD() descriptor computation,
    cesc() centering/scaling, and RT.spell()/getRT.smile() inference functions
  repo: https://github.com/olobion/Retip
- name: Keras
  role: Neural-network regression backend used for RT prediction; outperformed random
    forest, BRNN, XGBoost, and LightGBM with minimum overfitting
- name: MS-FINDER
  role: Consumes the RT-annotated library (v3.24+) to rank or exclude candidate structures
    during isomer search
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:5bcf4734231b2cb7078b3139173c1ca9cdba55c56a450d36a93df34e35814e5a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - MS-FINDER
  - R
  - Retip
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# neural-network-regression-inference

## Summary

Apply pre-trained Keras neural-network regression models to predict HILIC and reversed-phase LC retention times for small molecules from 286 CDK molecular descriptors, as done in Retip where Keras achieved MAEs of 0.78 min (HILIC) and 0.57 min (RPLC).

## When to use

Use when RT predictions are needed for large structure libraries (e.g. 320,919 SMILES from public databases) using a pre-trained neural network on molecular descriptors, with centering/scaling before inference.

## When NOT to use

- The query compounds lie outside the training chemical landscape (e.g. complex high-molecular-weight lipids like triacylglycerols or phosphatidylcholines that were largely excluded from training) — predictions may be more erroneous
- Narrow retention-time search windows are required (e.g. resolving closely eluting compounds in typical 12 min UPLC methods) — 1 min prediction errors preclude this
- Chromatographic conditions changed from the training methods — the study did not test adjusting predictions for small changes in chromatographic methods

## Inputs

- SMILES structure strings for query compounds (e.g. from HMDB, DrugBank, FooDB, STOFF and the other databases bundled in Retiplib)
- Pre-trained Keras model artifacts for HILIC and RPLC retention times stored in the Retip package
- Molecular descriptor matrix computed with the CDK descriptors via Retip getCD()

## Outputs

- Structured table (e.g. retip_rt_predictions_320919.csv) with compound identifier, SMILES, HILIC-predicted RT, and RPLC-predicted RT
- Out-of-domain flags for compounds outside the training chemical space (e.g. TAGs, PCs)

## How to apply

Load the pre-trained HILIC- and RPLC-trained Keras models bundled with the Retip R package. For each input SMILES, compute the 286 CDK descriptors via Retip's getCD() function, then center and scale the descriptor matrix with cesc() as required for neural-network (Keras) predictions — scaling is essential because the Keras network expects normalized inputs. Run inference separately with the HILIC and RPLC models via RT.spell()/getRT.smile(). Flag compounds outside the training chemical space (e.g. high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines, excluded from training) as out-of-domain, since predictions there may be more erroneous. Treat ~±1 min MAE as the practical threshold: Keras yielded 0.78 min (HILIC) and 0.57 min (RPLC) MAE, and this window was used to exclude false positives in MS-FINDER while discarding ~25% true positives.

## Related tools

- **Retip** (R package providing pre-trained Keras models, getCD() descriptor computation, cesc() centering/scaling, and RT.spell()/getRT.smile() inference functions) — https://github.com/olobion/Retip
- **Keras** (Neural-network regression backend used for RT prediction; outperformed random forest, BRNN, XGBoost, and LightGBM with minimum overfitting)
- **MS-FINDER** (Consumes the RT-annotated library (v3.24+) to rank or exclude candidate structures during isomer search)

## Examples

```
devtools::install_github('olobion/Retip'); devtools::install_github('olobion/Retiplib')
```

## Evaluation signals

- Output contains one prediction row per input compound with non-negative HILIC and RPLC RT values
- Spot-checked predictions agree with RT values shipped in MS-FINDER 3.24+ within the reported tolerances (0.78 min HILIC, 0.57 min RPLC MAE)
- Descriptor matrix is centered and scaled (cesc()) before Keras inference — predictions on raw unscaled descriptors indicate misuse
- Compounds outside the training domain (e.g. TAGs, phosphatidylcholines) are flagged as out-of-domain rather than trusted

## Limitations

- None of the current models yield validation MAEs better than ±1 min; molecular descriptors themselves may not be sufficiently complex to explain LC buffer/microenvironment interactions
- Prediction accuracy is limited by the number of training compounds and their chemical diversity; many test/validation chemicals were not well represented by the training models
- 1 min exclusion windows removed about 25% of true positives

## Evidence

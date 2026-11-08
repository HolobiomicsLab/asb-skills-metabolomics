---
name: cross-method-elution-order-mapping
description: Use when you have predicted RTs from one chromatographic context and
  experimental RTs from a different method (e.g. a PredRet ECM such as FEM_long, LIFE_old,
  RIKEN, or FEM_orbitrap_plasma) and need to align/project RTs across methods for
  candidate ranking or filtering.
when_to_use_negative:
- The target chromatographic method has too few identified molecules (<~50 anchors)
  to fit a stable quartic projection
- The analysis goal is annotation purely by accurate mass or MS/MS spectra without
  any retention-time information
- Predicted and experimental RTs come from the same chromatographic method, so no
  cross-method projection is needed
tools:
- name: R
  role: Fits the robust quartic polynomial least-squares projection via the stats::poly
    function
- name: Keras for R
  role: Provides the deep learning model (DLM) whose predicted RTs are projected onto
    other chromatographic methods
- name: HMDB
  role: Source of the putative candidate molecule list (molecules with KEGG numbers)
    searched at 10 ppm mass tolerance
- name: PredRet database
  role: 'Supplies experimental RT data for the projection target methods (ECMs: FEM_long,
    LIFE_old, RIKEN, FEM_orbitrap_plasma)'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:96aa1cc101b8876880dbd9558e61dc5205631347c945ffd798a114576308548b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - Keras for R
  - PredRet database
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-method-elution-order-mapping

## Summary

Projects deep-learning-predicted reverse-phase retention times (from the METLIN SMRT-trained model) onto a different experimental chromatographic method via a robust quartic polynomial regression, so that predicted RTs can be compared against experimental RTs from another lab's LC-MS method.

## When to use

Use when you have predicted RTs from one chromatographic context and experimental RTs from a different method (e.g. a PredRet ECM such as FEM_long, LIFE_old, RIKEN, or FEM_orbitrap_plasma) and need to align/project RTs across methods for candidate ranking or filtering.

## When NOT to use

- The target chromatographic method has too few identified molecules (<~50 anchors) to fit a stable quartic projection
- The analysis goal is annotation purely by accurate mass or MS/MS spectra without any retention-time information
- Predicted and experimental RTs come from the same chromatographic method, so no cross-method projection is needed

## Inputs

- DLM-predicted retention times for a molecule list (e.g. 6,823 HMDB molecules with KEGG numbers) from the SMRT-trained Keras model
- Experimental retention times for identified molecules in the target chromatographic method (e.g. PredRet ECM RT data from FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
- A random subset of ~50 identified molecules per ECM to serve as projection anchor points

## Outputs

- A fitted quartic projection function (poly-based lm model) per experimental chromatographic method
- Projected predicted RTs for all molecules on each target method's RT scale
- Per-candidate RT errors used to rank putative identities, plus projection error metrics, ROC curves (AUC 0.65-0.69), optimal thresholds, and top-3 ranking percentages compiled into a results table

## How to apply

Build a per-method projection function by fitting a robust polynomial (quartic, degree-4) least-squares regression of experimental RT on predicted RT using the R stats poly() function, trained on a random subset of 50 identified molecules from the target ECM. Apply this fitted projection to the DLM-predicted RTs of all molecules so they become comparable to that ECM's experimental RT scale. In the paper's validation, projected RTs were then paired with a 10 ppm accurate-mass search against 6,823 HMDB molecules (with KEGG numbers) to rank putative candidates by RT error, yielding top-3 correct-identity ranking in ~70% of cases and ROC AUCs of 0.65–0.69 across ECMs 1–4. The quartic form and the 50-molecule training subset are the key parameters: enough anchor points to capture non-linear RT scaling between methods, few enough to only require commonly identified compounds.

## Related tools

- **R** (Fits the robust quartic polynomial least-squares projection via the stats::poly function)
- **Keras for R** (Provides the deep learning model (DLM) whose predicted RTs are projected onto other chromatographic methods)
- **HMDB** (Source of the putative candidate molecule list (molecules with KEGG numbers) searched at 10 ppm mass tolerance)
- **PredRet database** (Supplies experimental RT data for the projection target methods (ECMs: FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma))

## Examples

```
fit_ecm <- lm(exp_rt ~ poly(pred_rt, 4, raw = TRUE), data = anchors_50); proj_rt <- predict(fit_ecm, newdata = dlm_rts)
```

## Evaluation signals

- Projected-vs-experimental RT projection errors are computed for each ECM and compiled in a results table
- ROC AUC from discretizing relative RT error (0-100% in 2.5% intervals) falls in the reported 0.65-0.69 range for ECMs 1-4
- Correct molecular identity ranks among the top 3 candidates in approximately 70% of cases for every tested ECM (Fig. 5e, j, o, t)
- The quartic projection model is fitted on exactly a random subset of 50 identified molecules per ECM and reproduces the reported optimal thresholds (Fig. 5d, i, n, s)

## Limitations

- Projection quality depends on the availability of identified molecules in the target method to anchor the polynomial fit; methods with sparse identifications may project poorly
- The ~70% top-3 ranking and 0.65-0.69 AUC figures were demonstrated on reverse-phase chromatography ECMs from PredRet and may not transfer to other separation modes
- Ranking relies on accurate-mass candidate lists (10 ppm), so isobaric candidates with similar predicted RTs remain ambiguous

## Evidence

- [abstract] SMRT dataset scope: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [intro] cross-method projection aim: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
- [intro] candidate ranking aim: "Further, we focused on the ability to rank and filter putative candidates in real metabolomics experiments based on accurate mass search and predicted RT"

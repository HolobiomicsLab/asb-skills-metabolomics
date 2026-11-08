---
name: putative-identity-ranking-by-rt-error
description: Use when accurate-mass search (e.g., 10 ppm) returns multiple putative
  candidates for a feature's m/z and an experimentally acquired RT is available, so
  candidates must be ranked or filtered using predicted-vs-observed retention time
  error.
when_to_use_negative:
- Accurate mass search returns a single unique candidate with no need for RT-based
  ranking (e.g., mass search already yields one hit at 10 ppm).
- No experimental RT is available for the feature, or the chromatographic method is
  not amenable to the RT projection (e.g., ion mobility or GC separation where no
  RT model was built).
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: METLIN SMRT dataset
  role: Source of experimentally acquired reverse-phase RTs (80,038 small molecules)
    used to train the RT prediction model whose outputs are projected and compared
    against experimental RTs for candidate ranking
- name: HMDB
  role: Compound library providing molecules (with KEGG numbers) against which the
    10 ppm accurate mass search retrieves putative candidates for RT-error ranking
- name: Keras for R
  role: Framework used to build and run the deep learning model that predicts RT for
    each candidate
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:c6dde5dda8babfa2fafd111b19de8e5902172cbe0142ab9e6240513d0869e629
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - Keras for R
  - METLIN SMRT dataset
  - PredRet database
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# putative-identity-ranking-by-rt-error

## Summary

Ranks and filters putative metabolite identities retrieved from accurate mass search by comparing predicted RTs (projected onto the experimental chromatographic method) against observed RTs, exploiting the 70%-top-3 ranking accuracy demonstrated on the METLIN SMRT dataset.

## When to use

Use when accurate-mass search (e.g., 10 ppm) returns multiple putative candidates for a feature's m/z and an experimentally acquired RT is available, so candidates must be ranked or filtered using predicted-vs-observed retention time error.

## When NOT to use

- Accurate mass search returns a single unique candidate with no need for RT-based ranking (e.g., mass search already yields one hit at 10 ppm).
- No experimental RT is available for the feature, or the chromatographic method is not amenable to the RT projection (e.g., ion mobility or GC separation where no RT model was built).

## Inputs

- List of putative candidates from accurate mass search (10 ppm) with candidate m/z and predicted RTs from a DLM trained on the METLIN SMRT dataset
- Experimental RTs for identified molecules used to fit the projection function per chromatographic method (e.g., PredRet ECM data: FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
- Experimental RT of the unknown feature being annotated

## Outputs

- Ranked list of putative identities ordered by RT error
- Filtered candidate set after applying the RT-error threshold
- ROC curve / top-3 ranking statistics per chromatographic method (AUC ~0.65–0.69, optimal thresholds)

## How to apply

Retrieve putative candidates by accurate mass search of the protonated/deprotonated ion m/z against a compound library such as HMDB at 10 ppm tolerance, then compute each candidate's predicted RT with the deep learning model trained on the METLIN SMRT dataset (80,038 small molecules). Build an RT projection function for the specific chromatographic method via robust polynomial least-squares regression (e.g., quartic, fitted with R stats::poly on ~50 identified molecules per ECM), and project predicted RTs onto the experimental RT scale. Rank candidates by the RT error between projected predicted RT and the feature's experimental RT, and apply an RT-error threshold (e.g., ROC-derived ~30% relative error window) to filter candidates; the correct identity lands in the top 3 in ~70% of cases across chromatographic methods.

## Related tools

- **METLIN SMRT dataset** (Source of experimentally acquired reverse-phase RTs (80,038 small molecules) used to train the RT prediction model whose outputs are projected and compared against experimental RTs for candidate ranking)
- **HMDB** (Compound library providing molecules (with KEGG numbers) against which the 10 ppm accurate mass search retrieves putative candidates for RT-error ranking)
- **Keras for R** (Framework used to build and run the deep learning model that predicts RT for each candidate)

## Examples

```
R: model <- keras_model_sequential(); predRT <- predict(model, feats); projRT <- poly(predRT, degree = 4, coefs = ecmFit); rank_cands(mz, rt_exp, projRT, tol_ppm = 10)
```

## Evaluation signals

- Correct molecular identity ranked among top 3 candidates in ~70% of cases per chromatographic method (ECMs 1–4).
- ROC AUC values for RT-error–based candidate filtering fall in the 0.65–0.69 range per ECM.
- Projection function is a quartic polynomial fitted with R stats::poly on ~50 identified molecules per method; projection error should be low enough for meaningful RT comparison.
- Accurate mass search tolerance is 10 ppm for protonated/deprotonated ions against HMDB molecules with KEGG numbers.

## Limitations

- RT projection quality depends on having ~50 identified molecules per chromatographic method to fit the polynomial; fewer anchors degrade ranking accuracy.
- Ranking performance is bounded (~70% top-3; AUC 0.65–0.69) — RT error alone may not always uniquely resolve the correct identity among near-isomeric candidates.
- Applicability tied to reverse-phase LC-MS chromatographic methods covered by the SMRT dataset and PredRet ECMs; other separation modes may not be supported.

## Evidence

- [abstract] Ranking candidates by RT error against predicted RT enables top-3 identification in 70% of cases: "in 70\documentclass[12pt]{minimal} ... \begin{document}$\%$\end{document}% of the cases, the correct molecular identity was ranked among the top 3 candidates based on their predicted retention time"
- [methods] Accurate mass search at 10 ppm retrieves the putative candidate pool that RT error then ranks: [evidence span withheld — anchor status: not_found]
- [methods] Projection function is a quartic polynomial fitted with R stats::poly on ~50 identified molecules per method: [evidence span withheld — anchor status: not_found]
- [results] ROC AUC for RT-error filtering is 0.65–0.69 across the experimental chromatographic methods: [evidence span withheld — anchor status: not_found]

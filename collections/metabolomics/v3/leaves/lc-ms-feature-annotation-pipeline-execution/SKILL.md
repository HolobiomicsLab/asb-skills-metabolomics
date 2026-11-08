---
name: lc-ms-feature-annotation-pipeline-execution
description: Use when a feature peak table from LC-MS untargeted metabolomics must
  be annotated against KEGG and per-peak compound candidates need diffusion-based
  prioritization rather than plain m/z matching.
when_to_use_negative:
- When the goal is annotation against databases other than KEGG (e.g. HMDB, Lipid
  Maps, T3DB) — mWISE currently supports only KEGG.
- When a plain m/z match to a database is sufficient without contextual prioritization
  or diffusion scoring.
- When the input is raw instrument data rather than a preprocessed, filtered feature
  peak table.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: mWISE
  role: 'Main pipeline: matching, clustering-based filtering, and diffusion-based
    candidate prioritization of KEGG compounds for LC-MS peaks'
  repo: https://dev.b2s.club/b2slab/mWISE
- name: GenomicRanges
  role: Rapidly matches all LC-MS peaks to the KEGG database given a set of adducts
    and fragments in the matching Stage
- name: DiffuStats
  role: Diffuses the label vector through a given biological network to produce diffusion
    scores in set.diffusion
- name: FELLA
  role: Source of the biological network used in one diffusion configuration
- name: xMSannotator
  role: State-of-the-art annotation baseline used for comparison against mWISE configurations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:8905331fcd035784abd8de2417441f19ffbf12cc5df51ce4eb0d6af08dd9fdd4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  tools:
  - DiffuStats
  - FELLA
  - GenomicRanges
  - R
  - mWISE
  - xMSannotator
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# lc-ms-feature-annotation-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute the mWISE annotation pipeline for LC-MS metabolomics features: match peaks to KEGG, filter candidates by feature clustering, and prioritize compounds with network diffusion to produce top-ranked candidate annotations per peak.

## When to use

Use when a feature peak table from LC-MS untargeted metabolomics must be annotated against KEGG and per-peak compound candidates need diffusion-based prioritization rather than plain m/z matching.

## When NOT to use

- When the goal is annotation against databases other than KEGG (e.g. HMDB, Lipid Maps, T3DB) — mWISE currently supports only KEGG.
- When a plain m/z match to a database is sufficient without contextual prioritization or diffusion scoring.
- When the input is raw instrument data rather than a preprocessed, filtered feature peak table.

## Inputs

- LC-MS feature peak table (m/z, retention time, intensity) from untargeted metabolomics
- KEGG compound database with adduct/fragment annotations
- FELLA or RClass biological network for diffusion
- Reference compound set (InChIKeys) for benchmarking

## Outputs

- Prioritized list of up to three KEGG compound candidates per peak with diffusion scores
- Matched and filtered peak-to-compound mapping (annotation table)
- Performance metrics (sensitivity, specificity, F1) when run against references

## How to apply

Run mWISE in R on a preprocessed LC-MS feature peak list, matching mass-to-charge ratios against the KEGG database using GenomicRanges over a set of adducts and fragments. Apply the cluster-based filter built from spectral clustering (DBSCAN or k-means variants) so features likely originating from the same metabolite are grouped and false positives entering diffusion are reduced. Compute diffusion labels via diffusion.input and diffuse them through DiffuStats over a biochemical network (FELLA or RClass) to obtain diffusion scores per candidate. Select the top three candidates per peak as the final prioritized proposal and compare against xMSannotator or mummichog using sensitivity, specificity, and F1 on reference compound sets.

## Related tools

- **mWISE** (Main pipeline: matching, clustering-based filtering, and diffusion-based candidate prioritization of KEGG compounds for LC-MS peaks) — https://dev.b2s.club/b2slab/mWISE
- **GenomicRanges** (Rapidly matches all LC-MS peaks to the KEGG database given a set of adducts and fragments in the matching Stage)
- **DiffuStats** (Diffuses the label vector through a given biological network to produce diffusion scores in set.diffusion)
- **FELLA** (Source of the biological network used in one diffusion configuration)
- **xMSannotator** (State-of-the-art annotation baseline used for comparison against mWISE configurations)

## Examples

```
Rscript -e "mWISE::matchingStage(peaks, adducts, fragments); mWISE::featuresClustering(peaks); mWISE::diffusion(peaks, network='FELLA', score='raw')"
```

## Evaluation signals

- Sensitivity of mWISE matching stage on Assay 1 reaches 0.92 (144 TP / 13 FN / 157 references) — a similar ratio of recovered reference peaks indicates the matching step is operating correctly.
- Cluster-based filter should reduce false positives substantially (e.g. Assay 2 FP drop from 17598 to 1671 with specificity ≈ 0.91); if FP counts stay high the clustering/filter parameters are mis-set.
- Diffusion prioritization should yield F1 above xMSannotator baseline (0.15–0.22); FELLA-based diffusion reached up to 0.49 (Assay 6, raw score, binary input).
- Tanimoto structural similarity of proposed candidates to original compounds should be higher than xMSannotator's (hit ratios ~0.59–0.76), confirming the diffusion step is prioritizing chemically relevant compounds.
- Computation time per annotation should be lower than xMSannotator, consistent with all four mWISE configurations outperforming it in runtime.

## Limitations

- mWISE only annotates against KEGG; it does not support HMDB, Lipid Maps, or T3DB unlike xMSannotator — an acknowledged limitation.
- Performance depends on the availability of adduct/fragment matching options and the quality of the input peak list; peaks without signal or violating the 80% intensity rule must be removed before running.
- Diffusion configuration choice matters: probability input with z-normalized scores or binary input with raw score/unique annotation produce different candidate rankings and must be selected deliberately.

## Evidence

- [methods] The matching Stage command uses the Genomic Ranges R package to rapidly match all the LC-MS peaks to KEGG database: [evidence span withheld — anchor status: not_found]
- [methods] cluster Based.filter command filters the data, thus reducing the number of false positive values: [evidence span withheld — anchor status: not_found]

---
name: metabolite-feature-class-annotation
description: Use when a feature-importance output (e.g., a SHAP Top-20 list from a
  fine-tuned VAE on ccRCC serum metabolomics) consists of raw ion features that need
  to be assigned to metabolite classes before being reported or validated experimentally.
when_to_use_negative:
- Input is already a fully annotated metabolite table with compound identities rather
  than raw ion features—mzLearn and this skill operate at the ion MS1 level and do
  not perform compound identification.
- The analysis requires definitive structural elucidation or MS/MS spectral matching
  within the mzLearn pipeline itself; mzLearn explicitly does not perform de-isotoping
  or MS/MS identification.
- The feature matrix comes from an LC mode other than HILIC positive mode, since the
  pre-trained model and its ion-feature mapping were developed specifically for HILIC
  positive mode.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mzLearn
  role: Produces the ion MS1 feature table (median rt/m/z with normalized intensities)
    that constitutes the ranked feature list to be annotated; does not itself perform
    de-isotoping or MS/MS identification, so its outputs serve as the input to downstream
    annotation workflows.
  repo: https://github.com/ReviveMed/mzEmbed
- name: SHAP
  role: Generates the Top-20 ranked feature-importance lists (from the fine-tuned
    prognostic and ICI-response models) whose ion features are then mapped to metabolite
    classes.
- name: VAE
  role: Fine-tuned model whose learned metabolite representations yield the feature-importance
    rankings (via SHAP) that are subsequently annotated; the same architecture is
    used to produce the prognostic and ICI-response Top-20 lists.
  repo: https://github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:c097cee677812a33f1c36d146a9911b118541b43bd0d0c91ff7b5ac26b04e7b0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  repo_url: https://github.com/ReviveMed/mzEmbed
  tools:
  - SHAP
  - VAE
  - mzLearn
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolite-feature-class-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Map the ranked ion features (m/z, retention time) emitted by mzLearn or a fine-tuned VAE onto annotated metabolite classes—such as acyl-carnitines and lysophosphatidylcholines—to turn an abstract SHAP feature-importance list into a biologically interpretable candidate set for downstream de-isotoping and MS/MS confirmation.

## When to use

Use when a feature-importance output (e.g., a SHAP Top-20 list from a fine-tuned VAE on ccRCC serum metabolomics) consists of raw ion features that need to be assigned to metabolite classes before being reported or validated experimentally.

## When NOT to use

- Input is already a fully annotated metabolite table with compound identities rather than raw ion features—mzLearn and this skill operate at the ion MS1 level and do not perform compound identification.
- The analysis requires definitive structural elucidation or MS/MS spectral matching within the mzLearn pipeline itself; mzLearn explicitly does not perform de-isotoping or MS/MS identification.
- The feature matrix comes from an LC mode other than HILIC positive mode, since the pre-trained model and its ion-feature mapping were developed specifically for HILIC positive mode.

## Inputs

- SHAP Top-20 ranked ion-feature list from a fine-tuned VAE (e.g., ICI-response or prognostic model)
- mzLearn ion-feature table with median rt and m/z values and normalized intensities across samples
- Reference mapping from ion features (m/z/rt) to metabolite classes (acyl-carnitines, LPC, etc.)

## Outputs

- Annotated list of ranked features with assigned metabolite classes (e.g., acyl-carnitines C6, C8, C12:1, C18:1; LPC C16:1)
- Overlap count between two independently ranked Top-20 lists
- Candidate set of prioritized features for downstream de-isotoping, annotation, and MS/MS confirmation

## How to apply

Start from the ion-feature matrix produced by mzLearn—a two-dimensional table of detected features defined by median rt and m/z values with normalized intensities across samples—rather than raw mzML files, since mzLearn outputs are at the ion MS1 level. Take the ranked feature list (for example the SHAP Top-20 from the fine-tuned ICI-response model) and map each ranked ion feature to an annotated metabolite class using the article's known-class assignments: acyl-carnitines (C6, C8, C12:1, C18:1) and LPC (C16:1). Because mzLearn explicitly does not perform de-isotoping or MS/MS identification, treat the class assignments as candidate-level annotations that feed downstream de-isotoping, annotation, and MS/MS confirmation workflows rather than final structural identifications. Compute the overlap between independently ranked lists (e.g., the prognostic and ICI-response Top-20 lists shared 3 of 20 features including lipid and carnitine members) to prioritise biologically coherent candidates. Validation is judged by whether the annotated classes reproduce the reported enrichment pattern—acyl-carnitines and LPC among the top-ranked features—and whether the identified features survive MS/MS confirmation.

## Related tools

- **mzLearn** (Produces the ion MS1 feature table (median rt/m/z with normalized intensities) that constitutes the ranked feature list to be annotated; does not itself perform de-isotoping or MS/MS identification, so its outputs serve as the input to downstream annotation workflows.) — https://github.com/ReviveMed/mzEmbed
- **SHAP** (Generates the Top-20 ranked feature-importance lists (from the fine-tuned prognostic and ICI-response models) whose ion features are then mapped to metabolite classes.)
- **VAE** (Fine-tuned model whose learned metabolite representations yield the feature-importance rankings (via SHAP) that are subsequently annotated; the same architecture is used to produce the prognostic and ICI-response Top-20 lists.) — https://github.com/ReviveMed/mzEmbed

## Examples

```
from mz_embed.pretrain.get_pretrain_encoder import get_pretrain_encoder; enc, Z, Y = get_pretrain_encoder('/path/to/out_finetune', '/path/to/finetune_csvs', '/path/to/encoder_out')
```

## Evaluation signals

- The annotated Top-20 list reproduces the reported class enrichment: acyl-carnitines (C6, C8, C12:1, C18:1) and LPC (C16:1) appear among the top-ranked features.
- Overlap between the prognostic and ICI-response Top-20 lists matches the expected 3 of 20 shared features, including the reported lipid and carnitine members.
- No ion feature in the annotated output is claimed to have a confirmed compound identity without a downstream MS/MS confirmation step.
- The feature matrix underlying the ranking has the mzLearn-defined shape of a two-dimensional table of features defined by median rt and m/z values with normalized intensities, not raw spectral peaks.
- Assigned classes are drawn from the article's explicitly named classes (acyl-carnitines, LPC/plasmalogen members) rather than invented categories.

## Limitations

- mzLearn outputs are limited to the ion MS1 level; it does not perform de-isotoping or MS/MS identification, and no MS/MS integration into pretraining is implemented, so annotations remain candidate-level until confirmed by MS/MS.
- The current pre-trained model was developed specifically for HILIC positive mode; other LC modes and modalities are not covered.
- Batch covariates or domain-invariant regularization are not yet incorporated into large-scale deployments, which may affect feature-level annotations across cohorts.

## Evidence

- [readme] Output: A two-dimensional table of detected features defined by median rt and m/z values, with normalized intensities across samples: [evidence span withheld — anchor status: not_found]

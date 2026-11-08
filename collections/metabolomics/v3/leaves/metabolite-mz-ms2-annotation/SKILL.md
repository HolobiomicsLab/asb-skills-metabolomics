---
name: metabolite-mz-ms2-annotation
description: Use when you have ranked m/z peaks from a feature-attribution step (e.g.
  ensemble_RISE.npy heatmap with contribution scores) and need to translate raw spectral
  signals into annotated metabolites before network or pathway analysis.
when_to_use_negative:
- Input is already a curated, identified metabolite feature table with compound names
  — annotation against HMDB is unnecessary
- The goal is conventional untargeted metabolomics peak picking and alignment across
  the whole dataset rather than attributing specific high-contribution signals from
  a trained model
- No MS2 spectral information or reference annotation database is available and only
  de novo unknown-signal handling is needed
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: DeepMSProfiler
  role: Generates the ensemble RISE feature heatmap whose high-contribution m/z peaks
    are the input to annotation
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: PIUMet
  role: Consumes the annotated metabolites as prizes to reconstruct the disease-related
    metabolite-protein network
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:930b993dfc38bcd52be05e238b408b193060884f760d5e1ff65752ef400bc53c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yjdeng9/DeepMSProfiler
  repo_url: https://github.com/yjdeng9/DeepMSProfiler
  tools:
  - DeepMSProfiler
  - MetaboAnalyst
  - PIUMet
  - Python (>= 3.6)
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolite-mz-ms2-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates high-contribution spectral peaks from the ensemble RISE feature heatmap by matching their m/z values and MS2 spectra against metabolite reference annotations (HMDB v3), converting deep-learning feature attribution into named metabolites that can seed downstream network inference.

## When to use

Use when you have ranked m/z peaks from a feature-attribution step (e.g. ensemble_RISE.npy heatmap with contribution scores) and need to translate raw spectral signals into annotated metabolites before network or pathway analysis.

## When NOT to use

- Input is already a curated, identified metabolite feature table with compound names — annotation against HMDB is unnecessary
- The goal is conventional untargeted metabolomics peak picking and alignment across the whole dataset rather than attributing specific high-contribution signals from a trained model
- No MS2 spectral information or reference annotation database is available and only de novo unknown-signal handling is needed

## Inputs

- ensemble RISE feature heatmap (ensemble_RISE.npy) from DeepMSProfiler run_feature
- Raw LC-MS spectral data (mzML or .npy format) containing m/z values and MS2 spectra
- HMDB v3 metabolite reference annotation database

## Outputs

- List of high-contribution m/z peaks (contribution score > 0.70) annotated with metabolite identities from HMDB v3
- Set of disease-associated metabolite annotations ready for use as PIUMet Steiner Forest prizes

## How to apply

First run DeepMSProfiler's run_feature step to produce the ensemble RISE heatmap (ensemble_RISE.npy), then rank spectral peaks by their contribution score and retain only peaks with contribution score > 0.70. Annotate each retained peak by matching its m/z value and its MS2 spectra against metabolite reference annotations in HMDB v3; peaks without reference matches remain usable downstream as unannotated/unknown metabolite signals rather than being discarded. The annotated metabolite list serves as the input 'prizes' for PIUMet's prize-collecting Steiner Forest network reconstruction. Rationale: the interpretability layer is what enables direct access to disease-related metabolite-protein networks without conventional full-dataset peak picking, alignment and annotation.

## Related tools

- **DeepMSProfiler** (Generates the ensemble RISE feature heatmap whose high-contribution m/z peaks are the input to annotation) — https://github.com/yjdeng9/DeepMSProfiler
- **PIUMet** (Consumes the annotated metabolites as prizes to reconstruct the disease-related metabolite-protein network)

## Examples

```
from DeepMSProfiler import *; run_feature(job_dir='DeepMSProfiler/example/out/jobs007'); show_feature(job_dir='DeepMSProfiler/example/out/jobs007',mode='ensemble')
```

## Evaluation signals

- Retained peak set is non-empty and every retained peak has contribution score > 0.70 from ensemble_RISE.npy
- Each annotation is supported by agreement of both m/z value and MS2 spectra against HMDB v3, not m/z alone
- Annotated metabolite list is compatible as PIUMet prize input (i.e. maps to HMDB v3 identifiers usable in the Steiner Forest step)
- Downstream PIUMet run on the annotated metabolites recovers the expected disease-specific subnetwork (e.g. LUAD network of 82 proteins and 121 metabolites in the reference case)

## Limitations

- The contribution-score cutoff of 0.70 and ranking of peaks depend on the ensemble RISE heatmap quality; an under-trained ensemble yields unstable annotations
- Unannotated (unknown) metabolites cannot be given compound identities and are instead carried forward as inferred signals associated with the classification
- Annotation is restricted by the coverage of HMDB v3 reference annotations; metabolites absent from the database remain unidentified
- Pre-trained model access for the reference workflow requires contacting the author

## Evidence

- [methods] Annotate the retained peaks by matching their m/z values and MS2 spectra against metabolite reference annotations (HMDB v3).: [evidence span withheld — anchor status: not_found]
- [methods] retaining only peaks with contribution score > 0.70: [evidence span withheld — anchor status: not_found]

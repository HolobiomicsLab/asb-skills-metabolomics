---
name: sample-metadata-interoperability-analysis
description: Use when evaluating how metabolomics repositories or metadata standardizers
  (e.g., ReDU, MetaboLights, Metabolomics Workbench, SMetaS) design sample-level metadata
  fields, to judge whether programmatic meta-analysis via string equality is feasible.
when_to_use_negative:
- The sample metadata are already expressed in fixed/constrained vocabularies, so
  no design-choice comparison or standardization-valence analysis is needed
- The question concerns spectral or feature-level data processing rather than sample-level
  metadata interoperability
- Only a single tool's metadata schema is in scope with no cross-tool comparison to
  perform
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: SMetaS
  role: Sample metadata standardizer whose fixed-vocabulary replacement of freetext
    terms provides the framing for judging which repository design choices support
    programmatic meta-analysis
  repo: https://github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Project name for the SMetaS implementation; documentation consulted during
    the review
  repo: https://github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:f5ad69780a759046e2d79c49de0fe1bebaf75b01be8db0384f33471b00f3ec0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-us/metadatastandardizer
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sample-metadata-interoperability-analysis

## Summary

A manual expert-review skill for extracting and classifying design choices in metabolomics sample-metadata tools and repositories (SMetaS, ReDU, Metabolomics Workbench, MetaboLights) as favorable or hindering for programmatic sample-level meta-analysis, where constrained vocabularies enable matching via string equality rather than entity recognition.

## When to use

Use when evaluating how metabolomics repositories or metadata standardizers (e.g., ReDU, MetaboLights, Metabolomics Workbench, SMetaS) design sample-level metadata fields, to judge whether programmatic meta-analysis via string equality is feasible.

## When NOT to use

- The sample metadata are already expressed in fixed/constrained vocabularies, so no design-choice comparison or standardization-valence analysis is needed
- The question concerns spectral or feature-level data processing rather than sample-level metadata interoperability
- Only a single tool's metadata schema is in scope with no cross-tool comparison to perform

## Inputs

- Source article text (e.g., SMetaS article, DOI 10.3390/metabo13080941) with passages describing tool design choices
- Task card specifying the research question and target tools (Metabolomics Workbench, ReDU, MetaboLights)

## Outputs

- tool_design_choices_comparison.csv with one row per tool-design-choice pair and columns: tool name, design choice description, valence (favorable/hindering)

## How to apply

Locate the passages in the source article (DOI 10.3390/metabo13080941) that discuss each tool in the context of programmatic sample-level meta-analysis. For each tool, extract the individual design choices the authors mention and classify each as favorable or hindering for that purpose, preserving the authors' wording where possible. Build a tabular structure with one row per tool-design-choice pair and columns for tool name, design choice description, and valence (favorable/hindering). Use the SMetaS framing as the analytical lens: fixed vocabularies replacing freetext terms (via synonym matching and typographical fixing in an n-grams/nearest neighbors model approach) are what make string-equality meta-analysis possible. Verify that each of the three tools (Metabolomics Workbench, ReDU, MetaboLights) is represented with its documented design choices before finalizing the output table.

## Related tools

- **SMetaS** (Sample metadata standardizer whose fixed-vocabulary replacement of freetext terms provides the framing for judging which repository design choices support programmatic meta-analysis) — https://github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Project name for the SMetaS implementation; documentation consulted during the review) — https://github.com/metabolomics-us/metadatastandardizer

## Evaluation signals

- Each of the three tools (Metabolomics Workbench, ReDU, MetaboLights) is represented in the output table with its documented design choices
- Every row carries an explicit valence classification (favorable/hindering) grounded in the authors' own statements
- Design choice descriptions preserve the authors' wording where possible rather than paraphrasing
- Each tool-design-choice pair appears as exactly one row with the three required columns (tool name, design choice description, valence)
- Claims about constrained vocabularies enabling string-equality meta-analysis trace to verbatim passages in the SMetaS article

## Limitations

- Classification of valence relies on manual expert review of the authors' statements; tools or design choices not discussed in the article cannot be classified
- The article's discussion contains no changelog, so version-specific behavior of the tools cannot be verified from this source
- The reference SMetaS deployment is documented as slow because of low i/o speed on the AWS instance, limiting interactive exploration during review

## Evidence

- [abstract] SMetaS replaces freetext sample metadata terms with terms from fixed vocabularies using synonym matching and typographical fixing in an n-grams/nearest neighbors model approach.: [evidence span withheld — anchor status: not_found]
- [abstract] Using SMetaS in frontends for metabolomic databases will increase the FAIRness of data for data reuse and finding datasets with comparable metadata.: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [readme] An example website is hosted at http://metasmetas.us-west-2.elasticbeanstalk.com/. This instance is slow because we chose to pay for low i/o speed on our AWS instance.: "This instance is slow because we chose to pay for low i/o speed on our AWS instance. The project can run much faster."

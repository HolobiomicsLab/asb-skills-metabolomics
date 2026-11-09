---
name: metabolomics-database-capability-assessment
description: Use when evaluating whether metabolomics data repositories support programmatic
  sample-level meta-analysis, e.g. comparing Metabolomics Workbench, ReDU and MetaboLights
  design choices.
when_to_use_negative:
- The goal is to standardize sample metadata itself rather than to assess repository
  design — that is the job of SMetaS, which replaces freetext terms with terms from
  fixed vocabularies via synonym matching and typographical fixing.
- Only a single repository is in scope with no comparative research question, so no
  favorable-versus-hindering classification across tools can be built.
- The source text does not name specific design choices for the tools under comparison,
  leaving no authors' wording to preserve in the table.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMetaS
  role: Source article's standardizer whose assessment of metabolomics databases (Workbench,
    ReDU, MetaboLights) provides the design-choice passages being classified; manual
    expert review of its text is the assessment method.
  repo: github.com/metabolomics-us/metadatastandardizer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:2a5c380feb861a068094a00c6c5839b621ad2097a52d912b31ea24c097472fcc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SMetaS
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolomics-database-capability-assessment

## Summary

A comparative literature-assessment skill that extracts the individual design choices of metabolomics data resources (here Metabolomics Workbench, ReDU and MetaboLights) from the SMetaS article (DOI 10.3390/metabo13080941) and classifies each as favorable or hindering for programmatic sample-level meta-analysis.

## When to use

Use when evaluating whether metabolomics data repositories support programmatic sample-level meta-analysis, e.g. comparing Metabolomics Workbench, ReDU and MetaboLights design choices.

## When NOT to use

- The goal is to standardize sample metadata itself rather than to assess repository design — that is the job of SMetaS, which replaces freetext terms with terms from fixed vocabularies via synonym matching and typographical fixing.
- Only a single repository is in scope with no comparative research question, so no favorable-versus-hindering classification across tools can be built.
- The source text does not name specific design choices for the tools under comparison, leaving no authors' wording to preserve in the table.

## Inputs

- Full text of the SMetaS article (DOI 10.3390/metabo13080941), specifically passages discussing Metabolomics Workbench, ReDU and MetaboLights
- Source cards specifying the research question and target tool set
- Repository documentation for discussed tools (e.g. metabolomics-us/metadatastandardizer README)

## Outputs

- tool_design_choices_comparison.csv with one row per tool-design-choice pair and columns: tool name, design choice description, valence (favorable/hindering)

## How to apply

Locate the passages in the SMetaS article (DOI 10.3390/metabo13080941) that discuss MS Workbench, ReDU and MetaboLights in the context of programmatic sample-level meta-analysis. For each tool, extract the individual design choices the authors mention and classify each as favorable or hindering for that purpose, e.g. ReDU's constrained vocabularies are favorable because they allow programmatic meta-analysis via string equality rather than entity recognition. Build a tabular structure with one row per tool-design-choice pair and columns for tool name, design choice description, and valence (favorable/hindering), preserving the authors' wording where possible. Write the table to tool_design_choices_comparison.csv and verify that each of the three tools is represented with its documented design choices before finalizing.

## Related tools

- **SMetaS** (Source article's standardizer whose assessment of metabolomics databases (Workbench, ReDU, MetaboLights) provides the design-choice passages being classified; manual expert review of its text is the assessment method.) — github.com/metabolomics-us/metadatastandardizer

## Examples

```
python -c "import csv; w=csv.writer(open('tool_design_choices_comparison.csv','w')); w.writerow(['tool','design_choice','valence']); w.writerows(rows)"
```

## Evaluation signals

- tool_design_choices_comparison.csv contains rows for all three tools: Metabolomics Workbench (MS Workbench), ReDU and MetaboLights.
- Every valence value in the table is restricted to the controlled labels 'favorable' or 'hindering'.
- Each design-choice description preserves the authors' wording from the article rather than paraphrase.
- The ReDU row captures the specific favorable finding that constrained vocabularies permit programmatic meta-analysis via string equality rather than entity recognition.
- Row count equals the number of distinct tool-design-choice pairs identified in the text (no tool-design pair appears twice).

## Limitations

- Assessment is limited to design choices the SMetaS authors explicitly document for the discussed tools; undocumented capabilities cannot be classified.
- The repository documentation for the associated tooling offers no changelog, so version-specific behavior of compared resources cannot be tracked from this context.
- Deployed reference instances may be slow; per the README, 'This instance is slow because we chose to pay for low i/o speed on our AWS instance.'

## Evidence

- [abstract] SMetaS context for the assessment: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [abstract] FAIRness rationale for metadata standardization across databases: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [readme] Deployed reference instance caveat: "An example website is hosted at http://metasmetas.us-west-2.elasticbeanstalk.com/. This instance is slow because we chose to pay for low i/o speed on our AWS instance."

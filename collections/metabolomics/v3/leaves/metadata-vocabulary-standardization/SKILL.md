---
name: metadata-vocabulary-standardization
description: Use when sample metadata records contain inconsistent, freetext descriptor
  terms that must be mapped to fixed vocabularies before database integration, cross-study
  comparison, or ontological parent aggregation.
when_to_use_negative:
- Input metadata already consist of terms from fixed vocabularies with no freetext
  variants — standardization would only risk remapping correct terms.
- The goal is capturing relationships beyond simple 'is a' links (e.g. co-cultures)
  — the fixed tabular representation cannot hold these; extend to a node/edge graph
  representation instead.
- Cross-study statistical meta-analysis on raw metabolomics data is needed — SMetaS
  standardizes metadata only, not metabolomics measurements.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0219
tools:
- name: SMetaS
  role: Containerized standardizer (classic database + API + frontend) that replaces
    freetext sample metadata terms with fixed-vocabulary terms via synonym matching
    and typographical fixing
  repo: github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Command-line/documentation-facing variant of the standardizer used to process
    metadata files against fixed vocabularies
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:26ef874ffe6383209b37c4863179b88bfc326ff83e47b2d7362cc830e8282d24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaS
  - SMetaS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metadata-vocabulary-standardization

## Summary

Standardize freetext metabolomics sample metadata by replacing freetext terms with terms from fixed vocabularies, using synonym matching and typographical fixing in an n-grams/nearest neighbors model approach (SMetaS/MetaS).

## When to use

Use when sample metadata records contain inconsistent, freetext descriptor terms that must be mapped to fixed vocabularies before database integration, cross-study comparison, or ontological parent aggregation.

## When NOT to use

- Input metadata already consist of terms from fixed vocabularies with no freetext variants — standardization would only risk remapping correct terms.
- The goal is capturing relationships beyond simple 'is a' links (e.g. co-cultures) — the fixed tabular representation cannot hold these; extend to a node/edge graph representation instead.
- Cross-study statistical meta-analysis on raw metabolomics data is needed — SMetaS standardizes metadata only, not metabolomics measurements.

## Inputs

- freetext sample metadata records (e.g. the SMetaS use-case sample metadata from the metadatastandardizer repository)
- fixed vocabularies/reference term lists used by SMetaS for synonym matching

## Outputs

- per-sample standardized metadata terms from fixed vocabularies
- intermediate artifacts for downstream extensions such as graph_storage.json (node/edge sample–descriptor graph) or ground_truth_verification.csv (parent-node aggregation checks)

## How to apply

Run SMetaS, which combines a classic database with an API and frontend in a containerized environment, over the freetext sample metadata records (e.g. the use-case data in the metadatastandardizer repository). Each freetext term is transformed by synonym matching against the fixed vocabularies and by typographical fixing in an n-grams/nearest neighbors model approach, chosen to maximize simplicity of the transformation. Record the per-sample standardized descriptor terms, which become the input for downstream analyses such as building node/edge graph representations of sample–descriptor relationships or aggregating samples under their highest ontological parent nodes. The output should preserve every original sample and descriptor term as either a standardized match or an explicit unmatched entry so nothing is silently dropped relative to the tabular input.

## Related tools

- **SMetaS** (Containerized standardizer (classic database + API + frontend) that replaces freetext sample metadata terms with fixed-vocabulary terms via synonym matching and typographical fixing) — github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Command-line/documentation-facing variant of the standardizer used to process metadata files against fixed vocabularies) — https://metabolomics-us.github.io/metadatastandardizer/

## Examples

```
docker run metabolomics-us/metadatastandardizer --input use_case_sample_metadata.csv --output standardized_metadata.json
```

## Evaluation signals

- Every freetext term from the tabular input is either mapped to a fixed-vocabulary term or explicitly preserved — no sample or descriptor is dropped relative to the input.
- Standardized outputs use terms from the reference fixed vocabularies only; spot-check known synonym pairs and typographical variants resolve to the same vocabulary term.
- Feeding standardized output to the highest-ontological-parent aggregation reproduces published ground truths (e.g. zero plant-kingdom samples annotated with cholesterol).
- Building the node/edge graph over the standardized terms yields 'is a' edges connecting every sample to its standardized descriptors, with no missing nodes.
- Running the containerized SMetaS instance (API or frontend) over the same input twice gives identical standardized mappings, since the transformation is designed to maximize simplicity.

## Limitations

- The fixed tabular representation limits how relationships between samples and descriptors are captured — richer relations (e.g. co-cultures beyond unique 'is a' links) require the graph-based storage extension, not the standardizer itself.
- The example hosted instance at metasmetas.us-west-2.elasticbeanstalk.com is slow because low i/o speed was chosen on the AWS instance; local/containerized execution is much faster.
- Transformation is designed to maximize simplicity, so ambiguous freetext terms may need manual curation after standardization.

## Evidence

- [abstract] SMetaS replaces freetext sample metadata terms with terms from fixed vocabularies using synonym matching and typographical fixing in an n-grams/nearest neighbors model approach.: [evidence span withheld — anchor status: not_found]
- [abstract] SMetaS combines a classic database with an API and frontend in a containerized environment.: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [abstract] FAIRness benefit of using the standardizer in database frontends.: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [readme] MetaS documentation location in the repository README.: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
- [readme] Hosted example instance is slow due to low AWS i/o speed.: "This instance is slow because we chose to pay for low i/o speed on our AWS instance. The project can run much faster."

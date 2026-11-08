---
name: sample-metadata-graph-storage
description: Use when sample metadata records from SMetaS/MetaS need relationships
  beyond tabular columns — e.g. co-culture links — that a fixed 'is a' table cannot
  express as first-class relations.
when_to_use_negative:
- The metadata relationships are adequately expressed by the existing fixed table
  of unique 'is a' links and no richer relations (e.g. co-cultures) are needed.
- Input data is raw, unstandardized freetext metadata that has not yet passed through
  SMetaS's vocabulary standardization step.
- The task is to run the standardization itself (n-grams/nearest-neighbors synonym
  fixing) rather than to restructure already-standardized records.
tools:
- name: SMetaS
  role: Provides the standardized sample metadata records and fixed-vocabulary descriptor
    terms that populate the graph nodes and 'is a' edges
  repo: https://github.com/metabolomics-us/metadatastandardizer
- name: MetaS
  role: Source project/API whose use-case metadata and documentation supply the input
    data for building the node/edge graph
  repo: https://metabolomics-us.github.io/metadatastandardizer/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo13080941
    title: 'SMetaS: A Sample Metadata Standardizer for Metabolomics'
  source_content_hash: sha256:8f21e1a2eb8489cf40b30100a599a501b8d63e002ba9f04d9e9e0591263caf85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabolomics-us/metadatastandardizer
  tools:
  - MetaS
  - SMetaS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# sample-metadata-graph-storage

## Summary

Replaces SMetaS's fixed tabular representation of sample metadata with a node/edge graph in which samples and standardized descriptor terms are nodes and relationships (including 'is a' and co-culture links) are first-class typed edges, serialized to graph_storage.json.

## When to use

Use when sample metadata records from SMetaS/MetaS need relationships beyond tabular columns — e.g. co-culture links — that a fixed 'is a' table cannot express as first-class relations.

## When NOT to use

- The metadata relationships are adequately expressed by the existing fixed table of unique 'is a' links and no richer relations (e.g. co-cultures) are needed.
- Input data is raw, unstandardized freetext metadata that has not yet passed through SMetaS's vocabulary standardization step.
- The task is to run the standardization itself (n-grams/nearest-neighbors synonym fixing) rather than to restructure already-standardized records.

## Inputs

- SMetaS use-case sample metadata records from the metadatastandardizer repository example/use-case data
- Per-sample descriptor terms standardized by SMetaS (synonym-matched, typo-fixed vocabulary terms)
- Additional relation definitions (e.g. co-culture pairs between samples)

## Outputs

- graph_storage.json containing node and edge lists
- Typed 'is a' edges linking each sample to its standardized descriptors
- Separately typed edges for additional relations (e.g. co-culture)

## How to apply

Load the SMetaS use-case sample metadata records and extract each sample's descriptor terms already standardized through SMetaS's fixed-vocabulary mapping (synonym matching and typographical fixing via the n-grams/nearest-neighbors approach). Build a graph where every sample and every descriptor is a node, and each sample–descriptor 'is a' relation is a typed edge, represented programmatically as a Python dict/JSON node-edge structure. Add additional relationship types (e.g. co-culture between samples) as separately typed edges so relations are first-class rather than implied by tabular columns. Serialize the node and edge lists to graph_storage.json. The rationale is that the tabular limitation motivates graph storage: edges programmatically store more detailed sample–descriptor relationships than unique 'is a' links.

## Related tools

- **SMetaS** (Provides the standardized sample metadata records and fixed-vocabulary descriptor terms that populate the graph nodes and 'is a' edges) — https://github.com/metabolomics-us/metadatastandardizer
- **MetaS** (Source project/API whose use-case metadata and documentation supply the input data for building the node/edge graph) — https://metabolomics-us.github.io/metadatastandardizer/

## Examples

```
import json; json.dump({"nodes": nodes, "edges": edges}, open("graph_storage.json", "w"))
```

## Evaluation signals

- Every use-case sample is connected to its standardized descriptors by an explicitly typed 'is a' edge in graph_storage.json.
- Additional relation edges (e.g. co-culture) are present and carry their own distinct edge type, not folded into 'is a'.
- No sample or descriptor node is dropped relative to the tabular input — node counts match the source records.
- The serialized JSON parses as a valid node/edge structure (Python dict/JSON) with both a nodes list and an edges list.
- Every edge's endpoints resolve to existing node identifiers in the graph (no dangling references).

## Limitations

- The paper only proposes the graph representation as a way to overcome the tabular limitation; it does not provide a finalized graph schema or changelog.
- SMetaS's demonstration instance is slow by design (low i/o speed on the AWS instance), so scaling the graph build over large metadata sets may require a faster local deployment.
- Only relations illustrated in the use case (unique 'is a' links plus co-culture examples) are described; other relation types would need their own typed-edge definitions.

## Evidence

- [abstract] SMetaS combines a classic database with an API and frontend in a containerized environment, providing the standardized records that populate the graph.: "SMetaS (Sample Metadata Standardizer) combines a classic database with an API and frontend and is provided in a containerized environment."
- [abstract] The standardization feeding the graph relies on synonym matching and typographical fixing via the n-grams/nearest-neighbors approach.: "This transformation process is designed to maximize simplicity and is guided by, among other strategies, synonym matching and typographical fixing in an n-grams/nearest neighbors model approach."
- [abstract] Graph storage of standardized metadata supports the FAIRness goal that motivates restructuring the tabular representation.: "Using such a tool in frontends for metabolomic databases will dramatically increase the FAIRness (Findability, Accessibility, Interoperability, and Reusability) of data"
- [readme] The MetaS/MetaS project README points to the official documentation and hosted example used to obtain use-case metadata.: "Extensive documentation is available at https://metabolomics-us.github.io/metadatastandardizer/"
- [readme] The hosted example instance is slow due to low i/o speed on the AWS instance, a deployment caveat for pulling metadata from it.: "An example website is hosted at http://metasmetas.us-west-2.elasticbeanstalk.com/. This instance is slow because we chose to pay for low i/o speed on our AWS instance."

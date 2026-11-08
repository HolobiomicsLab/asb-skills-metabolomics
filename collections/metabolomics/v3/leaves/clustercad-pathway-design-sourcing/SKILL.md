---
name: clustercad-pathway-design-sourcing
description: Use when designing or redesigning biosynthetic pathways computationally
  and you need verified BGC sequence/annotation data of known function as a starting
  point, or when confirming that a pathway-design tool sources its BGC data from MIBiG.
when_to_use_negative:
- The goal is mining candidate/novel BGCs from genomes or metagenomes rather than
  retrieving BGCs of known function — use antiSMASH or ClusterFinder on assemblies
  instead of the MIBiG reference set.
- The task requires a broad catalogue of candidate BGCs regardless of known products
  — databases like IMG-ABC or antiSMASH-DB store many thousands of candidate clusters,
  whereas MIBiG holds only curated clusters linked to known chemical structures.
- You need to deposit a new BGC entry rather than consume existing ones — that follows
  the MIBiG accession-request and extended entry-form workflow, not ClusterCAD sourcing.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3174
tools:
- name: MIBiG Repository
  role: Curated repository of BGCs with known function whose sequence and annotation
    data are consumed for pathway design
  repo: https://mibig.secondarymetabolites.org/
- name: ClusterCAD
  role: Computer-aided pathway design tool that sources BGC data from MIBiG
- name: GitHub
  role: Distribution point for MIBiG source code, JSON-formatted data standard, and
    SQL schema
  repo: https://github.com/mibig-secmet
- name: PostgreSQL
  role: Underlying relational database accessed through the MIBiG REST-like web API
    when retrieving BGC records
  repo: https://www.postgresql.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz882
    title: 'MIBiG 2.0: a repository for biosynthetic gene clusters of known function'
  source_content_hash: sha256:013fec8466ffb67bec8499180c9dcd7b4ca1b78e36cbd20453680158d50fc087
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterCAD
  - GitHub
  - MIBiG Repository
  - PostgreSQL
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# clustercad-pathway-design-sourcing

## Summary

Sourcing curated BGC sequence and annotation data from the MIBiG repository to support computer-aided design of new biochemical pathways via ClusterCAD. MIBiG 2.0 provides JSON-formatted, manually curated entries of BGCs with known chemical products as the substrate for such design workflows.

## When to use

Use when designing or redesigning biosynthetic pathways computationally and you need verified BGC sequence/annotation data of known function as a starting point, or when confirming that a pathway-design tool sources its BGC data from MIBiG.

## When NOT to use

- The goal is mining candidate/novel BGCs from genomes or metagenomes rather than retrieving BGCs of known function — use antiSMASH or ClusterFinder on assemblies instead of the MIBiG reference set.
- The task requires a broad catalogue of candidate BGCs regardless of known products — databases like IMG-ABC or antiSMASH-DB store many thousands of candidate clusters, whereas MIBiG holds only curated clusters linked to known chemical structures.
- You need to deposit a new BGC entry rather than consume existing ones — that follows the MIBiG accession-request and extended entry-form workflow, not ClusterCAD sourcing.

## Inputs

- MIBiG 2.0 JSON-formatted BGC entries (2,021 curated BGCs) from https://mibig.secondarymetabolites.org/ or GitHub (https://github.com/mibig-secmet)
- MIBiG REST API responses (https://github.com/mibig-secmet/mibig-api/) backed by the PostgreSQL database
- ClusterCAD tool configuration

## Outputs

- Structured CSV table of BGC metadata (accession, biosynthetic class, taxonomic identifier, product annotation) sourced from MIBiG for ClusterCAD pathway design
- Confirmation record that ClusterCAD sources BGC sequence and annotation data from the MIBiG repository

## How to apply

Download MIBiG 2.0 data from the official GitHub repository (https://github.com/mibig-secmet) or via the MIBiG REST-like web API (https://github.com/mibig-secmet/mibig-api/), which handles access to the underlying PostgreSQL database. Parse the JSON-formatted BGC entries — which follow the JSON schema with embedded validation and dependency rules — to extract biosynthetic class, taxonomic identifier, and product annotation fields. Load the ClusterCAD tool configuration and confirm that it sources BGC sequence and annotation data directly from the MIBiG repository, since MIBiG entries are manually curated and associated with molecules of known chemical structure, which is the prerequisite for using them as a starting point for computer-aided design of new biochemical pathways. Collate the sourcing confirmation and the extracted BGC metadata into a structured CSV table for downstream pathway-design use.

## Related tools

- **MIBiG Repository** (Curated repository of BGCs with known function whose sequence and annotation data are consumed for pathway design) — https://mibig.secondarymetabolites.org/
- **ClusterCAD** (Computer-aided pathway design tool that sources BGC data from MIBiG)
- **GitHub** (Distribution point for MIBiG source code, JSON-formatted data standard, and SQL schema) — https://github.com/mibig-secmet
- **PostgreSQL** (Underlying relational database accessed through the MIBiG REST-like web API when retrieving BGC records) — https://www.postgresql.org/

## Examples

```
curl -s https://mibig.secondarymetabolites.org/api/v1/entries | python -c "import sys,json; rows=json.load(sys.stdin); print(len(rows), 'MIBiG entries sourced for ClusterCAD')"
```

## Evaluation signals

- Each parsed BGC entry validates against the MIBiG JSON schema, including its embedded validation and dependency rules.
- The extracted record count matches the MIBiG 2.0 release size (2,021 curated BGCs, expanded from 1,170 by 851 new entries).
- ClusterCAD configuration demonstrably points to MIBiG as its BGC data source, matching the article's statement that ClusterCAD 'sources BGC data from MIBiG'.
- Retrieved entries carry non-empty biosynthetic class, taxonomic identifier, and product annotation fields and cross-links (e.g. Natural Products Atlas) where expected.
- The collated CSV contains one row per MIBiG accession with no duplicate entries (MIBiG 2.0 curation removed duplicates).

## Limitations

- MIBiG contains only BGCs associated with molecules of known chemical structure; it is not a catalogue of candidate BGCs from genome mining.
- ClusterCAD usage depends on the MIBiG API server, which requires Go 1.11.x for development and is licensed under Apache License version 2.
- Data quality depends on ongoing manual curation (Annotathons, re-annotation of the entire database); no changelog was found for the repository, so version drift must be checked against the release.

## Evidence

- [intro] ClusterCAD sources BGC data from MIBiG: "the ClusterCAD tool (9), which sources BGC data from MIBiG"
- [intro] MIBiG as starting point for computer-aided design: "a starting point for the computer-aided design of new biochemical pathways"
- [intro] MIBiG as central reference database of known function: "the MIBiG repository has become a central reference database for BGCs of known function"
- [intro] REST API backed by PostgreSQL: "A REST-like web API (https://github.com/mibig-secmet/mibig-api/) handles access to the underlying PostgreSQL"
- [readme] API server implementation requirements: "Make sure you have a recent version of Go (1.11.x), as the dependency handling requires that."
- [intro] Standardized deposition of BGCs with known structures: "This calls for a standardized deposition and extraction of BGCs associated with molecules of known chemical structure"

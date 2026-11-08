---
name: metabolite-identifier-translation
description: Use when metabolomics data contains metabolite names or identifiers in
  heterogeneous database formats and a downstream method (e.g. network construction)
  requires KEGG or PubChem CIDs, especially for GC-TOF-MS metabolite lists including
  unknowns.
when_to_use_negative:
- Input identifiers are already in the target format required by downstream tools
  (e.g. metabolites already carry KEGG or PubChem CIDs) — translation adds latency
  and possible resolution errors
- The Chemical Translation Service API is unavailable — CTSgetR explicitly stops when
  https://cts.fiehnlab.ucdavis.edu/services is unreachable
- The task requires identifier mapping among metabolites outside the >200 databases
  supported by CTS (translation will silently return NA for unsupported identifier
  types)
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: CTSgetR
  role: R interface to the Chemical Translation Service used to perform the identifier
    translation; called directly in R or deployed as a dockerized opencpu API
  repo: https://github.com/dgrapov/CTSgetR
- name: Chemical Translation System
  role: Backend web service that resolves metabolite identifiers between >200 common
    biochemical databases; consumes CTSgetR requests
- name: MetaMapR
  role: Downstream consumer of the translated KEGG/PubChem CIDs, which it requires
    to calculate biochemical and structural similarity networks
  repo: https://github.com/dgrapov/MetaMapR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:2df05fdb3c150eb413c2efe263b9835e41a240e313c954ea676cc51aca432242
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/CTSgetR
  tools:
  - CTSgetR
  - Chemical Translation System
  - MetaMapR
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolite-identifier-translation

## Summary

Translate metabolite names and identifiers (e.g. chemical names, BinBase-derived IDs, KEGG, PubChem CIDs, InChIKey, HMDB) between >200 biochemical databases using CTSgetR, the R interface to the Chemical Translation System, so downstream network calculations can resolve KEGG and PubChem references.

## When to use

Use when metabolomics data contains metabolite names or identifiers in heterogeneous database formats and a downstream method (e.g. network construction) requires KEGG or PubChem CIDs, especially for GC-TOF-MS metabolite lists including unknowns.

## When NOT to use

- Input identifiers are already in the target format required by downstream tools (e.g. metabolites already carry KEGG or PubChem CIDs) — translation adds latency and possible resolution errors
- The Chemical Translation Service API is unavailable — CTSgetR explicitly stops when https://cts.fiehnlab.ucdavis.edu/services is unreachable
- The task requires identifier mapping among metabolites outside the >200 databases supported by CTS (translation will silently return NA for unsupported identifier types)

## Inputs

- Vector or data.frame of metabolite identifiers or chemical names (e.g. 'alanine', 'lactic acid', HMDB IDs, KEGG IDs)
- Source database designation ('from', e.g. 'Chemical Name', 'KEGG', 'Human Metabolome Database')
- Target database designation(s) ('to', e.g. 'PubChem CID', 'KEGG', 'InChIKey', 'Human Metabolome Database')

## Outputs

- data.frame mapping each input id to translated identifiers (e.g. InChIKey, KEGG C-number, PubChem CID, HMDB accession)
- NA values for ids that cannot be translated

## How to apply

Install CTSgetR from GitHub (install_github('dgrapov/CTSgetR')) and first confirm the CTS API at https://cts.fiehnlab.ucdavis.edu/services is reachable before translating. Define the input identifiers and specify 'from'/'to' database pairs — e.g. 'Chemical Name' to 'PubChem CID' and 'KEGG' — using valid source databases listed by valid_from() (BioCyc, CAS, ChEBI, HMDB, InChIKey, KEGG, LipidMAPS, PubChem CID, etc.). For simple queries pass vectors of ids with a single from/to; for complex many-to-many queries build a data.frame of id/from/to rows, split by 'from', and map CTSgetR over each subset. Initialize a local sqlite cache (init_CTSgetR_db) to speed routine translations, and rely on translation through InChIKey with biological/popularity scoring to disambiguate chemical names. Untranslatable entries (e.g. a nonexistent name like 'foo') return NA rather than errors, so keep NA rows for downstream filtering. In the MetaMapR workflow this step precedes biochemical reaction (KEGG RPAIR) and structural similarity (PubChem fingerprint) network generation.

## Related tools

- **CTSgetR** (R interface to the Chemical Translation Service used to perform the identifier translation; called directly in R or deployed as a dockerized opencpu API) — https://github.com/dgrapov/CTSgetR
- **Chemical Translation System** (Backend web service that resolves metabolite identifiers between >200 common biochemical databases; consumes CTSgetR requests)
- **MetaMapR** (Downstream consumer of the translated KEGG/PubChem CIDs, which it requires to calculate biochemical and structural similarity networks) — https://github.com/dgrapov/MetaMapR

## Examples

```
library(CTSgetR); ids <- c('alanine','lactic acid'); CTSgetR(ids, from='Chemical Name', to=c('PubChem CID','KEGG','Human Metabolome Database'), db_name='ctsgetr.sqlite')
```

## Evaluation signals

- Known-positive control: 'alanine' (Chemical Name) must translate to InChIKey QNAYBMKLOCPYGJ-REOHCLBHSA-N, HMDB0000161, KEGG C00041, PubChem CID 5950
- 'lactic acid' must translate to InChIKey JVTAAEKCZFNVCJ-UHFFFAOYSA-N, HMDB0144295, KEGG C01432, PubChem CID 19789253
- Invalid input names (e.g. 'foo') return NA in all target columns rather than throwing errors — verify NA-handling downstream
- API heartbeat endpoint (http://localhost/ocpu/library/CTSgetR/R/heartbeat) or GET on https://cts.fiehnlab.ucdavis.edu/services returns Success before running translations
- Local sqlite cache (db_name, e.g. 'ctsgetr.sqlite') exists and db_stats() reports non-trivial content after init_CTSgetR_db

## Limitations

- Translation of free-text chemical names is inherently hard; CTSgetR mitigates via InChIKey conversion plus biological/popularity scoring, but ambiguous names may still resolve incorrectly
- Nonexistent or unsupported identifiers return NA (e.g. 'foo' yields NA for PubChem CID, KEGG, HMDB), so missing coverage must be handled explicitly
- The CTS web service must be online; CTSgetR deliberately stops if https://cts.fiehnlab.ucdavis.edu/services is down
- Identifier translation is optional in MetaMapR and is only needed when user ids are not already KEGG or PubChem CIDs

## Evidence

- [methods] Identifier Translation can be optionally used to map user metabolite names or identifiers to KEGG or Pub Chem CIDs required to calculate biochemical and chemical similarity networks: [evidence span withheld — anchor status: not_found]
- [readme] CTSgetR provides a consitent interface to translation of chemical names and over 200 database identifiers including InChIKey, HMDB, KEGG and PubChem.: [evidence span withheld — anchor status: external_doc]
- [readme] Use CTSgetR to robustly translate chemical names to other identifiers through conversion to InChIKey, biological or popularity scoring and translation to over 200 biological database identifiers.: [evidence span withheld — anchor status: external_doc]

---
name: molecular-fingerprint-retrieval
description: Use when a metabolomics network analysis requires structural similarity
  links between compounds — e.g. after translating metabolite names to PubChem CIDs
  and needing binary substructure fingerprint bit vectors for Tanimoto-based edge
  weights.
when_to_use_negative:
- The goal is biochemical transformation edges rather than structural similarity —
  use KEGG RPAIR substrate-product pairs instead
- Compounds lack structural annotation (unknowns) with no CID or fingerprint available
  — mass spectral similarity is the alternative route in MetaMapR for structurally
  unknown metabolites
- The input already consists of fingerprint bit vectors or precomputed similarity
  scores, making retrieval redundant
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3336
tools:
- name: CTSgetR
  role: Translates user metabolite identifiers (e.g. Chemical Name, KEGG, HMDB) to
    PubChem CIDs required for fingerprint retrieval, with a local sqlite cache for
    routine queries
  repo: https://github.com/dgrapov/CTSgetR
- name: ChemmineR
  role: R package used to retrieve PubChem Substructure Fingerprints for each CID
    via the PubChem Power User Gateway (PUG)
- name: MetaMapR
  role: Consumes the fingerprint-based structural similarity edges (Tanimoto >0.7)
    as one network layer alongside biochemical, spectral, and correlation networks
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Implementation language for the identifier translation, fingerprint retrieval,
    and Tanimoto computation steps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:7ca1edf1a339a30226d524f2b707eb79f12fc31f40cf52222cf852a0f1be2a1e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/CTSgetR
  tools:
  - CTSgetR
  - ChemmineR
  - MetaMapR
  - PubChem Substructure Fingerprints
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Molecular-fingerprint-retrieval

## Summary

Retrieve PubChem Substructure Fingerprints for each compound (PubChem CID) and compute pairwise Tanimoto coefficients so that structural similarity edges can be filtered and exported for metabolomic network reconstruction, as done in MetaMapR.

## When to use

Use when a metabolomics network analysis requires structural similarity links between compounds — e.g. after translating metabolite names to PubChem CIDs and needing binary substructure fingerprint bit vectors for Tanimoto-based edge weights.

## When NOT to use

- The goal is biochemical transformation edges rather than structural similarity — use KEGG RPAIR substrate-product pairs instead
- Compounds lack structural annotation (unknowns) with no CID or fingerprint available — mass spectral similarity is the alternative route in MetaMapR for structurally unknown metabolites
- The input already consists of fingerprint bit vectors or precomputed similarity scores, making retrieval redundant

## Inputs

- Metabolite identifier list (CSV or pasted names/IDs, e.g. chemical names, KEGG IDs)
- PubChem CIDs for annotated compounds
- PubChem Substructure Fingerprints (binary bit vectors, ftp://ftp.ncbi.nlm.nih.gov/pubchem/specifications/pubchem_fingerprints.txt)

## Outputs

- Pairwise Tanimoto similarity scores (range 0-1) between all compound pairs
- Structural similarity edge list (CSV) restricted to Tanimoto score >0.7, importable into Cytoscape

## How to apply

First map user metabolite identifiers (names, KEGG, HMDB) to PubChem CIDs via CTSgetR, the R interface to the Chemical Translation System, which supports >200 database identifier types. Then retrieve the PubChem Substructure Fingerprints for each CID using the R package ChemmineR through the PubChem Power User Gateway (PUG). Compute all pairwise Tanimoto coefficients between the binary fingerprint bit vectors; scores are bounded in [0,1]. Retain structural similarity edges exceeding the threshold of Tanimoto score >0.7 (as used for the T1D NOD mouse network in MetaMapR), and export the retained edge list as a CSV suitable for Cytoscape import.

## Related tools

- **CTSgetR** (Translates user metabolite identifiers (e.g. Chemical Name, KEGG, HMDB) to PubChem CIDs required for fingerprint retrieval, with a local sqlite cache for routine queries) — https://github.com/dgrapov/CTSgetR
- **ChemmineR** (R package used to retrieve PubChem Substructure Fingerprints for each CID via the PubChem Power User Gateway (PUG))
- **MetaMapR** (Consumes the fingerprint-based structural similarity edges (Tanimoto >0.7) as one network layer alongside biochemical, spectral, and correlation networks) — https://github.com/dgrapov/MetaMapR
- **R** (Implementation language for the identifier translation, fingerprint retrieval, and Tanimoto computation steps)

## Examples

```
library(CTSgetR); CTSgetR(c("alanine","lactic acid"), from="Chemical Name", to="PubChem CID", db_name="ctsgetr.sqlite")
```

## Evaluation signals

- Every edge weight in the exported CSV is a Tanimoto score strictly within (0.7, 1], consistent with the 0-1 similarity bounds
- Number of compounds with fingerprints equals the number of CIDs successfully resolved by CTSgetR; unmatched identifiers return NA and should be excluded
- Edge list is a well-formed CSV importable into Cytoscape with source, target, and weight columns
- Spot-check a known structurally similar pair (e.g. glucose and 1,5-anhydroglucitol, reported as structurally similar in the T1D results) to confirm a high Tanimoto score

## Limitations

- Fingerprint retrieval depends on the CTS API (https://cts.fiehnlab.ucdavis.edu) being reachable; the CTSgetR README warns the service may be down
- Compounds without PubChem CIDs or structural annotations cannot get fingerprints; MetaMapR falls back to mass spectral similarity for such unknowns
- KEGG database limitations on transformation directionality compound sparsity for metabolites not covered by structural data

## Evidence

- [methods] Structural Similarity Networks are determined based on similarities between Pub Chem Substructure Fingerprints: [evidence span withheld — anchor status: not_found]
- [readme] CTSgetR provides a consitent interface to translation of chemical names and over 200 database identifiers including `InChIKey`, `HMDB`, `KEGG` and `PubChem`.: [evidence span withheld — anchor status: not_found]
- [other] Retrieve the PubChem Substructure Fingerprints for each PubChem CID using the R package ChemmineR with the PubChem Power User Gateway (PUG).: [evidence span withheld — anchor status: not_found]

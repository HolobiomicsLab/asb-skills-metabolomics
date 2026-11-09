---
name: tanimoto-structural-similarity-calculation
description: Use when building structural similarity networks from a metabolite list
  (e.g. GC-TOF-MS plasma metabolites) where biochemical pathway knowledge is missing
  and edges between structurally similar compounds are needed.
when_to_use_negative:
- The goal is biochemical reaction (KEGG RPAIR substrate-product) edges rather than
  structural similarity edges
- Metabolites lack structural annotation (unknowns) — use mass spectral (cosine correlation
  >0.7) or empirical correlation networks instead, since fingerprints cannot be computed
  without structures
- Edge weights are already provided similarity scores and only thresholding/export
  is required
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: CTSgetR
  role: Translates user metabolite identifiers to PubChem CIDs via the Chemical Translation
    Service, a prerequisite for fingerprint retrieval
  repo: https://github.com/dgrapov/CTSgetR
- name: ChemmineR
  role: R package used to retrieve PubChem Substructure Fingerprints for each CID
    from the PubChem PUG gateway
- name: MetaMapR
  role: Integrates the structural similarity network with biochemical, spectral, and
    empirical networks; applies the Tanimoto >0.7 threshold and exports the edge list
  repo: https://github.com/dgrapov/MetaMapR
- name: R
  role: Implementation language for the fingerprint retrieval, Tanimoto computation,
    and CSV export steps
  repo: http://cran.us.r-project.org/
- name: Cytoscape
  role: Downstream import target for the exported structural similarity edge list
    and node attribute visualization
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btv194
    title: 'MetaMapR: pathway independent metabolomic network analysis incorporating
      unknowns'
  source_content_hash: sha256:4f5776cdb0f154e1102074e731bf951a9085a32dabbf8adf30b6c50050e44dfb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dgrapov/CTSgetR
  tools:
  - CTSgetR
  - ChemmineR
  - Cytoscape
  - MetaMapR
  - PubChem Substructure Fingerprints
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# tanimoto-structural-similarity-calculation

## Summary

Computes pairwise Tanimoto structural similarity edges between metabolites from PubChem Substructure Fingerprints, retaining edges with Tanimoto score >0.7 as used in the MetaMapR T1D NOD mouse metabolomics network.

## When to use

Use when building structural similarity networks from a metabolite list (e.g. GC-TOF-MS plasma metabolites) where biochemical pathway knowledge is missing and edges between structurally similar compounds are needed.

## When NOT to use

- The goal is biochemical reaction (KEGG RPAIR substrate-product) edges rather than structural similarity edges
- Metabolites lack structural annotation (unknowns) — use mass spectral (cosine correlation >0.7) or empirical correlation networks instead, since fingerprints cannot be computed without structures
- Edge weights are already provided similarity scores and only thresholding/export is required

## Inputs

- Metabolite identifier list (CSV file or pasted identifiers, e.g. metabolite names or KEGG/HMDB IDs)
- PubChem CIDs obtained via CTSgetR identifier translation
- PubChem Substructure Fingerprints retrieved per CID

## Outputs

- Structural similarity network edge list as a comma separated value (.csv) file where every edge weight is a Tanimoto score in the range (0.7, 1]
- Network importable into Cytoscape for visualization with node attribute mapping

## How to apply

First translate user metabolite identifiers to PubChem CIDs via CTSgetR, the R interface to the Chemical Translation Service, which supports translation between >200 biochemical databases. Retrieve PubChem Substructure Fingerprints for each CID from the PubChem Power User Gateway (PUG) using the R package ChemmineR. Compute pairwise Tanimoto coefficients between the binary fingerprint bit vectors of all compound pairs; scores are bounded between 0 and 1. Retain only edges whose Tanimoto score exceeds the threshold of >0.7, as applied for the T1D NOD mouse network. Export the retained edge list as a CSV file suitable for import into Cytoscape; in MetaMapR, structural edges rank below biochemical (KEGG RPAIR) edges when the unique edges option is enabled.

## Related tools

- **CTSgetR** (Translates user metabolite identifiers to PubChem CIDs via the Chemical Translation Service, a prerequisite for fingerprint retrieval) — https://github.com/dgrapov/CTSgetR
- **ChemmineR** (R package used to retrieve PubChem Substructure Fingerprints for each CID from the PubChem PUG gateway)
- **MetaMapR** (Integrates the structural similarity network with biochemical, spectral, and empirical networks; applies the Tanimoto >0.7 threshold and exports the edge list) — https://github.com/dgrapov/MetaMapR
- **R** (Implementation language for the fingerprint retrieval, Tanimoto computation, and CSV export steps) — http://cran.us.r-project.org/
- **Cytoscape** (Downstream import target for the exported structural similarity edge list and node attribute visualization)

## Examples

```
library(CTSgetR); cids <- CTSgetR(id, from="Chemical Name", to="PubChem CID", db_name="ctsgetr.sqlite")
```

## Evaluation signals

- Every edge weight in the exported CSV is a Tanimoto score in the range (0.7, 1], consistent with the 0-1 similarity score bounds
- Edge list is a well-formed CSV parseable by standard CSV readers with consistent columns
- All compounds in the edge list have successfully resolved PubChem CIDs (identifiers translated via CTSgetR without NA mappings)
- Network density is enriched relative to a pathway-limited (KEGG-only) representation, reflecting MetaMapR's rationale of avoiding sparse disconnected graphs
- Applying MetaMapR's unique-edges hierarchy, retained structural edges do not duplicate compounds already connected by biochemical (KEGG RPAIR) edges

## Limitations

- Structural similarity cannot be computed for molecules with unknown structures; MetaMapR extends analysis to such unknowns via mass spectral similarity instead
- The Tanimoto >0.7 threshold was set empirically for the T1D NOD mouse figures and may need retuning for other datasets
- Fingerprint retrieval depends on identifier translation succeeding; unmatched identifiers (e.g. 'foo' in CTSgetR examples) return NA and drop out of the network
- CTS availability matters: CTSgetR requires the Chemical Translation Service API to be reachable

## Evidence

- [methods] Fingerprint basis quote: [evidence span withheld — anchor status: not_found]
- [readme] CTSgetR readme quote: [evidence span withheld — anchor status: not_found]
- [readme] Unique edges hierarchy quote: [evidence span withheld — anchor status: external_doc]

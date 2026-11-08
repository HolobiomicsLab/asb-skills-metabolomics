---
name: chemspider-reference-count-retrieval
description: 'Use when ranking MetFrag2.2 candidate structures needs non-spectral
  evidence: ChemSpider reference/data-source/patent counts per candidate must be fetched
  via web services and normalized across the candidate list.'
when_to_use_negative:
- Candidates were retrieved from a database with no ChemSpider mapping (e.g. KEGG-only
  or local SDF without ChemSpider IDs), so no reference counts can be fetched
- The analysis relies only on HR-MS/MS fragment scores and no reference/retention
  information is desired (MetFrag2.2 with PubChem alone ranked only 30 Top 1)
- Offline/local database mode (LocalPubChemDatabase settings) where web-service retrieval
  of reference counts is not intended
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: MetFrag2.2
  role: Consumes the normalized ChemSpider combined reference score as a weighted
    scoring term during candidate ranking
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: ChemSpider
  role: Web-service source of the five per-candidate reference, data source and patent
    counts
- name: metfRag
  role: R interface to MetFrag that can be used to drive candidate retrieval and scoring
    workflows
  repo: http://ipb-halle.github.io/MetFrag/projects/metfragr/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:03c954702aeb6bceada757a4d882a56d684876b1fc6cb7b1971dcaaacc2d799e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - ChemSpider
  - MetFrag2.2
  - metfRag
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chemspider-reference-count-retrieval

## Summary

Retrieves five per-candidate reference counts from ChemSpider web services and combines them into a normalized ChemSpider combined reference score (S_C_CCR) used as a scoring term in MetFrag2.2 candidate ranking, improving identification when spectral libraries are sparse.

## When to use

Use when ranking MetFrag2.2 candidate structures needs non-spectral evidence: ChemSpider reference/data-source/patent counts per candidate must be fetched via web services and normalized across the candidate list.

## When NOT to use

- Candidates were retrieved from a database with no ChemSpider mapping (e.g. KEGG-only or local SDF without ChemSpider IDs), so no reference counts can be fetched
- The analysis relies only on HR-MS/MS fragment scores and no reference/retention information is desired (MetFrag2.2 with PubChem alone ranked only 30 Top 1)
- Offline/local database mode (LocalPubChemDatabase settings) where web-service retrieval of reference counts is not intended

## Inputs

- Candidate structure list (e.g. from ChemSpider, PubChem, KEGG or user-supplied SDF) with per-candidate ChemSpider identifiers
- ChemSpider web services access (token, e.g. ChemSpiderToken in settings.properties)
- MetFrag2.2 parameter file with scoring weights (ωFrag+ωRT+ωRefs=1)

## Outputs

- Per-candidate five ChemSpider reference counts
- Normalized ChemSpider combined reference score S_C_CCR per candidate
- Tab-separated results table with S_C_CCR values and contributing counts

## How to apply

For each candidate structure in the retrieved list, query the ChemSpider web services to obtain five reference-related counts (references, data sources and patent information, as described in Eqs. 5-8 of the MetFrag relaunched paper). Combine these counts into a cumulative ChemSpider combined reference term N_C_CCR, then normalize it across the candidate list as S_C_CCR = N_C_CCR / max over all candidates, so the term is comparable within the candidate set. Supply S_C_CCR as a scoring term alongside the in silico fragment score (with weights ωFrag, ωRT, ωRefs summing to 1) in MetFrag2.2. Write per-candidate S_C_CCR values with their contributing counts to a tab-separated results table. The rationale is that reference and data-source coverage acts as orthogonal evidence to HR-MS/MS fragmentation, which the paper showed lifted MetFrag2.2 to 420 Top 1 ranks (89%) with ChemSpider.

## Related tools

- **MetFrag2.2** (Consumes the normalized ChemSpider combined reference score as a weighted scoring term during candidate ranking) — https://github.com/ipb-halle/MetFragRelaunched
- **ChemSpider** (Web-service source of the five per-candidate reference, data source and patent counts)
- **metfRag** (R interface to MetFrag that can be used to drive candidate retrieval and scoring workflows) — http://ipb-halle.github.io/MetFrag/projects/metfragr/

## Examples

```
mvn clean install -pl MetFragCommandLine -am && metfragcl -DPeakList=... -DDatabase=ChemSpider -DChemSpiderToken=$TOKEN
```

## Evaluation signals

- S_C_CCR values lie in [0, 1] after normalization, with at least one candidate at the maximum (= 1)
- The tab-separated table contains exactly five contributing reference counts per candidate alongside S_C_CCR
- Candidates with higher ChemSpider reference/data-source counts generally receive higher normalized scores; the correct candidate should rank higher when the reference term is included versus fragment score alone
- Weight combination satisfies ωFrag+ωRT+ωRefs=1 (validated in the paper by randomly drawing 1000 weight combinations)
- Web-service calls succeed for all candidates (no missing ChemSpider token or failed queries in the results table)

## Limitations

- Normalization is relative to the candidate list, so S_C_CCR values are not comparable across different runs or databases
- ChemSpider requires an access token (ChemSpiderToken in MetFragWeb settings.properties); missing token blocks retrieval
- Reference information improves ranking only when database coverage exists; spectra dominated by few fragments still lack discriminatory power and smaller, specific databases often improve identification more

## Evidence

- [abstract] Retrieval of reference, data source and patent information via ChemSpider and PubChem web services: "retrieval of reference, data source and patent information via ChemSpider and PubChem web services"
- [results] Weight sum constraint: [evidence span withheld — anchor status: not_found]
- [abstract] Improved top ranks with references: "Including reference and retention information in MetFrag2.2 improved this to 420 and 336 Top 1 ranks with ChemSpider and PubChem (89 and 71 %)"
- [readme] ChemSpider token requirement: "## define chemspider token to query ChemSpider database
ChemSpiderToken = ..."

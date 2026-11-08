---
name: combined-reference-score-computation
description: Use when MetFrag2.2 candidate lists already carry per-candidate PubChem
  patent/PubMed counts (PNP, PPC) or the five ChemSpider reference counts, and these
  must be converted into normalized scoring terms for ranking.
when_to_use_negative:
- When the candidate list was retrieved from a local repository (e.g. LocalPubChemDatabase
  in the MetFragWeb settings) without web-service reference/patent counts, so PNP,
  PPC or the five ChemSpider counts are unavailable
- When the goal is the fragment score itself (comparing in silico bond-dissociation
  fragments with measured product ions) — reference scoring only supplements, it does
  not replace fragmentation scoring
- When candidate filtering (element/substructure or InChIKey stereoisomer filtering)
  has not yet been done and the candidate list is not final, since normalization maxima
  depend on the exact list
tools:
- name: MetFrag2.2
  role: Computes and normalizes the combined reference scoring terms and integrates
    them into the weighted candidate ranking; available as command line, web and R
    (metfRag) versions
  repo: http://c-ruttkies.github.io/MetFrag/
- name: PubChem
  role: Source of the per-candidate patent counts (PNP) and PubMed counts (PPC) retrieved
    via PubChem web services
- name: ChemSpider
  role: Source of the five per-candidate reference counts retrieved via ChemSpider
    web services (requires a ChemSpiderToken in MetFrag settings)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:5210e2281ea84baa00d9708a25f7592e593217f98e111ba6c330bb36f0c67263
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpider
  - MetFrag2.2
  - PubChem
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# combined-reference-score-computation

## Summary

Computes the PubChem combined reference score (patent count + PubMed count, Eq. 5-6) and the ChemSpider combined reference score (five reference counts) per candidate, each normalized across the candidate list so reference evidence contributes comparably to MetFrag2.2 candidate ranking.

## When to use

Use when MetFrag2.2 candidate lists already carry per-candidate PubChem patent/PubMed counts (PNP, PPC) or the five ChemSpider reference counts, and these must be converted into normalized scoring terms for ranking.

## When NOT to use

- When the candidate list was retrieved from a local repository (e.g. LocalPubChemDatabase in the MetFragWeb settings) without web-service reference/patent counts, so PNP, PPC or the five ChemSpider counts are unavailable
- When the goal is the fragment score itself (comparing in silico bond-dissociation fragments with measured product ions) — reference scoring only supplements, it does not replace fragmentation scoring
- When candidate filtering (element/substructure or InChIKey stereoisomer filtering) has not yet been done and the candidate list is not final, since normalization maxima depend on the exact list

## Inputs

- Candidate list L from PubChem and ChemSpider retrievals (e.g. June 2015 snapshot) with per-candidate identifiers
- Per-candidate PubChem patent counts (PNP_C) and PubMed counts (PPC_C) from PubChem web services
- Five per-candidate ChemSpider reference counts from the ChemSpider web services
- Binary inclusion flags a1, a2 (each in {0,1}) selecting whether patent and/or PubMed counts enter the PubChem reference term

## Outputs

- Per-candidate normalized PubChem combined reference score S_C_PCR
- Per-candidate normalized ChemSpider combined reference score S_C_CCR
- Tab-separated results table with S_C_PCR, S_C_CCR and the contributing raw counts per candidate

## How to apply

For each candidate C in the list L, first build the PubChem cumulative reference term N_C_PCR = a1*PNP_C + a2*PPC_C, where PNP_C is the patent count and PPC_C the PubMed count retrieved from PubChem web services and a1, a2 are binary inclusion flags in {0,1} controlling which count types contribute. Normalize it over the whole candidate list as S_C_PCR = N_C_PCR / max_{C' in L} N_C'_PCR so the best-referenced candidate reaches 1.0 and all other candidates are scaled relative to it. Do the analogous computation for ChemSpider: combine the five per-candidate reference counts retrieved via the ChemSpider web services into S_C_CCR and normalize across the same candidate list. Write per-candidate S_C_PCR and S_C_CCR values together with the contributing counts to a tab-separated results table. In the MetFrag2.2 scoring scheme these reference terms are one of the weighted components combined with the fragment score and RT–logP score, with weights drawn such that ωFrag + ωRT + ωRefs = 1 (assessed over 1000 random weight combinations).

## Related tools

- **MetFrag2.2** (Computes and normalizes the combined reference scoring terms and integrates them into the weighted candidate ranking; available as command line, web and R (metfRag) versions) — http://c-ruttkies.github.io/MetFrag/
- **PubChem** (Source of the per-candidate patent counts (PNP) and PubMed counts (PPC) retrieved via PubChem web services)
- **ChemSpider** (Source of the five per-candidate reference counts retrieved via ChemSpider web services (requires a ChemSpiderToken in MetFrag settings))

## Examples

```
install_github("ipb-halle/MetFragR/metfRag")  # metfRag R interface to run MetFrag scoring incl. reference terms
```

## Evaluation signals

- Every normalized S_C_PCR and S_C_CCR falls in [0, 1], and for each term at least one candidate in L attains exactly 1.0 (the candidate holding the maximum raw count)
- Recomputing N_C_PCR with a1, a2 in {0,1} reproduces the stored values: dropping a flag to 0 zeroes the corresponding count contribution
- The tab-separated output table contains, for every candidate, both the raw counts (PNP, PPC and the five ChemSpider counts) and the two normalized scores
- In the final MetFrag2.2 weighted score the reference weight satisfies ωFrag + ωRT + ωRefs = 1, consistent with the 1000-drawn weight-combination optimization
- Sanity check: candidates absent from PubChem/ChemSpider (no references) receive S_C_PCR = 0 and S_C_CCR = 0, contributing nothing to ranking

## Limitations

- Counts depend on web-service retrieval at query time; the paper's benchmark used candidate structures retrieved from PubChem and ChemSpider in June 2015, so scores are not reproducible against later database states
- Mass spectra dominated by few fragments, and databases of growing size (PubChem had doubled since 2009), shift the relative impact of reference scores on ranking
- The card extraction covers only the PubChem term composition (a1, a2 binary weights) and list-normalization explicitly; the exact aggregation of the five ChemSpider counts is not detailed in the available context

## Evidence

- [abstract] retrieval of reference, data source and patent information via ChemSpider and PubChem web services: "retrieval of reference, data source and patent information via ChemSpider and PubChem web services"
- [abstract] Including reference and retention information improved ranking: "Including reference and retention information in MetFrag2.2 improved this to 420 and 336 Top 1 ranks with ChemSpider and PubChem (89 and 71 %), respectively"
- [results] Candidate retrieval from PubChem and ChemSpider snapshot: "candidate structures were retrieved from the compound databases PubChem and ChemSpider in June, 2015"
- [results] Weight combination constraint for scoring terms: [evidence span withheld — anchor status: not_found]
- [readme] ChemSpider token configuration in MetFrag web settings: "## define chemspider token to query ChemSpider database"

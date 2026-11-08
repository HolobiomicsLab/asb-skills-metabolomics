---
name: pubchem-patent-pubmed-count-retrieval
description: Use when performing MetFrag2.2 candidate ranking from HR-MS/MS spectra
  and reference/patent/literature counts must be fetched per candidate structure from
  PubChem and ChemSpider web services to enrich the score beyond in silico fragmentation
  alone.
when_to_use_negative:
- Candidates are drawn from smaller, specific databases (e.g. KEGG, HMDB, STOFF-IDENT,
  NORMAN suspects) where web-service reference lookups are not defined — the count
  terms are built from PubChem/ChemSpider data only
- Only spectral HR-MS/MS fragment matching is required and no reference/patent enrichment
  of the ranking is desired (the 'ωRefs' term is set to 0)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: MetFrag2.2
  role: Computes the combined and normalized reference scoring terms from the retrieved
    counts and incorporates them into candidate ranking
  repo: http://c-ruttkies.github.io/MetFrag/
- name: PubChem
  role: Source of per-candidate patent counts (PNP) and PubMed reference counts (PPC)
    via web services
- name: ChemSpider
  role: Source of the five per-candidate reference counts combined into S_C_CCR, accessed
    via ChemSpider web services with a token
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:fa6fb90fafd8b6416da6c5bd242a2e8d20ad3c2512480e4516536305402f74a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpider
  - MetFrag2.2
  - PubChem
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pubchem-patent-pubmed-count-retrieval

## Summary

Retrieves per-candidate patent counts (PNP) and PubMed reference counts (PPC) from PubChem web services — and the analogous five reference counts from ChemSpider — to build MetFrag2.2's combined reference scoring terms (Eqs. 5–8), which are normalized across the candidate list and used to boost ranking of literature-supported compounds.

## When to use

Use when performing MetFrag2.2 candidate ranking from HR-MS/MS spectra and reference/patent/literature counts must be fetched per candidate structure from PubChem and ChemSpider web services to enrich the score beyond in silico fragmentation alone.

## When NOT to use

- Candidates are drawn from smaller, specific databases (e.g. KEGG, HMDB, STOFF-IDENT, NORMAN suspects) where web-service reference lookups are not defined — the count terms are built from PubChem/ChemSpider data only
- Only spectral HR-MS/MS fragment matching is required and no reference/patent enrichment of the ranking is desired (the 'ωRefs' term is set to 0)

## Inputs

- MetFrag2.2 candidate list (structures retrieved from PubChem, ChemSpider, KEGG, or user-supplied SDF), with InChIKeys/CIDs
- PubChem web service responses giving per-candidate patent counts (PNP) and PubMed reference counts (PPC)
- ChemSpider web service responses giving five reference counts per candidate (requires ChemSpiderToken)

## Outputs

- Per-candidate combined reference scores S_C_PCR (PubChem, normalized by the list maximum) and S_C_CCR (ChemSpider, normalized by the list maximum)
- Tab-separated results table with S_C_PCR, S_C_CCR and the contributing patent/PubMed/reference counts per candidate

## How to apply

After MetFrag2.2 retrieves candidate structures (from PubChem, ChemSpider, KEGG or a user-supplied SDF), query the PubChem web services for each candidate's patent count (PNP_C) and PubMed count (PPC_C), then form the cumulative reference term N_C_PCR = a1·PNP_C + a2·PPC_C with weights a1, a2 ∈ {0,1}. Normalize to S_C_PCR = N_C_PCR / max over all candidates C' in the list so the term is comparable within the candidate set. Analogously, retrieve the five ChemSpider reference counts via ChemSpider web services (using the ChemSpider token), combine them into S_C_CCR, and normalize over the same candidate list. Write per-candidate S_C_PCR, S_C_CCR and the contributing raw counts to a tab-separated results table; include these terms in the weight optimization where ωFrag + ωRT + ωRefs = 1 (1000 randomly drawn weight combinations). Including reference and retention information improved MetFrag2.2 to 420 and 336 Top 1 ranks with ChemSpider and PubChem (89 and 71%), respectively.

## Related tools

- **MetFrag2.2** (Computes the combined and normalized reference scoring terms from the retrieved counts and incorporates them into candidate ranking) — http://c-ruttkies.github.io/MetFrag/
- **PubChem** (Source of per-candidate patent counts (PNP) and PubMed reference counts (PPC) via web services)
- **ChemSpider** (Source of the five per-candidate reference counts combined into S_C_CCR, accessed via ChemSpider web services with a token)

## Examples

```
install.packages("metfRag",repos=NULL,type="source"); library(metfRag)
```

## Evaluation signals

- S_C_PCR and S_C_CCR values lie in (0, 1] with at least one candidate per term at exactly 1.0 (the list maximum)
- Recomputing N_C_PCR = a1·PNP_C + a2·PPC_C from the raw counts in the results table reproduces the reported normalized score
- The tab-separated table contains both the normalized scores and the raw contributing patent/PubMed/reference counts for every candidate
- The candidate list is identical before and after reference-score annotation (annotation adds scores, does not remove candidates)

## Limitations

- Reference counts depend on live PubChem/ChemSpider web services and the database snapshot; the paper's candidates were retrieved in June 2015 and rankings shift as databases grow ('PubChem is now double the size of the 2009 version')
- ChemSpider access requires a valid ChemSpiderToken configured in the MetFrag settings file
- Ranking gains depend on optimized weights; reference/retention terms require separate optimization (ωFrag + ωRT + ωRefs = 1) per dataset

## Evidence

- [abstract] retrieval of reference, data source and patent information via ChemSpider and PubChem web services: "retrieval of reference, data source and patent information via ChemSpider and PubChem web services"
- [results] candidate structures were retrieved from the compound databases PubChem and ChemSpider in June, 2015: "candidate structures were retrieved from the compound databases PubChem and ChemSpider in June, 2015"
- [results] the combination of each scoring term was assessed by randomly drawing 1000 different weight combinations such that (ωFrag+ωRT+ωRefs=1): [evidence span withheld — anchor status: not_found]
- [readme] define chemspider token to query ChemSpider database: "## define chemspider token to query ChemSpider database
ChemSpiderToken = ..."
- [readme] install_github("ipb-halle/MetFragR/metfRag"): "install_github("ipb-halle/MetFragR/metfRag")"

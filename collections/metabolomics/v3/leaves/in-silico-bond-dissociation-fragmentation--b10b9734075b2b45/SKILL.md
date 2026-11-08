---
name: in-silico-bond-dissociation-fragmentation
description: Use when ranking candidate molecule structures for a tandem HR-MS/MS
  spectrum (e.g. Orbitrap product-ion peaks) by how well each candidate explains the
  observed fragments, without spectral-library search.
when_to_use_negative:
- The task is library-based identification — the spectrum can be matched directly
  against MassBank reference spectra instead of scored against candidate structures
- The mass spectrum is dominated by one or only a few fragments (e.g. a water loss)
  that most candidates can explain — such spectra 'simply do not contain enough information'
  to discriminate candidates by fragment scoring
- Only MSn (multi-stage) fragmentation data is available — MAGMa handles MSn information,
  MetFrag's bond-dissociation fragmenter works on single-stage MS/MS product ions
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3520
tools:
- name: MetFrag
  role: Original 2010 in silico bond-dissociation fragmenter used to explain product
    ions and rank database candidates
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFrag2.2
  role: Relaunched MetFrag providing the bond-dissociation fragmenter with the refined
    fragment score (alpha=1.84, beta=0.59, gamma=0.47) plus InChIKey stereoisomer
    filtering
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: CFM-ID
  role: Competitive fragment modelling alternative fragmenter used to rank the same
    candidate sets by fragmentation information only
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:02dbf402c03a7edd9841dc7959e1bbf3037fb386f5c816b1c1421601a6de1600
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - Maven
  - MetFrag
  - MetFrag2.2
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# in-silico-bond-dissociation-fragmentation

## Summary

MetFrag's core in silico fragmentation skill: each candidate structure from PubChem/ChemSpider (or a user-supplied SDF) is systematically cleaved via a bond-dissociation approach, and the generated neutral fragments are matched against measured HR-MS/MS product ions to compute a fragment-explanation score used to rank candidates.

## When to use

Use when ranking candidate molecule structures for a tandem HR-MS/MS spectrum (e.g. Orbitrap product-ion peaks) by how well each candidate explains the observed fragments, without spectral-library search.

## When NOT to use

- The task is library-based identification — the spectrum can be matched directly against MassBank reference spectra instead of scored against candidate structures
- The mass spectrum is dominated by one or only a few fragments (e.g. a water loss) that most candidates can explain — such spectra 'simply do not contain enough information' to discriminate candidates by fragment scoring
- Only MSn (multi-stage) fragmentation data is available — MAGMa handles MSn information, MetFrag's bond-dissociation fragmenter works on single-stage MS/MS product ions

## Inputs

- Measured HR-MS/MS peak list / spectrum (e.g. Orbitrap, from MassBank records or the EA, UF, EQEx, EQExPlus, HILL dataset parameter files)
- Candidate structure list (SDF or database search results from PubChem, ChemSpider, KEGG)
- Neutral precursor mass and formula constraints
- Optional suspect list supplied as InChIKeys (e.g. STOFF-IDENT converted via OpenBabel)

## Outputs

- Fragment score per candidate (optionally normalized so the top candidate = 1), with per-peak fragment-match assignments
- Score-ranked candidate list per spectrum (candidate, score, rank)
- Top 1 / Top 3 / Top 5 correct-candidate counts or median rank summaries, e.g. written to ranking_summary.csv or metfrag2010_hill_ranking_results.csv

## How to apply

Retrieve candidate structures from PubChem, ChemSpider or KEGG (or accept an SDF), then apply the MetFrag bond-dissociation fragmenter to every candidate, cleaving bonds and recording bond dissociation energies (BDEs) per fragment. Match the in silico fragments against the measured product-ion peaks and compute the fragment score as the sum over matched peaks of (relative peak mass)^alpha x (relative intensity)^beta divided by (summed BDE of cleaved bonds)^gamma; when several fragments explain one peak, keep the one with the lowest denominator value. For MetFrag2.2 use the optimized exponents alpha=1.84, beta=0.59, gamma=0.47; the original MetFrag2010 fragmenter (as reproduced on the Hill 102-compound dataset) can be run via MetFragCommandLine for fragmentation-only scoring with no reference or retention terms. After scoring, optionally apply stereoisomer filtering by keeping only the best-scored candidate per first InChIKey block, and optionally normalize candidate scores so the top-scoring candidate receives 1. When combining with other scoring terms, draw 1000 random weight combinations constrained to sum to 1 (omega_Frag + omega_RT + omega_Refs = 1).

## Related tools

- **MetFrag** (Original 2010 in silico bond-dissociation fragmenter used to explain product ions and rank database candidates) — http://c-ruttkies.github.io/MetFrag/
- **MetFrag2.2** (Relaunched MetFrag providing the bond-dissociation fragmenter with the refined fragment score (alpha=1.84, beta=0.59, gamma=0.47) plus InChIKey stereoisomer filtering) — https://github.com/ipb-halle/MetFragRelaunched
- **CFM-ID** (Competitive fragment modelling alternative fragmenter used to rank the same candidate sets by fragmentation information only)

## Examples

```
mvn clean install -pl MetFragCommandLine -am -DskipTests
```

## Evaluation signals

- On the 473 merged Orbitrap spectra with PubChem candidates and fragmentation-only scoring, Top 1 counts must equal 30 for MetFrag2.2 and 43 for CFM-ID
- On the Hill 102-compound dataset, the original MetFrag2010 fragmenter must reproduce median rank 8 (338 average candidates) against the Feb. 2006 PubChem snapshot and median rank 31.5 (2508 average candidates) against 2009 PubChem
- On the Hill dataset, MetFrag2.2 should improve the median rank to 14.5 (from 18.5 for MetFrag2010) with top 1/3/5 counts of 9/24/34
- If scores are normalized, all values must lie in [0, 1] with the top-scoring candidate exactly 1, and the ranking must match the result and ranking files deposited in the CHIN-D-15-00088 ZIP archive
- After InChIKey filtering, only one candidate per first InChIKey block remains, keeping the best-scored stereoisomer

## Limitations

- Spectra dominated by a single dominant fragment (e.g. a water loss) do not contain enough information to distinguish candidates, so fragment scoring alone will fail there
- Fragmentation-only scoring underperforms heavily on large modern databases: MetFrag2.2 got only 30 Top 1 ranks on PubChem vs 420/336 Top 1 ranks when reference and retention-time terms were added (ChemSpider/PubChem)
- Result reproducibility depends on the database snapshot: candidate ranking degrades as PubChem grows (2006 snapshot gave median rank 8 vs 31.5 in 2009), so results must be tied to the database version and retrieval date (June 2015 for the benchmark)
- Scores depend on BDE estimates of cleaved bonds; the parameter, spectral data, candidate, result and ranking files of the EA, UF, EQEx, EQExPlus and HILL datasets are only available as external ZIP archives

## Evidence

- [intro] Candidates are then fragmented using a bond dissociation approach: "Candidates are then fragmented using a bond dissociation approach"
- [intro] these fragments are compared with the product ions in the measured mass spectrum to determine which candidates best explain the measured data: "these fragments are compared with the product ions in the measured mass spectrum to determine which candidates best explain the measured data"
- [results] the combination of each scoring term was assessed by randomly drawing 1000 different weight combinations such that (ωFrag+ωRT+ωRefs=1): [evidence span withheld — anchor status: not_found]
- [results] Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [readme] mvn clean install -pl MetFragCommandLine -am: "mvn clean install -pl MetFragCommandLine -am"

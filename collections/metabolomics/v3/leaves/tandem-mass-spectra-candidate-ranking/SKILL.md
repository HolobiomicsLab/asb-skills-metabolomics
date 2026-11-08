---
name: tandem-mass-spectra-candidate-ranking
description: Use when you have tandem mass spectra and database-retrieved candidate
  structures and need a fragmentation-only candidate ranking baseline, e.g. to benchmark
  in silico fragmenters before adding reference, retention-time, or user-defined scores.
when_to_use_negative:
- Reference (literature/patent counts) or retention-time information is available
  and intended for scoring — this skill isolates the fragmentation-only baseline,
  so including those terms here would confound the comparison
- Candidates come from a spectral library (e.g. MassBank) search rather than in silico
  fragmentation of database candidates — library matching is a different identification
  strategy
- Spectra contain no fragment peaks other than the precursor (e.g. spectra dominated
  by a single water loss) — such spectra 'simply do not contain enough information'
  to distinguish candidates
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3520
tools:
- name: MetFrag2.2
  role: In silico bond dissociation fragmenter scoring PubChem candidates by fragment
    match only to produce candidate rankings
  repo: http://c-ruttkies.github.io/MetFrag/
- name: CFM-ID
  role: Competitive fragment modelling used to rank the same PubChem candidate sets
    with fragmentation information only, as the comparison baseline
- name: MetFragCommandLine
  role: Command-line MetFrag2.2 distribution used to run the fragmenter on the HILL
    spectra and candidate lists
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: xcms
  role: mzClust_hclust used to merge spectra into the 473-spectrum dataset (eppm =
    5e-6, eabs = 0.001 Da)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:dbcfd1108d6d2eacbc87a7fdbc691e67f9e62d2d2068e9e0ea5e954a8b3ae507
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MetFrag2.2
  - MetFragCommandLine
  - xcms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tandem-mass-spectra-candidate-ranking

## Summary

Ranks candidate structures retrieved from PubChem against merged HR-MS/MS spectra (473 merged Orbitrap spectra from the HILL dataset) using fragmentation information only, comparing MetFrag2.2's in silico bond dissociation against CFM-ID's competitive fragment modelling. It reproduces the fragmentation-only baseline (30 vs 43 Top 1 ranks) that motivates MetFrag2.2's additional scoring terms.

## When to use

Use when you have tandem mass spectra and database-retrieved candidate structures and need a fragmentation-only candidate ranking baseline, e.g. to benchmark in silico fragmenters before adding reference, retention-time, or user-defined scores.

## When NOT to use

- Reference (literature/patent counts) or retention-time information is available and intended for scoring — this skill isolates the fragmentation-only baseline, so including those terms here would confound the comparison
- Candidates come from a spectral library (e.g. MassBank) search rather than in silico fragmentation of database candidates — library matching is a different identification strategy
- Spectra contain no fragment peaks other than the precursor (e.g. spectra dominated by a single water loss) — such spectra 'simply do not contain enough information' to distinguish candidates

## Inputs

- 473 merged HR-MS/MS Orbitrap spectra from the HILL dataset (spectra retained only if they contain at least one fragment peak other than the precursor)
- Candidate structure lists retrieved from PubChem (June 2015 snapshot) via MetFrag's database search
- Correct compound identities per spectrum for computing Top 1/Top 5 counts

## Outputs

- Per-spectrum ranked candidate lists from MetFrag2.2 and CFM-ID
- Top 1 and Top 5 rank counts per tool (baseline: 30 vs 43 Top 1; 145 vs 170 Top 5)
- ranking_summary.csv containing per-spectrum rankings and the Top 1 counts

## How to apply

Load the 473 merged HR-MS/MS spectra (HILL dataset, merged with mzClust_hclust in xcms using eppm = 5e-6 and eabs = 0.001 Da) and retrieve candidate structures from PubChem via MetFrag's database search. For each spectrum, generate candidate fragments with MetFrag2.2's bond dissociation approach and score candidates solely by the fragment match score, explicitly excluding reference and retention-time terms; then repeat with CFM-ID's competitive fragment modelling on the same candidate sets. Apply InChIKey filtering to keep only the best-scored candidate per first InChIKey block, since stereoisomers otherwise inflate candidate lists without adding chemical information. Rank candidates per spectrum and count spectra where the correct compound is ranked first (Top 1) and within the top five (Top 5). Rationale: the fragment-match-only comparison establishes how much discriminative power pure in silico fragmentation provides before MetFrag2.2's additional scoring terms (references, log P-based retention, user-defined scores) are layered on.

## Related tools

- **MetFrag2.2** (In silico bond dissociation fragmenter scoring PubChem candidates by fragment match only to produce candidate rankings) — http://c-ruttkies.github.io/MetFrag/
- **CFM-ID** (Competitive fragment modelling used to rank the same PubChem candidate sets with fragmentation information only, as the comparison baseline)
- **MetFragCommandLine** (Command-line MetFrag2.2 distribution used to run the fragmenter on the HILL spectra and candidate lists) — https://github.com/ipb-halle/MetFragRelaunched
- **xcms** (mzClust_hclust used to merge spectra into the 473-spectrum dataset (eppm = 5e-6, eabs = 0.001 Da))

## Examples

```
git clone https://github.com/ipb-halle/MetFragRelaunched.git && mvn clean install -pl MetFragCommandLine -am
```

## Evaluation signals

- MetFrag2.2 fragmentation-only Top 1 count must equal 30 and CFM-ID must equal 43 on the 473-spectrum PubChem benchmark, matching the paper's abstract claim
- Top 5 counts should reproduce 145 (MetFrag2.2) and 170 (CFM-ID)
- After InChIKey filtering, no two retained candidates per spectrum share the same first InChIKey block (stereoisomer redundancy removed, best-scoring candidate kept)
- Every ranked spectrum contained at least one fragment peak other than the precursor
- Per-spectrum rankings and Top 1 counts are present in ranking_summary.csv for both tools

## Limitations

- Mass spectra dominated by one or only a few fragments (e.g. a water loss) that most candidates can explain do not contain enough information for discrimination, capping fragmentation-only Top 1 performance
- Further validation of the MetFrag2.2+CFM-ID combination was beyond the article's scope, as CFM-ID could be retrained on Orbitrap data
- Parameter files, spectral data, candidate, result and ranking files of the HILL dataset are only available as external ZIP archives (http://msbi.ipb-halle.de/download/CHIN-D-15-00088/), and candidate structures were retrieved from a June 2015 PubChem snapshot that no longer reflects the current database

## Evidence

- [abstract] fragmentation-only baseline: "Using HR-MS/MS information only, MetFrag2.2 and CFM-ID had 30 and 43 Top 1 ranks, respectively, using PubChem as a database."
- [results] stereoisomer handling: [evidence span withheld — anchor status: not_found]
- [intro] information content limitation: "mass spectra that are dominated by one or only a few fragments (e.g. a water loss) that can be explained by most of the candidates simply do not contain enough information"
- [intro] scoring mechanism: "these fragments are compared with the product ions in the measured mass spectrum to determine which candidates best explain the measured data"
- [readme] tool build path: "MetFrag commandline version depends on MetFragLib"

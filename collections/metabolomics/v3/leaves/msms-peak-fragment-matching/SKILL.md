---
name: msms-peak-fragment-matching
description: Use when you have a measured HR-MS/MS peak list and a set of candidate
  structures, and need to rank candidates by how well their in silico fragments explain
  the product ions.
when_to_use_negative:
- Spectra dominated by one or only a few fragments (e.g. a water loss) that can be
  explained by most candidates — the article states these 'simply do not contain enough
  information' to distinguish candidates
- You need final identification confidence rather than fragment-based ranking — this
  score alone is insufficient and should be combined with reference, retention time
  (RT–log P) and user-defined scores
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetFrag2.2
  role: Performs the in silico bond dissociation fragmentation of candidates and computes/normalizes
    the fragment score by matching fragments to measured product-ion peaks
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFrag
  role: Original in silico fragmenter whose bond dissociation approach underlies MetFrag2.2;
    available as command line, web and R (metfRag) versions
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:9b03c9ede5fbd5d59e9828fc998ce3ee2564aebc0ab5aff4efd39fb8d8caaf5f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag
  - MetFrag2.2
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# msms-peak-fragment-matching

## Summary

Score candidate structures for an HR-MS/MS spectrum by matching in silico-generated fragments (from MetFrag's bond dissociation approach) against measured product-ion peaks, computing the MetFrag2.2 fragment score S_C_Frag with exponents alpha=1.84, beta=0.59, gamma=0.47 and normalizing so the best candidate scores 1.

## When to use

Use when you have a measured HR-MS/MS peak list and a set of candidate structures, and need to rank candidates by how well their in silico fragments explain the product ions.

## When NOT to use

- Spectra dominated by one or only a few fragments (e.g. a water loss) that can be explained by most candidates — the article states these 'simply do not contain enough information' to distinguish candidates
- You need final identification confidence rather than fragment-based ranking — this score alone is insufficient and should be combined with reference, retention time (RT–log P) and user-defined scores

## Inputs

- Measured HR-MS/MS peak list (product ions with m/z and intensity, from the EA, UF, EQEx, EQExPlus or HILL dataset archives)
- Candidate structure list (e.g. retrieved from PubChem, ChemSpider or KEGG, or user-supplied SDF)
- In silico fragment list per candidate with summed bond dissociation energies of cleaved bonds
- Scoring exponents alpha=1.84, beta=0.59, gamma=0.47

## Outputs

- fragment_scores.csv with normalized fragment scores in [0, 1] (top candidate exactly 1) and per-peak match assignments
- Candidate ranking based on the fragment score

## How to apply

Fragment each candidate structure in silico with MetFrag2.2's bond dissociation approach, recording the bond dissociation energies (BDEs) of every cleaved bond per generated fragment. Match generated fragments against the product-ion peaks of the measured spectrum to determine which measured peaks each candidate explains. For each candidate, compute the fragment score as the sum over matched peaks p of (relative mass)^alpha x (relative intensity)^beta divided by (sum of BDEs of cleaved bonds)^gamma; when multiple fragments match a peak, use the fragment with the lowest denominator value. Use the optimized exponents alpha=1.84, beta=0.59, gamma=0.47, then normalize all candidate scores so the highest-scoring candidate receives 1 and write normalized scores plus per-peak match assignments to fragment_scores.csv.

## Related tools

- **MetFrag2.2** (Performs the in silico bond dissociation fragmentation of candidates and computes/normalizes the fragment score by matching fragments to measured product-ion peaks) — http://c-ruttkies.github.io/MetFrag/
- **MetFrag** (Original in silico fragmenter whose bond dissociation approach underlies MetFrag2.2; available as command line, web and R (metfRag) versions) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
git clone https://github.com/ipb-halle/MetFragRelaunched.git; mvn clean install -pl MetFragCommandLine -am
```

## Evaluation signals

- Normalized scores fall in the range [0, 1] with the top candidate exactly 1
- The produced ranking is consistent with the result and ranking files deposited in the dataset archive (EA, UF, EQEx, EQExPlus or HILL ZIP archives)
- Every matched peak contribution uses the fragment with the lowest summed-BDE denominator among fragments explaining that peak
- On the Hill et al. dataset the top 1/3/5 correct candidates should be 9, 24 and 34 respectively (median rank 14.5) when reproduced with the published parameters

## Limitations

- Mass spectra dominated by one or only a few fragments (e.g. a water loss) explainable by most candidates do not contain enough information for ranking
- Fragment scoring alone is weaker than combined scoring: with HR-MS/MS only on PubChem, MetFrag2.2 achieved 30 Top 1 ranks, versus 420 and 336 with reference and retention information (ChemSpider and PubChem)
- Candidate redundancy from stereoisomerism distorts rankings unless InChIKey filtering keeps only the best-scored candidate per first InChIKey block
- MetFrag2.2 was outperformed by CFM-ID (43 vs 30 Top 1 ranks with PubChem, HR-MS/MS only) in the published comparison

## Evidence

- [methods] For each candidate, compute the fragment score as the sum over matched peaks of (relative peak mass)^alpha × (relative intensity)^beta divided by (summed BDE of cleaved bonds)^gamma: [evidence span withheld — anchor status: not_found]
- [methods] Fragment each candidate in silico with the MetFrag2.2 bond dissociation approach, recording the bond dissociation energies (BDEs) of every cleaved bond per generated fragment.: [evidence span withheld — anchor status: not_found]
- [intro] Candidates are then fragmented using a bond dissociation approach: "Candidates are then fragmented using a bond dissociation approach"
- [intro] these fragments are compared with the product ions in the measured mass spectrum to determine which candidates best explain the measured data: "these fragments are compared with the product ions in the measured mass spectrum to determine which candidates best explain the measured data"
- [abstract] Using HR-MS/MS information only, MetFrag2.2 and CFM-ID had 30 and 43 Top 1 ranks, respectively, using PubChem as a database.: "Using HR-MS/MS information only, MetFrag2.2 and CFM-ID had 30 and 43 Top 1 ranks, respectively, using PubChem as a database."
- [results] the median rank improved from 18.5 to 14.5, while the number of correct ranked candidates in the top 1, 3 and 5 improved from 8 to 9, 20 to 24 and 28 to 34: "the median rank improved from 18.5 to 14.5, while the number of correct ranked candidates in the top 1, 3 and 5 improved from 8 to 9, 20 to 24 and 28 to 34"

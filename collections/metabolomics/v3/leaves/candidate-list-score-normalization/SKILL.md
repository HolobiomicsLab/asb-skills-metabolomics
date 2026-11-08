---
name: candidate-list-score-normalization
description: Use when you have MetFrag fragment scores for multiple candidate structures
  (from PubChem/ChemSpider/SDF retrieval) plus auxiliary terms (PubChem/ChemSpider
  reference counts, log P-based RT predictions, suspect/substructure penalties) that
  must be placed on a common scale before weighted.
when_to_use_negative:
- When the candidate list has been reduced to a single candidate — normalization across
  candidates is meaningless
- When comparing raw counts or BDE-derived denominators directly without normalization
  is the desired output (e.g. for reporting literature statistics rather than ranking
  candidates)
- When the goal is to rank candidates without auxiliary information and only one scoring
  term (e.g. fragment score alone) is used — no cross-term normalization is needed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MetFrag2.2
  role: Performs candidate scoring (fragment, reference, RT, user-defined terms) and
    applies the normalization and weighted combination that this skill describes
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFrag
  role: Command-line / webapp implementation used to compute and combine the normalized
    candidate scores
  repo: http://c-ruttkies.github.io/MetFrag/
- name: PubChem
  role: Source of patent counts (PNP) and PubMed reference counts (PPC) that form
    the normalized PubChem combined reference term S_C_PCR
- name: ChemSpider
  role: Source of five reference count types that form the normalized ChemSpider combined
    reference term S_C_CCR
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:ed868fdf04aacc074ec08643dacae5bd25d6818351aef748024ee481fbfe6885
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - ChemSpider
  - MetFrag
  - MetFrag2.2
  - PubChem
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# candidate-list-score-normalization

## Summary

Normalizes MetFrag2.2's per-candidate fragment (S_C_Frag), reference (S_C_Ref/PCR/CCR), retention (S_C_RT) and user-defined scores across the candidate list so weights summing to 1 combine them into one final ranking score.

## When to use

Use when you have MetFrag fragment scores for multiple candidate structures (from PubChem/ChemSpider/SDF retrieval) plus auxiliary terms (PubChem/ChemSpider reference counts, log P-based RT predictions, suspect/substructure penalties) that must be placed on a common scale before weighted combination for ranking.

## When NOT to use

- When the candidate list has been reduced to a single candidate — normalization across candidates is meaningless
- When comparing raw counts or BDE-derived denominators directly without normalization is the desired output (e.g. for reporting literature statistics rather than ranking candidates)
- When the goal is to rank candidates without auxiliary information and only one scoring term (e.g. fragment score alone) is used — no cross-term normalization is needed

## Inputs

- Per-candidate MetFrag2.2 fragment scores S_C_Frag with per-peak match assignments and BDE denominators
- Per-candidate PubChem patent counts (PNP) and PubMed counts (PPC) retrieved via PubChem web services
- Per-candidate ChemSpider reference counts (five count types) retrieved via ChemSpider web services
- Retention times extracted from MassBank records used to fit the RT-log P model per dataset
- User-defined score values per candidate (e.g. substructure/suspect-list flags or penalties)

## Outputs

- Normalized fragment score S_C_Frag per candidate (range [0,1], top candidate = 1)
- Normalized PubChem combined reference score S_C_PCR per candidate
- Normalized ChemSpider combined reference score S_C_CCR per candidate
- Normalized retention score S_C_RT per candidate
- Final combined score per candidate (weighted sum of normalized terms) used for ranking
- Tab-separated results table listing normalized terms and contributing counts

## How to apply

After computing the raw fragment score S_C_Frag for each candidate (sum over matched peaks of (relative mass)^alpha x (relative intensity)^beta divided by (summed BDE of cleaved bonds)^gamma, with optimized exponents alpha=1.84, beta=0.59, gamma=0.47), normalize all candidate fragment scores so the top-scoring candidate equals 1. Build the PubChem combined reference score S_C_PCR from the candidate's PubChem patent count (PNP_C) and PubMed count (PPC_C), and the ChemSpider combined reference score S_C_CCR from the five ChemSpider reference counts, then normalize each of these across the candidate list by dividing by the maximum value over all candidates (max-normalization), so the highest-scoring candidate gets 1. Predicted retention scores S_C_RT from the RT-log P model (Eq. 9, fitted per dataset with tenfold cross-validation) and any user-defined scores are normalized the same way. Combine the normalized terms with weights omega Frag, omega RT, omega Refs that sum to 1; the article assessed weight combinations by randomly drawing 1000 weight combinations such that (omegaFrag + omegaRT + omegaRefs = 1). Rationale: raw counts and absolute scores have incomparable ranges, so max-normalizing each term across the same candidate list makes the weighted sum meaningful and sensitive to weight optimization.

## Related tools

- **MetFrag2.2** (Performs candidate scoring (fragment, reference, RT, user-defined terms) and applies the normalization and weighted combination that this skill describes) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFrag** (Command-line / webapp implementation used to compute and combine the normalized candidate scores) — http://c-ruttkies.github.io/MetFrag/
- **PubChem** (Source of patent counts (PNP) and PubMed reference counts (PPC) that form the normalized PubChem combined reference term S_C_PCR)
- **ChemSpider** (Source of five reference count types that form the normalized ChemSpider combined reference term S_C_CCR)

## Examples

```
metfragcl --fragment-weights 1.84 0.59 0.47 --normalize-scores --ref-sources pubchem,chemspider --output candidate_scores.csv
```

## Evaluation signals

- All normalized scores fall in [0, 1] with the top-scoring candidate exactly equal to 1
- Normalized S_C_PCR and S_C_CCR values are monotonically consistent with the raw PubChem/ChemSpider counts used to derive them
- Ranking produced by the weighted combination of normalized terms is consistent with the result and ranking files deposited in the EA, UF, EQEx, EQExPlus or HILL dataset archives
- Weights drawn for combination satisfy the constraint omegaFrag + omegaRT + omegaRefs = 1 (verified over the 1000 drawn combinations)
- Stereoisomer-filtered candidate list shows consistent normalized scores when only the best-scored candidate per first-InChIKey block is retained

## Limitations

- Max-normalization depends on the candidate list: a single extreme candidate (very high patent/PubMed count or fragment score) can compress all other candidates toward 0
- Normalization is performed within a fixed candidate set; adding or removing candidates changes all normalized values and thus the ranking
- Mass spectra dominated by one or only a few fragments (e.g. a water loss) explainable by most candidates give little discriminating power, so normalized fragment scores alone may not rank the correct candidate highly
- Optimal weights were verified on 824 merged HR-MS/MS spectra in three additional datasets, but transfer to other acquisition platforms or databases was not demonstrated in this article

## Evidence

- [methods] MetFrag2.2 fragment score normalized so top candidate = 1: [evidence span withheld — anchor status: not_found]
- [results] PubChem combined reference term definition and max-normalization: [evidence span withheld — anchor status: not_found]
- [results] Weight-combination optimization scheme: [evidence span withheld — anchor status: not_found]
- [abstract] Retention-time term incorporated as user-defined score: "or incorporated separately as "user-defined scores" to be included in candidate ranking"
- [intro] Database size growth makes ranking harder, motivating multi-term normalization: "As PubChem is now double the size of the 2009 version, the candidate ranking becomes more challenging over time"

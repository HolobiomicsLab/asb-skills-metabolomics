---
name: physical-vs-reaction-pathway-case-handling
description: Use when computing per-pathway functional similarity (SimPi) for two
  KEGG organisms and a reference pathway is missing from one organism or has no reactions
  in either, requiring special-case values instead of a Jaccard ratio.
when_to_use_negative:
- The pathway is present in both organisms and contains reactions — then the plain
  Jaccard index |Ri∩Ri′|/|Ri∪Ri′| applies and no special case is triggered
- Computing the structural-level index SimSi — it has a different case rule set (1/(1+k)
  for isolated-vs-connected nodes) and must not reuse SimPi's 0/1 defaults
- Input networks were not reconstructed from KEGG KGML data — the reference-pathway
  decomposition underpinning the case handling is KEGG-specific
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0091
tools:
- name: MetNet
  role: Java tool that reconstructs the two KEGG organisms' networks and implements
    the functional-level comparison window with 'set' or 'multiset' modes where the
    SimPi special cases are applied
  repo: https://github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of KGML pathway files and the standardized reference-pathway decomposition
    that defines which pathways are checked for presence/absence and empty reaction
    content
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:baad202a5c1842778d2de64bbf68e438d3e7c1b5e207ba193836cfaec00d9ab6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/simeoni-biolab/MetNet
  tools:
  - Java
  - KEGG database
  - MetNet
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# physical-vs-reaction-pathway-case-handling

## Summary

Handles the degenerate pathway cases MetNet's functional similarity index SimPi must distinguish — pathways present in only one organism, and pathways present in both but containing no reactions ('physical' pathways) — assigning SimPi=0 and SimPi=1 respectively before Jaccard computation on reaction sets/multisets.

## When to use

Use when computing per-pathway functional similarity (SimPi) for two KEGG organisms and a reference pathway is missing from one organism or has no reactions in either, requiring special-case values instead of a Jaccard ratio.

## When NOT to use

- The pathway is present in both organisms and contains reactions — then the plain Jaccard index |Ri∩Ri′|/|Ri∪Ri′| applies and no special case is triggered
- Computing the structural-level index SimSi — it has a different case rule set (1/(1+k) for isolated-vs-connected nodes) and must not reuse SimPi's 0/1 defaults
- Input networks were not reconstructed from KEGG KGML data — the reference-pathway decomposition underpinning the case handling is KEGG-specific

## Inputs

- Parsed KGML-derived pathway data for both organisms (reaction sets/multisets per reference pathway, reconstructed from KEGG)
- pathwayList.txt configuration file listing the KEGG reference pathways to consider
- Comparison mode selection: "set" or "multiset"
- KEGG organism codes for the two organisms (e.g. hsa, ptr)

## Outputs

- Per-pathway table with one row per KEGG reference pathway present in either organism, recording SimPi (with special-case 0/1 values) and SimSi
- |Ri∪Ri′| reaction-count column supporting weighted aggregation

## How to apply

Iterate over the KEGG reference pathways listed in pathwayList.txt. For each reference pathway Pi, first check presence: if the pathway occurs in only one organism, set SimPi=0 because there is nothing to intersect; if the pathway occurs in both organisms but contains no reactions (a 'physical' pathway, e.g. one drawn only with compounds), set SimPi=1. Only for pathways present in both with reactions do you compute the Jaccard index |Ri∩Ri′|/|Ri∪Ri′| over the reaction sets or multisets, honoring the user's 'set' or 'multiset' comparison mode (multiset weighs repeated reactions individually). Record the |Ri∪Ri′| denominator to support the weighted aggregation into the global functional index, and keep structural handling separate: SimSi has its own special cases (isolated nodes in both graphs → 1; isolated in one graph vs degree k in the other → 1/(1+k)).

## Related tools

- **MetNet** (Java tool that reconstructs the two KEGG organisms' networks and implements the functional-level comparison window with 'set' or 'multiset' modes where the SimPi special cases are applied) — https://github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of KGML pathway files and the standardized reference-pathway decomposition that defines which pathways are checked for presence/absence and empty reaction content)

## Examples

```
java -jar MetNet.jar hsa ptr set
```

## Evaluation signals

- Every computed SimPi and SimSi value lies in the interval [0, 1] as required by the index definitions
- Pathways present in only one organism have SimPi exactly 0, not a partial Jaccard value
- Pathways present in both organisms with empty reaction sets have SimPi exactly 1 (physical pathway convention)
- The results table covers all KEGG reference pathways present in O or O′, i.e. those listed in pathwayList.txt
- For multiset mode, repeated reactions contribute individually to |Ri∩Ri′| and |Ri∪Ri′|, so values can differ from set mode

## Limitations

- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information
- The case rules apply per reference pathway only; global aggregation requires the separate weighted scheme using |Ri∪Ri′|
- SimPi special-case values are conventions of the method (0 for absence, 1 for empty physical pathways) and may not transfer to other similarity frameworks

## Evidence

- [other] set SimPi=0 if the pathway is present in only one organism, SimPi=1 if the pathway is present in both but contains no reactions (physical pathway): [evidence span withheld — anchor status: not_found]
- [methods] comparison method: "set" or "multiset": "3. comparison method: "set" or "multiset""
- [abstract] they allow for both a local comparison, pathway by pathway, and a global comparison of the entire metabolism: "They allow for both a local comparison, pathway by pathway, and a global comparison of the entire"
- [readme] Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory.: "Please ensure that the configuration files organismList.txt and pathwayList.txt are in the tool's directory."

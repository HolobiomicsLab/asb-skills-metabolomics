---
name: set-vs-multiset-pathway-representation-selection
description: Use when reconstructing two organisms' metabolic networks from KEGG KGML
  and the user must pick how repeated reactions within a reference pathway are counted
  — as identical members of a set or as separately weighted multiset elements — before
  computing SimPi.
when_to_use_negative:
- When the structural level (graph of pathways connected by shared non-ubiquitous
  compounds, SimSi) is the target — the set/multiset choice applies only to the functional
  reaction-content level, not to the structural graph comparison.
- 'When the input is a single organism''s network or a non-KEGG data source: the choice
  presupposes two KEGG organisms whose reference pathways have been reconstructed
  by MetNet, and the article notes the reconstruction is limited to the KEGG database.'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: MetNet
  role: Provides the 'set' vs 'multiset' comparison-method option that instantiates
    this skill; reconstructs both networks from KEGG and computes SimPi and the global
    indexes under the chosen mode.
  repo: github.com/simeoni-biolab/MetNet
- name: KEGG database
  role: Source of the KGML files and reference pathways whose reaction content is
    represented either as a set or as a multiset.
- name: guava-19.0.jar
  role: Java library linked when compiling MetNet, which executes the set/multiset
    comparison.
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1371/journal.pone.0246962
    title: 'MetNet: A two-level approach to reconstructing and comparing metabolic
      networks'
  source_content_hash: sha256:731a816ad878981e071a50380f49a0aad9be7d1730d557c5cd115d9ba7ba3e2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - KEGG database
  - MetNet
  - gs-algo-1.3.jar
  - gs-core-1.3.jar
  - gs-ui-1.3.jar
  - guava-19.0.jar
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# set-vs-multiset-pathway-representation-selection

## Summary

Choose between 'set' and 'multiset' reaction-content comparison in MetNet, deciding whether each KEGG reference pathway's reaction set is treated as an unordered collection (SimPi on |Ri∩Ri′|/|Ri∪Ri′| Jaccard) or as a bag where repeated reactions carry individual weight.

## When to use

Use when reconstructing two organisms' metabolic networks from KEGG KGML and the user must pick how repeated reactions within a reference pathway are counted — as identical members of a set or as separately weighted multiset elements — before computing SimPi.

## When NOT to use

- When the structural level (graph of pathways connected by shared non-ubiquitous compounds, SimSi) is the target — the set/multiset choice applies only to the functional reaction-content level, not to the structural graph comparison.
- When the input is a single organism's network or a non-KEGG data source: the choice presupposes two KEGG organisms whose reference pathways have been reconstructed by MetNet, and the article notes the reconstruction is limited to the KEGG database.

## Inputs

- KGML pathway files for two KEGG organisms retrieved via the public KEGG APIs (extracted reactions and compounds per reference pathway)
- configuration file pathwayList.txt listing the KEGG reference pathways to consider
- configuration file organismList.txt listing the KEGG organism codes
- User's chosen comparison method: 'set' or 'multiset'

## Outputs

- Per-reference-pathway table of SimPi values computed under the selected set or multiset representation
- Global similarity indexes (Psim, PsimW, Ssim, Csim) for the organism pair under the selected mode
- Second complete result set if the organism pair is re-run under the alternative mode (e.g. similarity_results.csv for set and multiset runs)

## How to apply

After MetNet downloads and parses the KGML files of both organisms via the public KEGG APIs, choose the reaction-content representation for the functional level. Under 'set', each reference pathway Pi is represented by the plain set of its reactions and SimPi reduces to the Jaccard index |Ri∩Ri′|/|Ri∪Ri′| over those sets (0 if Pi is present in only one organism, 1 if present in both but with no reactions). Under 'multiset', repeated reactions within Pi are retained as separate elements, so a reaction occurring twice contributes twice to the intersection and union cardinalities; SimPi still lies in [0,1]. The choice is supplied as MetNet's third command-line parameter, so the same pair can be re-run under both modes (as done for api vs bap) to check whether reaction multiplicities change the global indexes (Psim, PsimW, Ssim, Csim).

## Related tools

- **MetNet** (Provides the 'set' vs 'multiset' comparison-method option that instantiates this skill; reconstructs both networks from KEGG and computes SimPi and the global indexes under the chosen mode.) — github.com/simeoni-biolab/MetNet
- **KEGG database** (Source of the KGML files and reference pathways whose reaction content is represented either as a set or as a multiset.)
- **guava-19.0.jar** (Java library linked when compiling MetNet, which executes the set/multiset comparison.)

## Examples

```
java -jar MetNet.jar api bap multiset
```

## Evaluation signals

- Every computed SimPi value lies in [0, 1], as required by the index definition under both set and multiset representations.
- Pathways present in only one organism yield SimPi = 0, and pathways present in both with no reactions yield SimPi = 1, regardless of mode.
- Re-running the same organism pair under both modes produces two distinct global-index sets (Psim, PsimW, Ssim, Csim); for api vs bap all four indexes remained low (5.02%–14.73%) in both modes, confirming the metabolisms are largely different.
- The organism pair and comparison method are correctly passed as the three command-line parameters (KEGG code 1, KEGG code 2, 'set' or 'multiset') and reflected in the results.

## Limitations

- The choice is binary and applies only to the functional (reaction-content) level; the structural-level SimSi comparison of pathway graphs is unaffected by set vs multiset.
- Reconstruction is based uniquely on the KEGG database, which can be a limit with respect to the represented information; non-KEGG pathway sources do not feed this choice.
- Both modes must typically be run to see the effect of reaction multiplicities, doubling the KEGG retrieval and comparison work per organism pair.

## Evidence

- [methods] Comparison method is a user choice: "3. comparison method: "set" or "multiset""
- [readme] README documents the third CLI parameter: "3. comparison method: "set" or "multiset"

For instance, the command

java -jar MetNet.jar hsa ptr set"
- [abstract] Article explains the two-level representation and the functional level: "Our proposal relies on a two-level representation of the huge metabolic network: the first level is graph-based and depicts pathways as nodes and relations between pathways as edges; the second level"
- [results] Task card grounds the Jaccard index and mode semantics: [evidence span withheld — anchor status: not_found]

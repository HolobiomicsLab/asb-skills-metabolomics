---
name: lipid-isomer-ambiguity-handling
description: Use when lipid identifiers contain unresolved mass-isomer groups such
  as 'TAG 48:2 total (14:0/16:0/18:2)(14:0/16:1/18:1)(16:0/16:1/16:1)', and residue
  extraction must account for or discard isobaric uncertainty.
when_to_use_negative:
- The lipid identifier is a single unambiguous species with no isomer group — no ambiguity
  policy is needed and the standard residue extraction suffices.
- The input contains total lipid class counts or non-numerical columns — make_residues_table's
  cleanup/unwanted handling, not ambiguity handling, is the relevant step.
- The identifiers are RefMet-compliant common names or FA(N:N) forms — use the .refmet_residues()
  method instead of the isomer-group parser.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3359
tools:
- name: liputils
  role: Provides the Lipid class (.residues() with drop_ambiguous parameter) and make_residues_table()
    that implement isomer ambiguity detection, counting, rejection, or uncertainty
    division
  repo: https://github.com/Stemanz/liputils
- name: pandas
  role: Holds the lipidomics table (lipid names as index, samples in columns) that
    make_residues_table() transforms while applying the drop_ambiguous policy
- name: Python
  role: Runtime environment in which liputils' Lipid class and ambiguity-handling
    parameters are invoked
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-70259-9
    title: 'liputils: a Python module to manage individual fatty acid moieties from
      complex lipids'
  source_content_hash: sha256:0f1342c7857b1c53234dab312b9746ff7bffe44cad0735d3f1cc99ce61a70c68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Stemanz/liputils
  tools:
  - Python
  - liputils
  - pandas
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3359
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipid-isomer-ambiguity-handling

## Summary

A liputils-based strategy for managing unresolved isomer ambiguities when extracting fatty acid residues from lipid identifiers, either by counting all isomer-group members with an ambiguity count or by rejecting ambiguous lipids via the drop_ambiguous parameter.

## When to use

Use when lipid identifiers contain unresolved mass-isomer groups such as 'TAG 48:2 total (14:0/16:0/18:2)(14:0/16:1/18:1)(16:0/16:1/16:1)', and residue extraction must account for or discard isobaric uncertainty.

## When NOT to use

- The lipid identifier is a single unambiguous species with no isomer group — no ambiguity policy is needed and the standard residue extraction suffices.
- The input contains total lipid class counts or non-numerical columns — make_residues_table's cleanup/unwanted handling, not ambiguity handling, is the relevant step.
- The identifiers are RefMet-compliant common names or FA(N:N) forms — use the .refmet_residues() method instead of the isomer-group parser.

## Inputs

- Lipid identifiers containing unresolved isomer groups in the form 'CLASS(N:N/M:M)(other mass isomers)' (e.g. 'TAG 48:2 total (14:0/16:0/18:2)(14:0/16:1/18:1)(16:0/16:1/16:1)')
- Lipid identifiers in generic format 'CLASS N:N/M:M/../..' (e.g. 'PG 18:1/20:1')
- A pandas.DataFrame of lipidomics data with lipid names as vertical index and samples in columns (for make_residues_table)

## Outputs

- A (residues, ambiguity_count) tuple from Lipid.residues(), e.g. (['14:0', '16:0', '18:2', '14:0', '16:1', '18:1', '16:0', '16:1', '16:1'], 3)
- An empty result ([], 0) for lipids rejected with drop_ambiguous=True
- A residues table (pandas DataFrame) from make_residues_table() in which each residue is divided by its uncertainty when drop_ambiguous=False

## How to apply

Parse each identifier with liputils' Lipid class and call .residues(), which recognizes both 'CLASS N:N/M:M' and the isomer-group format 'CLASS(N:N/M:M)(other mass isomers)' and returns a (residues, ambiguity) tuple. For unambiguous lipids like 'PG 18:1/20:1' the result is (['18:1', '20:1'], 1); for an isomer group like 'TAG 48:2 total (14:0/16:0/18:2)(14:0/16:1/18:1)(16:0/16:1/16:1)' it is (['14:0', '16:0', '18:2', '14:0', '16:1', '18:1', '16:0', '16:1', '16:1'], 3). Decide the policy: when drop_ambiguous=True the ambiguous lipid is rejected and returns ([], 0); when False (the default in make_residues_table) each residue is divided by its uncertainty count. When converting whole tables, pass drop_ambiguous through make_residues_table() so the policy is applied uniformly across the lipid index. Record the ambiguity counts alongside residues (e.g. in a pandas DataFrame) so downstream submolar-fraction aggregation correctly down-weights uncertain species.

## Related tools

- **liputils** (Provides the Lipid class (.residues() with drop_ambiguous parameter) and make_residues_table() that implement isomer ambiguity detection, counting, rejection, or uncertainty division) — https://github.com/Stemanz/liputils
- **pandas** (Holds the lipidomics table (lipid names as index, samples in columns) that make_residues_table() transforms while applying the drop_ambiguous policy)
- **Python** (Runtime environment in which liputils' Lipid class and ambiguity-handling parameters are invoked)

## Examples

```
from liputils import Lipid; l = Lipid("TAG 48:2 total (14:0/16:0/18:2)(14:0/16:1/18:1)(16:0/16:1/16:1)"); l.residues(drop_ambiguous=True)
```

## Evaluation signals

- 'PG 18:1/20:1' with .residues() must return (['18:1', '20:1'], 1) — ambiguity count 1 for an unambiguous species.
- 'TAG 48:2 total (14:0/16:0/18:2)(14:0/16:1/18:1)(16:0/16:1/16:1)' must yield the 9-residue list with ambiguity count 3 (one per isomer group).
- Calling .residues(drop_ambiguous=True) on the same TAG isomer group must return ([], 0), confirming rejection of the ambiguous lipid.
- With drop_ambiguous=False in make_residues_table, each residue value in the output table must equal the original amount divided by its uncertainty count.
- make_residues_table defaults drop_ambiguous to False, so ambiguous lipids must appear (down-weighted) unless the parameter is explicitly set to True.

## Limitations

- drop_ambiguous=True discards all information for ambiguous lipids rather than partially attributing residues, reducing coverage when many species are unresolved isomer groups.
- With drop_ambiguous=False, each residue is divided by its uncertainty, which spreads the measured amount across isomers — appropriate for aggregate residue totals but not for isomer-specific quantitation.
- The default (drop_ambiguous=False) silently down-weights ambiguous species, which should be considered when interpreting submolar fractions.

## Evidence

- [readme] it is possible to reject non unambiguous lipids altogether by calling ``.residues()`` with the ``drop_ambiguous`` parameter: "it is possible to reject non unambiguous lipids altogether by calling ``.residues()`` with the ``drop_ambiguous`` parameter"
- [readme] l.residues() returns the isomer-group residues with ambiguity 3: "(['14:0', '16:0', '18:2', '14:0', '16:1', '18:1', '16:0', '16:1', '16:1'], 3)"
- [readme] drop_ambiguous parameter doc: defaults to False, each residue divided by its uncertainty: "drop_ambiguous: <bool> don't take isobars into consideration. Defaults to False. If True,
    each residue is divided by its uncertainty."
- [methods] Article methods on the drop_ambiguous option: "it is possible to reject non unambiguous lipids altogether by calling ``residues() with the drop_ambiguous parameter"
- [methods] Article methods on make_residues_table usage: "make_residues_table() will take care of dropping non-numerical columns, as well as to trim the lipid list of elements that should not be processed"

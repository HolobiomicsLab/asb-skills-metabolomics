---
name: candidate-filter-module-implementation
description: Use when a MetFrag candidate list from PubChem/ChemSpider retrieval or
  a user-supplied SDF must be pruned or re-scored by InChIKey skeleton block, element
  restrictions, or SMARTS substructure inclusion/exclusion before fragmentation and
  ranking.
when_to_use_negative:
- Candidates are already unique at the first-InChIKey-block level, so stereoisomer
  redundancy reduction would be a no-op
- The analysis goal is suspect-list presence filtering (e.g. STOFF-IDENT suspects
  supplied to MetFrag), which is a separate MetFrag2.2 feature
- The task is fragment-vs-product-ion scoring after bond dissociation fragmentation
  rather than pre-fragmentation candidate filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0092
tools:
- name: MetFrag2.2
  role: Host implementation whose MetFragLib/CommandLine provide the InChIKey filter,
    element restriction modes, and SMARTS substructure scoring behind a single configuration
    interface
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: Chemistry Development Kit
  role: Performs the SMARTS substructure matching used to compute N_included and N_excluded
    match counts per candidate
- name: metfRag
  role: R wrapper (MetFragR) exposing the MetFrag library for scripted candidate-filtering
    runs
  repo: https://github.com/ipb-halle/MetFragR/metfRag
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:8b5ef06740b2f79b31d65f11f1f20d275aeef60cbb1a3e3769f2929301f083d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - Chemistry Development Kit
  - MetFrag2.2
  - metfRag
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# candidate-filter-module-implementation

## Summary

Implements the MetFrag2.2 pre-fragmentation candidate-filtering modules: InChIKey first-block filtering to collapse stereoisomeric redundancy, element restriction in include/require/exclude modes, and SMARTS substructure inclusion/exclusion scoring (Eqs. 3-4, W_include = W_exclude = 0.5).

## When to use

Use when a MetFrag candidate list from PubChem/ChemSpider retrieval or a user-supplied SDF must be pruned or re-scored by InChIKey skeleton block, element restrictions, or SMARTS substructure inclusion/exclusion before fragmentation and ranking.

## When NOT to use

- Candidates are already unique at the first-InChIKey-block level, so stereoisomer redundancy reduction would be a no-op
- The analysis goal is suspect-list presence filtering (e.g. STOFF-IDENT suspects supplied to MetFrag), which is a separate MetFrag2.2 feature
- The task is fragment-vs-product-ion scoring after bond dissociation fragmentation rather than pre-fragmentation candidate filtering

## Inputs

- Candidate structure list retrieved from PubChem/ChemSpider or a user-supplied structure data file (SDF)
- InChIKey per candidate (or computable from SMILES/InChI)
- Element restriction lists in include-only, require, and exclude modes (e.g. included elements = C, S)
- SMARTS patterns for substructure inclusion (e.g. S(=O)(=O)O; CCc1ccc(cc1)S(=O)(=O)O) and exclusion (e.g. S(=O)(=O)OC)
- Candidate fragment scores for best-representative selection per InChIKey first-block group

## Outputs

- Deduplicated candidate list with one best-scored candidate per first InChIKey block
- Filtered candidate list after element restriction
- Substructure-scored candidate list with N_included/N_excluded match counts normalized per Eqs. 3-4
- Report file (report.md) documenting modules, example parameterizations, and a test run

## How to apply

Parse the candidate list and compute each candidate's InChIKey, grouping candidates that share the first InChIKey block (molecular skeleton, not stereochemistry), then keep only the single best-scored representative per group to remove stereoisomeric redundancy. Apply element restrictions in three modes — filter included elements, filter required elements, filter excluded elements — evaluated against candidate molecular formulas (e.g. filter included elements = C, S when isotope signals were detected in the full scan). Implement SMARTS substructure matching via the Chemistry Development Kit, computing per-candidate match counts N_included and N_excluded and scoring with Eq. 3 weights (W_include = W_exclude = 0.5) and the Eq. 4 normalization across the candidate list. Assemble the modules behind a single configuration interface so a user can enable any combination of filter vs. score and element vs. substructure per MetFrag run, and build the command-line tool from the MetFragRelaunched source with Maven. Validate on the unknown 199.0428 case, where SMARTS S(=O)(=O)OC on the exclusion list eliminates the alkyl sulfonate species.

## Related tools

- **MetFrag2.2** (Host implementation whose MetFragLib/CommandLine provide the InChIKey filter, element restriction modes, and SMARTS substructure scoring behind a single configuration interface) — https://github.com/ipb-halle/MetFragRelaunched
- **Chemistry Development Kit** (Performs the SMARTS substructure matching used to compute N_included and N_excluded match counts per candidate)
- **metfRag** (R wrapper (MetFragR) exposing the MetFrag library for scripted candidate-filtering runs) — https://github.com/ipb-halle/MetFragR/metfRag

## Examples

```
git clone https://github.com/ipb-halle/MetFragRelaunched.git; mvn clean install -pl MetFragLib -am
```

## Evaluation signals

- InChIKey filter reduces the candidate list to exactly one representative per first-InChIKey-block group, keeping the best-scored stereoisomer
- Element include-only mode with elements = C, S retains only candidates whose molecular formula contains C and S (and nothing outside the include list), and require/exclude modes behave accordingly
- SMARTS exclusion pattern S(=O)(=O)OC eliminates alkyl sulfonate species in the unknown 199.0428 test case, as in the 4-isopropylbenzenesulfonic acid example
- Substructure scores follow Eq. 3 with W_include = W_exclude = 0.5 and the Eq. 4 per-list normalization of match counts
- Enabling/disabling each module independently via the configuration interface changes the candidate list only through that module's effect

## Limitations

- Identification results (e.g. 4-isopropylbenzenesulfonic acid) could not be confirmed for lack of a sufficiently pure reference standard
- Element include filtering is justified by isotope signals detected in the full scan; without such signals the element choice is speculative
- Parameter, candidate, result, and ranking files for the EA, UF, EQEx, EQExPlus and HILL test datasets are only available as external ZIP archives, complicating reproduction

## Evidence

- [results] Stereoisomer filtering by first InChIKey block: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [abstract] Element/substructure filter rationale: "Candidates can be filtered or scored differently based on criteria like occurence of certain elements and/or substructures prior to fragmentation, or presence in so-called "suspect lists""
- [results] SMARTS exclusion example for unknown 199.0428: "adding the SMARTS S(=O)(=O)OC to the exclusion list eliminates the alkyl sulfonate species"
- [results] Element include filter based on isotope signals: "filter included elements = C, S (as isotope signals were detected in the full scan)"
- [readme] Building the filtering library from source: "mvn clean install -pl MetFragLib -am"

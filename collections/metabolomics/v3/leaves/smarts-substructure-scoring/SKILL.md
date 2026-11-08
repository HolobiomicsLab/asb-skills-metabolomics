---
name: smarts-substructure-scoring
description: Use when tandem-MS candidate lists from PubChem/ChemSpider retrieval
  or user-supplied SDFs contain stereoisomeric redundancy, when isotope signals reveal
  required elements (e.g. C, S), or when known substructures (e.g. sulfonate SMARTS)
  should bias candidate ranking.
when_to_use_negative:
- When the measured mass spectrum is dominated by one or a few fragments (e.g. a water
  loss) explainable by most candidates — substructure scoring cannot compensate for
  uninformative spectra
- When candidates should be restricted by spectral-library membership rather than
  structure rules — use suspect lists (e.g. STOFF-IDENT InChIKeys) instead
- When stereochemistry itself must be resolved — the InChIKey filter intentionally
  discards stereoisomeric distinctions by keeping only the best-scored candidate per
  first block
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MetFrag2.2
  role: Hosts the InChIKey filtering, element restriction, and SMARTS substructure
    filter/scoring modules applied to candidates prior to fragmentation
  repo: http://c-ruttkies.github.io/MetFrag/
- name: ChemSpider
  role: Source of candidate structures retrieved via web services prior to filtering
- name: PubChem
  role: Source of candidate structures retrieved via web services prior to filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:4208cab159b1e8fa23302c10fa4254bac0772ebff0322a869936777e9551f151
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpider
  - MetFrag2.2
  - PubChem
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# smarts-substructure-scoring

## Summary

MetFrag2.2's candidate filtering/scoring modules that (a) reduce stereoisomer redundancy via first-InChIKey-block filtering, (b) restrict candidates by element include/require/exclude lists, and (c) score or filter candidates by SMARTS substructure inclusion/exclusion matches computed with the Chemistry Development Kit (Eqs. 3-4, W_include = W_exclude = 0.5).

## When to use

Use when tandem-MS candidate lists from PubChem/ChemSpider retrieval or user-supplied SDFs contain stereoisomeric redundancy, when isotope signals reveal required elements (e.g. C, S), or when known substructures (e.g. sulfonate SMARTS) should bias candidate ranking.

## When NOT to use

- When the measured mass spectrum is dominated by one or a few fragments (e.g. a water loss) explainable by most candidates — substructure scoring cannot compensate for uninformative spectra
- When candidates should be restricted by spectral-library membership rather than structure rules — use suspect lists (e.g. STOFF-IDENT InChIKeys) instead
- When stereochemistry itself must be resolved — the InChIKey filter intentionally discards stereoisomeric distinctions by keeping only the best-scored candidate per first block

## Inputs

- Candidate structure list from PubChem/ChemSpider web service retrieval or a user-supplied SDF
- Element restriction lists (include-only, required, excluded elements), e.g. 'included elements = C, S'
- SMARTS substructure patterns for inclusion (e.g. S(=O)(=O)O, CCc1ccc(cc1)S(=O)(=O)O) and exclusion (e.g. S(=O)(=O)OC)
- MetFrag run configuration/parameter file

## Outputs

- Stereoisomer-reduced candidate list (one representative per first InChIKey block)
- Element-filtered candidate list per mode (include/require/exclude)
- Per-candidate substructure match counts (N_included, N_excluded) and normalized substructure score contributions
- Final ranked candidate list (e.g. result and ranking files as in the EA/UF/EQEx/EQExPlus/HILL datasets)

## How to apply

Parse the candidate list and compute each candidate's InChIKey, then group candidates by the first InChIKey block (molecular skeleton, not stereochemistry) and keep only the single best-scored representative per group, removing stereoisomer redundancy before further processing. Apply element restrictions in three modes — filter included elements, filter required elements, filter excluded elements — against candidate molecular formulas (e.g. include elements = C, S when isotope signals were detected in the full scan). Implement SMARTS substructure matching via the Chemistry Development Kit, counting per candidate the number of included (N_included) and excluded (N_excluded) substructures, and score them with the Eq. 3 weights (W_include = W_exclude = 0.5) and the Eq. 4 per-candidate normalization across the candidate list. For the unknown 199.0428 case, adding the SMARTS S(=O)(=O)OC to the exclusion list eliminates alkyl sulfonate species, while S(=O)(=O)O and CCc1ccc(cc1)S(=O)(=O)O serve as inclusion SMARTS. Assemble the three modules behind a single configuration interface so a user can enable any combination (filter vs. score, element vs. substructure) per MetFrag run, applied prior to fragmentation.

## Related tools

- **MetFrag2.2** (Hosts the InChIKey filtering, element restriction, and SMARTS substructure filter/scoring modules applied to candidates prior to fragmentation) — http://c-ruttkies.github.io/MetFrag/
- **ChemSpider** (Source of candidate structures retrieved via web services prior to filtering)
- **PubChem** (Source of candidate structures retrieved via web services prior to filtering)

## Examples

```
git clone https://github.com/ipb-halle/MetFragRelaunched.git && mvn clean install -pl MetFragCommandLine -am
```

## Evaluation signals

- InChIKey filter reduces stereoisomer redundancy to exactly one candidate per first-InChIKey-block group
- Element modes produce the expected include-only / require / exclude behavior on a test candidate list (e.g. C, S include keeps only C- and S-containing candidates)
- SMARTS substructure scoring matches the Eq. 3-4 scheme with W_include = W_exclude = 0.5 and per-candidate normalization across the list
- For the unknown 199.0428, adding exclusion SMARTS S(=O)(=O)OC eliminates alkyl sulfonate species from the ranked list
- Modules can be independently toggled (filter vs. score, element vs. substructure) via the single configuration interface

## Limitations

- Element/substructure filtering discards candidates outright when used as filters, so over-restrictive SMARTS or element lists can eliminate the correct candidate
- The InChIKey filter removes stereoisomeric distinctions, which is undesirable if stereochemistry matters for the identification
- Parameter files and candidate/result/ranking files for the EA, UF, EQEx, EQExPlus and HILL datasets are only available as external ZIP archives, complicating reproduction
- No version number for MetFrag2.2 stated beyond the project name; no changelog found

## Evidence

- [abstract] InChIKey filtering to reduce candidate redundancy due to stereoisomerism: "as well as InChIKey filtering to reduce candidate redundancy due to stereoisomerism"
- [results] Stereoisomer filter keeps best-scored candidate per first InChIKey block: [evidence span withheld — anchor status: not_found]
- [results] Element filter example C, S from isotope signals: "filter included elements = C, S (as isotope signals were detected in the full scan)"
- [results] Exclusion SMARTS removes alkyl sulfonates for unknown 199.0428: "adding the SMARTS S(=O)(=O)OC to the exclusion list eliminates the alkyl sulfonate species"
- [abstract] Candidates can be filtered or scored by elements/substructures prior to fragmentation: "Candidates can be filtered or scored differently based on criteria like occurence of certain elements and/or substructures prior to fragmentation"
- [readme] MetFragRelaunched build and command line entry points: "mvn clean install -pl MetFragCommandLine -am"

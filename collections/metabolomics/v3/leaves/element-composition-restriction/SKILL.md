---
name: element-composition-restriction
description: Use when ranking candidate structures from HR-MS/MS compound databases
  (PubChem, ChemSpider) or a user-supplied SDF and isotope signals or prior chemical
  knowledge imply certain elements must be present, required, or excluded.
when_to_use_negative:
- Candidates are already a small, curated suspect list (e.g. STOFF-IDENT entries converted
  to InChIKeys) where element restriction would discard valid suspects
- No isotope patterns or prior chemical knowledge are available to justify which elements
  to include, require, or exclude — an unjustified element filter may remove the true
  candidate
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0121
tools:
- name: MetFrag2.2
  role: Implements the element restriction module (include/require/exclude element
    modes) behind its configuration interface, applied before fragmentation or as
    part of candidate ranking
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFrag
  role: In silico fragmenter whose candidate-retrieval and scoring pipeline the element
    filter feeds into
  repo: http://c-ruttkies.github.io/MetFrag/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:fe99362c42b69e9544bc0749ab89c44ee021e81672bc38dc8d2e7b233e406d7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - MetFrag
  - MetFrag2.2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# element-composition-restriction

## Summary

MetFrag2.2's element restriction module restricts candidate structures by their molecular formulas using three modes: filter-included elements, filter-required elements, and filter-excluded elements, applied before fragmentation or as part of candidate ranking.

## When to use

Use when ranking candidate structures from HR-MS/MS compound databases (PubChem, ChemSpider) or a user-supplied SDF and isotope signals or prior chemical knowledge imply certain elements must be present, required, or excluded.

## When NOT to use

- Candidates are already a small, curated suspect list (e.g. STOFF-IDENT entries converted to InChIKeys) where element restriction would discard valid suspects
- No isotope patterns or prior chemical knowledge are available to justify which elements to include, require, or exclude — an unjustified element filter may remove the true candidate

## Inputs

- Candidate structure list from PubChem/ChemSpider retrieval or a user-supplied SDF
- Element restriction configuration (include-elements list, e.g. C, S; require-elements list; exclude-elements list)
- Molecular formula per candidate (evaluated against the element modes)

## Outputs

- Element-filtered candidate list (candidates removed or retained per include/require/exclude mode)
- Filtered/scorred candidate ranking feedable into the MetFrag2.2 scoring and InChIKey stereoisomer-reduction pipeline
- Run report documenting which element modes were applied and their effect on the candidate list

## How to apply

After retrieving candidates (from PubChem/ChemSpider retrieval or a user-supplied SDF), evaluate each candidate's molecular formula against the element restriction configuration in one of three modes: 'filter included elements' (keep only candidates containing the specified elements), 'filter required elements' (candidates must contain all listed elements), or 'filter excluded elements' (drop candidates containing any listed element). In the paper's worked example for the unknown at m/z 199.0428, the include-elements parameter was set to C, S because isotope signals were detected in the full scan, and combined with SMARTS substructure include filters (S(=O)(=O)O and CCc1ccc(cc1)S(=O)(=O)O) plus the exclusion SMARTS S(=O)(=O)OC to eliminate alkyl sulfonate species. Element restriction can be applied purely as a filter (pre-fragmentation) or as a scoring term alongside other MetFrag2.2 scoring terms, and the module is assembled behind a single configuration interface so any combination of element and substructure filtering/scoring can be enabled per run.

## Related tools

- **MetFrag2.2** (Implements the element restriction module (include/require/exclude element modes) behind its configuration interface, applied before fragmentation or as part of candidate ranking) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFrag** (In silico fragmenter whose candidate-retrieval and scoring pipeline the element filter feeds into) — http://c-ruttkies.github.io/MetFrag/

## Evaluation signals

- Each of the three modes (include, require, exclude) produces its documented behavior on a test candidate list: include keeps only candidates containing the listed elements, require demands all listed elements, exclude drops any candidate containing a listed element
- The example include-elements = C, S configuration retains sulfur-containing candidates (e.g. benzenesulfonic acid species for unknown 199.0428) and removes candidates lacking C or S
- Element filtering composes correctly with SMARTS substructure include/exclude scoring (W_include = W_exclude = 0.5, Eq. 4 normalization) when both are enabled in one run
- Filtered output candidate lists for the EA, UF, EQEx, EQExPlus and HILL datasets should match the published result files in the external ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/

## Limitations

- Element restriction requires justification (e.g. detected isotope signals) — an incorrect element filter silently removes the correct candidate before fragmentation
- The module is one of several candidate-filtering/scoring options; misconfigured combinations with substructure or InChIKey filters can over-restrict the candidate space
- Parameter files, candidate, result and ranking files for the reference datasets are only available as external ZIP archives, complicating reproducibility checks

## Evidence

- [abstract] Candidates can be filtered or scored differently based on criteria like occurence of certain elements and/or substructures prior to fragmentation: "Candidates can be filtered or scored differently based on criteria like occurence of certain elements and/or substructures prior to fragmentation, or presence in so-called "suspect lists""
- [results] Element filter C, S applied because isotope signals were detected: "filter included elements = C, S (as isotope signals were detected in the full scan)"
- [results] Three element modes from task card finding: "adding the SMARTS S(=O)(=O)OC to the exclusion list eliminates the alkyl sulfonate species"

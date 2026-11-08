---
name: mass-difference-rule-filtering
description: Use when you have paired known-anchor and unknown-suspect molecules with
  precursor m/z values and a rule database of reaction mass shifts, and need to select
  only rules consistent with the observed mass difference before generating derivatives.
when_to_use_negative:
- No anchor molecule with known structure and precursor mass is available, so no anchor–suspect
  mass difference can be computed
- The observed precursor mass difference is unknown or unreliable (e.g., uncalibrated
  m/z), since rule selection is made at unit resolution
- You need structure candidates unrelated to biotransformations (e.g., formula-level
  annotation with SIRIUS), which is handled separately in the BAM workflow
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: BAM
  role: Identifies the relevant mass differences between suspects and anchors and
    selects the appropriate biotransformation rules corresponding to those mass differences
    before derivative generation
  repo: https://github.com/HassounLab/BAM
- name: PROXIMAL2
  role: Applies the mass-filtered biotransformation rules to the anchor molecule,
    identifying suitable reaction centers and generating candidate derivative products
  repo: https://github.com/HassounLab/PROXIMAL2
- name: GNN-SOM
  role: Ranks the mass-compatible derivative candidates by predicted site-of-metabolism
    likelihood
  repo: https://github.com/HassounLab/GNN-SOM
- name: RetroRules
  role: Source database of biotransformation rules whose mass shifts are matched against
    observed precursor mass differences
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:eaed9b72b3f57a0fe66cdcc64d3b3c78435e296ca71523dc1d0eaa0c5e672ab8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HassounLab/BAM
  tools:
  - BAM
  - BioServices
  - GNN-SOM
  - PROXIMAL2
  - PubChemPy
  - Python 3.9
  - RetroRules
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mass-difference-rule-filtering

## Summary

Filters a large biotransformation rule database (e.g., RetroRules) down to only those rules whose reaction mass shifts match the observed precursor mass difference between each anchor–suspect pair, so that downstream derivative generation only applies chemically plausible transformations.

## When to use

Use when you have paired known-anchor and unknown-suspect molecules with precursor m/z values and a rule database of reaction mass shifts, and need to select only rules consistent with the observed mass difference before generating derivatives.

## When NOT to use

- No anchor molecule with known structure and precursor mass is available, so no anchor–suspect mass difference can be computed
- The observed precursor mass difference is unknown or unreliable (e.g., uncalibrated m/z), since rule selection is made at unit resolution
- You need structure candidates unrelated to biotransformations (e.g., formula-level annotation with SIRIUS), which is handled separately in the BAM workflow

## Inputs

- molecules_of_interest CSV: suspect identifier, suspect mass, anchor identifier, anchor mass, and anchor SMILES (default BAM input from the global molecular network)
- reaction_list CSV of biotransformation rules with columns 'id', 'formula', 'EC' (e.g., RetroRules-derived: 42,307 unique rules from 84,614 reactant–product pairs)
- metabolites_list CSV of metabolites and their SMILES structures included in the reaction dataset

## Outputs

- Filtered set of mass-difference-compatible biotransformation rules per anchor–suspect pair (17,254 rules at unit resolution in the benchmark)
- Candidate derivative structures generated from the filtered rules (PROXIMAL2 JSON product files with GeneratedProduct, TemplateReaction, and QueryInformation fields)
- GNN-SOM-ranked derivative list feeding the benchmark table used to compute the 24.2% correct-structure assignment rate

## How to apply

Compute the precursor mass difference between each suspect and its anchor molecule in the molecules_of_interest file (identifier, masses, anchor SMILES). Match this observed mass difference against the mass shifts implied by the reaction rule database — in the BAM benchmark, 17,254 of the 42,307 unique rules extracted from the 351,704 RetroRules rules matched observed precursor mass differences at unit resolution, i.e., 41% of rules remained applicable. Pass only the matched rules (reaction_list with 'id', 'formula', 'EC' columns) to PROXIMAL2, which applies them to the anchor via suitable reaction centers, then rank the resulting derivative candidates with GNN-SOM site-of-metabolism likelihoods. The rationale is that mass-difference matching prunes the rule space before any structure generation, avoiding the enumeration of derivatives that could never produce the observed precursor.

## Related tools

- **BAM** (Identifies the relevant mass differences between suspects and anchors and selects the appropriate biotransformation rules corresponding to those mass differences before derivative generation) — https://github.com/HassounLab/BAM
- **PROXIMAL2** (Applies the mass-filtered biotransformation rules to the anchor molecule, identifying suitable reaction centers and generating candidate derivative products) — https://github.com/HassounLab/PROXIMAL2
- **GNN-SOM** (Ranks the mass-compatible derivative candidates by predicted site-of-metabolism likelihood) — https://github.com/HassounLab/GNN-SOM
- **RetroRules** (Source database of biotransformation rules whose mass shifts are matched against observed precursor mass differences)

## Examples

```
sh runBAM.sh  # with reaction_list (RetroRules rules) and molecules_of_interest (anchor–suspect pairs) configured so BAM matches precursor mass differences to rule mass shifts before PROXIMAL2 applies them
```

## Evaluation signals

- The fraction of rules surviving mass-difference matching should be ~41% of the RetroRules-derived rule set (17,254 of 42,307 unique rules at unit resolution)
- Every rule applied by PROXIMAL2 to a given anchor–suspect pair should imply the same mass shift as the observed suspect-minus-anchor precursor mass difference
- Derivatives generated from filtered rules should reproduce the benchmark: correct InChIKey (first 14 characters) matches in 4,171 of 17,271 derivative-generating queries (24.2% recall)
- Cofactor-only reactions and reactions whose substrates/products are entirely cofactors should have been removed during input generation, per PROXIMAL2's GenerateInput.py

## Limitations

- Matching is performed at unit resolution of precursor mass differences; higher-resolution filtering may change the applicable-rule set
- Rule databases can be extracted from RetroRules, KEGG, or MetaCyc, and the choice of source affects which mass shifts are available
- Even after mass filtering, PROXIMAL2 may fail to produce a chemically sensible SMILES for some derivatives (RDKit cannot find chemical sense about the prediction)

## Evidence

- [results] BAM identifies relevant mass differences between the suspects and the anchor, selects appropriate biotransformation rules corresponding to these mass differences: "BAM identifies relevant mass differences between the suspects and the anchor, selects appropriate biotransformation rules corresponding to these mass differences"
- [results] 17,254 rules matched at unit resolution: "observed precursor mass differences matched 17,254 biotransformation rules, corresponding to 41% of rules from the RetroRules database that are applicable for BAM"
- [results] Rule extraction from RetroRules: "yielding 42,307 unique rules—covering a wide range of possible biochemical transformations. These rules were extracted from the 351,704 biotransformation rules in RetroRules"
- [readme] molecules_of_interest input format: "molecules_of_interest = csv file of list of queries. It must have an identifier and mass for the suspect as well as anchor and a SMILES that represents the anchor."
- [intro] PROXIMAL2 applies rules to identify reaction centers: "BAM makes use of PROXIMAL211 to apply the biotransformation rules to the anchor molecule, identifying suitable reaction centers for the proposed structural changes."

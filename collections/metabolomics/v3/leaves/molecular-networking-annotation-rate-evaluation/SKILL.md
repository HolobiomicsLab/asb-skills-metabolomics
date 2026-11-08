---
name: molecular-networking-annotation-rate-evaluation
description: Use when you have a molecular network–derived set of anchor–suspect spectrum
  pairs with known SMILES for anchors, and you need to quantify how many suspects
  a rule-based annotator correctly recovers.
when_to_use_negative:
- The suspects already have trusted spectral library identifications — this skill
  measures rule-based annotation recall on unresolved anchor–suspect pairs, not library-search
  validation
- You lack structurally unique, annotated spectrum pairs from a molecular network
  (e.g., only raw MS/MS spectra without anchor SMILES) — the benchmark requires the
  60,368-pair anchor–suspect format
- You only need candidate derivative generation or SOM ranking for a single query
  — this skill is the full benchmark evaluation, not the derivation pipeline itself
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: BAM
  role: Main pipeline run via runBAM.sh to generate and rank candidate suspect structures
    from anchors using biotransformation rules
  repo: https://github.com/HassounLab/BAM
- name: PROXIMAL2
  role: Applies biotransformation rules to anchor molecules, identifying suitable
    reaction centers to generate candidate derivatives
  repo: https://github.com/HassounLab/PROXIMAL2
- name: GNN-SOM
  role: Graph neural network that predicts site-of-metabolism likelihood to rank putative
    derivatives, with average-rank tie resolution
  repo: https://github.com/HassounLab/GNN-SOM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c01565
    title: Molecular Structure Discovery for Untargeted Metabolomics Using Biotransformation
      Rules and Global Molecular Networking
  source_content_hash: sha256:8bb881f093d8451fafe9b7810bdda31c1399af8afcbc49d594f9f910592ff8e4
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
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# molecular-networking-annotation-rate-evaluation

## Summary

Benchmark protocol for measuring what fraction of anchor–suspect pairs in a global molecular network a biotransformation-rule-based annotator (BAM) can correctly resolve, yielding a recall metric such as the reported 24.2% correct-structure assignment rate.

## When to use

Use when you have a molecular network–derived set of anchor–suspect spectrum pairs with known SMILES for anchors, and you need to quantify how many suspects a rule-based annotator correctly recovers.

## When NOT to use

- The suspects already have trusted spectral library identifications — this skill measures rule-based annotation recall on unresolved anchor–suspect pairs, not library-search validation
- You lack structurally unique, annotated spectrum pairs from a molecular network (e.g., only raw MS/MS spectra without anchor SMILES) — the benchmark requires the 60,368-pair anchor–suspect format
- You only need candidate derivative generation or SOM ranking for a single query — this skill is the full benchmark evaluation, not the derivation pipeline itself

## Inputs

- Anchor–suspect pair dataset of 60,368 pairs (CSV: molecules_of_interest with suspect identifier, suspect mass, anchor identifier, anchor mass, and anchor SMILES) from Zenodo 10.5281/zenodo.13903946
- Biotransformation rule dataset (RetroRules dump; optionally KEGG reaction list CSV with columns 'id', 'formula', 'EC')
- Metabolites list CSV mapping metabolites to SMILES for the reaction dataset
- BAM code with PROXIMAL2 and GNN-SOM staged under the 'BAM-main' directory and 'proximal2' and 'som' conda environments created

## Outputs

- Per-pair benchmark table recording outcome of derivative generation, ranking, and correctness per anchor–suspect pair
- benchmark_results.csv with the overall correct-structure assignment rate (target: 24.2%)
- Generated derivative folders per query (JSON with GeneratedProduct SMILES, TemplateReaction, and QueryInformation fields from PROXIMAL2)

## How to apply

Load the anchor–suspect pair dataset (60,368 pairs derived from 30,184 structurally unique annotated spectrum pairs in the global molecular network) from the Zenodo deposit (10.5281/zenodo.13903946). Stage PROXIMAL2 and GNN-SOM under the BAM-main directory and create the 'proximal2' and 'som' conda environments as required by the BAM README. Run runBAM.sh with the global molecular network molecules_of_interest file as default input, extracting biotransformation rules from RetroRules (42,307 unique rules used; optionally KEGG) and applying PROXIMAL2 to generate candidate derivatives whose mass differences match observed precursor mass differences. Rank generated derivatives with GNN-SOM site-of-metabolism likelihoods, resolving rank ties by averaging duplicate ranks. Evaluate correctness per anchor–suspect pair by exact first-14-character InChIKey matching, record per-pair outcomes in a benchmark table, and compute the overall correct-structure assignment rate. Validation: the computed rate should reproduce the reported 24.2% (4,171 correct among 17,271 queries with at least one generated derivative).

## Related tools

- **BAM** (Main pipeline run via runBAM.sh to generate and rank candidate suspect structures from anchors using biotransformation rules) — https://github.com/HassounLab/BAM
- **PROXIMAL2** (Applies biotransformation rules to anchor molecules, identifying suitable reaction centers to generate candidate derivatives) — https://github.com/HassounLab/PROXIMAL2
- **GNN-SOM** (Graph neural network that predicts site-of-metabolism likelihood to rank putative derivatives, with average-rank tie resolution) — https://github.com/HassounLab/GNN-SOM

## Examples

```
sh runBAM.sh
```

## Evaluation signals

- Overall correct-structure assignment rate computed from benchmark_results.csv reproduces 24.2% (4,171 correct suspect structures out of 17,271 queries with ≥1 generated derivative)
- Input pair count equals 60,368 anchor–suspect pairs derived from 30,184 structurally unique annotated spectrum pairs
- Correctness per pair is judged by exact matching of the first 14 characters of the InChIKey between generated derivative and suspect structure
- Rule-matching stage reports 17,254 biotransformation rules matched to observed precursor mass differences at unit resolution (41% of applicable RetroRules rules)
- Ranked derivative lists show rank ties resolved by averaging duplicate ranks (sum of tied ranks divided by number of tied candidates)

## Limitations

- Only queries where BAM generated one or more derivatives (17,271 of the examined cases) count toward the recall denominator; cases with no generated derivative are excluded from the 24.2% figure
- Results depend on the choice of rule source — RetroRules was used in the benchmark (42,307 unique rules extracted from 351,704 RetroRules rules representing 84,614 reactant–product pairs), with KEGG as an optional alternative
- Derivative candidates whose SMILES RDKit cannot render chemical sense for may be missing product structures, limiting evaluability

## Evidence

- [abstract] BAM assigned correct molecular structures to 24.2% of examined anchor-suspect cases: "BAM was able to assign correct molecular structures to 24.2% of examined anchor-suspect cases"
- [results] 30,184 structurally unique annotated pairs yield 60,368 anchor–suspect pairs: "a set of 30,184 structurally unique, annotated spectrum pairs were derived from neighboring annotated nodes from this global molecular network, yielding 60,368 total anchor–suspect pairs"
- [methods] InChIKey-based correctness matching for benchmark table: [evidence span withheld — anchor status: not_found]
- [readme] README run command for the BAM pipeline: "Once the code and conda environments are structured as specified in the Installation and Requirements section  above, run the runBAM.sh file as it is"
- [results] Mass-difference rule matching stage defines which rules apply: "observed precursor mass differences matched 17,254 biotransformation rules, corresponding to 41% of rules from the RetroRules database that are applicable for BAM"

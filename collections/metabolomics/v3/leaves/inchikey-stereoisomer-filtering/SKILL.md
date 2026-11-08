---
name: inchikey-stereoisomer-filtering
description: Use when candidate lists retrieved from PubChem, ChemSpider or KEGG contain
  multiple stereoisomers sharing a first InChIKey block, and scores must be assigned
  per unique connectivity before ranking Top 1/3/5.
when_to_use_negative:
- When candidate sets contain no stereoisomeric duplicates (unique first InChIKey
  blocks), the filter is a no-op and adds no value
- When the analysis goal is to explicitly enumerate and report stereoisomer-level
  distinctions rather than collapse them to one connectivity-level representative
edam_operation: http://edamontology.org/operation_3679
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: MetFrag2.2
  role: Scoring engine whose candidate rankings the InChIKey filter is applied to;
    provides the InChIKey-based stereoisomer filtering in its candidate processing
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFragCommandLine
  role: Command-line MetFrag build from ipb-halle/MetFragRelaunched used to run scored-candidate
    processing with InChIKey filtering over dataset parameter files
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFragR/metfRag
  role: R interface to MetFragLib through which scored candidates and InChIKey filtering
    can be driven from R scripts
  repo: https://github.com/ipb-halle/MetFragR
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:a0b082215dfd2729daf3295fa31d6b97f539b71da00d46b0a56ec67938b08d05
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MetFrag2.2
  - MetFragCommandLine
  - MetFragLib
  - MetFragR/metfRag
  - OpenBabel
  - xcms
  edam_operation: http://edamontology.org/operation_3679
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# inchikey-stereoisomer-filtering

## Summary

Reduces candidate redundancy from stereoisomerism by comparing the first block of candidates' InChIKeys and retaining only the best-scored candidate per block before final ranking in MetFrag2.2 workflows.

## When to use

Use when candidate lists retrieved from PubChem, ChemSpider or KEGG contain multiple stereoisomers sharing a first InChIKey block, and scores must be assigned per unique connectivity before ranking Top 1/3/5.

## When NOT to use

- When candidate sets contain no stereoisomeric duplicates (unique first InChIKey blocks), the filter is a no-op and adds no value
- When the analysis goal is to explicitly enumerate and report stereoisomer-level distinctions rather than collapse them to one connectivity-level representative

## Inputs

- Scored candidate list per spectrum from MetFrag2.2 (candidate structures with InChIKeys and fragment-match scores, optionally reference and RT scores)
- Candidates retrieved from PubChem or ChemSpider (June 2015 snapshot) or supplied as user SDF

## Outputs

- Per-spectrum candidate ranking with one representative candidate per first InChIKey block
- Top 1 / Top 3 / Top 5 rank counts of correct candidates (e.g. ranking_summary.csv, verification_rankings.csv)

## How to apply

After MetFrag2.2 has scored candidates via in silico bond dissociation fragmentation (optionally plus reference counts and RT–log P user-defined scores), compute the InChIKey for each candidate and group candidates by the first InChIKey block, which encodes the molecular skeleton independent of stereochemistry. Within each group, keep only the candidate with the highest MetFrag score and discard the remaining stereoisomers, so that stereoisomeric duplicates cannot crowd the rank list. Apply this filtering after scoring but before counting Top 1, Top 3 and Top 5 ranks so that the correct compound's rank is not inflated or deflated by its stereochemical duplicates. The same filter is used consistently across the 473-spectrum HILL dataset (with fragmentation-only MetFrag2.2 vs CFM-ID comparisons) and the 824-spectrum validation datasets (UF, EQEx, EQExPlus), ensuring comparability of reported counts such as 30 vs 43 Top 1 ranks.

## Related tools

- **MetFrag2.2** (Scoring engine whose candidate rankings the InChIKey filter is applied to; provides the InChIKey-based stereoisomer filtering in its candidate processing) — http://c-ruttkies.github.io/MetFrag/
- **MetFragCommandLine** (Command-line MetFrag build from ipb-halle/MetFragRelaunched used to run scored-candidate processing with InChIKey filtering over dataset parameter files) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFragR/metfRag** (R interface to MetFragLib through which scored candidates and InChIKey filtering can be driven from R scripts) — https://github.com/ipb-halle/MetFragR

## Examples

```
mvn clean install -pl MetFragCommandLine -am && metfragcl -ParamFile EA.parameters -SpectrumPath spectra/ -DatabasePath candidates.sdf
```

## Evaluation signals

- No two retained candidates per spectrum share the same first InChIKey block
- Top 1 counts on the 473 merged HILL spectra with PubChem candidates reproduce the benchmark 30 (MetFrag2.2) and 43 (CFM-ID) fragmentation-only ranks
- Enhanced runs with reference + RT scoring reproduce 420 (ChemSpider), 336 (PubChem) and 343 (PubChem + CFM-ID) Top 1 ranks on the same spectra
- Rank counts are computed only after filtering, so removing the filter should visibly increase apparent Top 5 counts if stereoisomer duplicates were present
- Filtering is applied identically to both MetFrag2.2 and CFM-ID runs so the 30 vs 43 comparison remains apples-to-apples

## Limitations

- Filtering collapses stereoisomers, so distinctions that depend on stereochemistry cannot be resolved by fragment-match scoring alone
- Parameter files, spectral data, candidate, result and ranking files for the EA, UF, EQEx, EQExPlus and HILL datasets are only available as external ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/
- Retained best-scored representative per InChIKey block may not be the biologically correct stereoisomer; correctness is judged at the connectivity level

## Evidence

- [results] Stereoisomers filtered keeping the best-scored candidate by first InChIKey block: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [abstract] InChIKey filtering to reduce candidate redundancy due to stereoisomerism: "as well as InChIKey filtering to reduce candidate redundancy due to stereoisomerism"
- [abstract] Benchmark Top 1 counts to validate filtering-consistent ranking: "Using HR-MS/MS information only, MetFrag2.2 and CFM-ID had 30 and 43 Top 1 ranks, respectively, using PubChem as a database."
- [readme] MetFragCommandLine build used to run the filtering pipeline: "mvn clean install -pl MetFragCommandLine -am"

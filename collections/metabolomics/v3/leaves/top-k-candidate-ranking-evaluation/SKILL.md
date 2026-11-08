---
name: top-k-candidate-ranking-evaluation
description: Use when a tandem-MS candidate-ranking tool has been parameterized on
  one dataset and you must verify Top 1/3/5 hit rates on independent HR-MS/MS spectra
  with known compound identities.
when_to_use_negative:
- Input is already a ranked candidate list with no known ground-truth compound identities
  to score against — the skill requires verified compound identities for Top-k counting.
- You need de novo structure elucidation without any candidate database, since this
  skill evaluates rankings of pre-retrieved candidate structures from PubChem/ChemSpider
  rather than generating candidates.
- Spectra lack fragment peaks besides the precursor, as such spectra were excluded
  and cannot meaningfully discriminate candidates.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetFrag2.2
  role: Command-line candidate ranking engine run over all spectra of each dataset
    to produce candidate rankings evaluated at Top 1/3/5
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFragCommandLine
  role: Command-line wrapper used to execute MetFrag2.2 across all spectra per dataset
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFragLib
  role: Core Java library implementing in silico fragmentation, scoring and InChIKey
    filtering behind the ranking evaluation
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: xcms
  role: R package providing mzClust_hclust used to merge replicate spectra (eppm=5e-6,
    eabs=0.001 Da) before ranking
- name: OpenBabel
  role: Converts SMILES to InChIKeys when preparing suspect lists for candidate filtering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:fbdb9c9bf7147c79d045a51d17ffcf14449dc3bac58f74d591c671ff040f2dfb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag2.2
  - MetFragCommandLine
  - MetFragLib
  - OpenBabel
  - xcms
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# top-k-candidate-ranking-evaluation

## Summary

Evaluate MetFrag2.2 candidate rankings by counting correct compound identities in Top 1, Top 3 and Top 5 positions across merged HR-MS/MS spectra datasets, verifying that optimized scoring weights (ωFrag+ωRT+ωRefs=1) generalize to independent validation data.

## When to use

Use when a tandem-MS candidate-ranking tool has been parameterized on one dataset and you must verify Top 1/3/5 hit rates on independent HR-MS/MS spectra with known compound identities.

## When NOT to use

- Input is already a ranked candidate list with no known ground-truth compound identities to score against — the skill requires verified compound identities for Top-k counting.
- You need de novo structure elucidation without any candidate database, since this skill evaluates rankings of pre-retrieved candidate structures from PubChem/ChemSpider rather than generating candidates.
- Spectra lack fragment peaks besides the precursor, as such spectra were excluded and cannot meaningfully discriminate candidates.

## Inputs

- Merged HR-MS/MS spectra (e.g. from mzClust_hclust in xcms, eppm=5e-6, eabs=0.001 Da)
- Candidate lists retrieved from PubChem/ChemSpider as configured in MetFrag parameter files
- MetFrag parameter files specifying scoring weights and filters
- Known compound identities (ground truth) for each spectrum
- External ZIP archives of parameter, spectral, candidate, result and ranking files

## Outputs

- Per-dataset Top 1, Top 3 and Top 5 rank counts and percentages for correct candidates
- Aggregated ranking statistics across all spectra (e.g. 824 merged spectra)
- A single comma-separated table (verification_rankings.csv) of per-dataset ranking statistics

## How to apply

Merge replicate spectra with mzClust_hclust in xcms (eppm=5e-6, eabs=0.001 Da), retaining only spectra with at least one fragment peak besides the precursor, and retrieve candidate lists from PubChem/ChemSpider as configured in the parameter files. Run MetFrag2.2 (command-line version from c-ruttkies.github.io/MetFrag/) with the verified configuration: in silico bond-dissociation fragmentation, comparison with measured product ions, InChIKey filtering to keep the best-scored stereoisomer, and scoring weights summing to ωFrag+ωRT+ωRefs=1. For each dataset, execute MetFrag2.2 over all spectra and collect the resulting candidate rankings. Compare rankings against known compound identities, counting correct candidates in Top 1, Top 3 and Top 5 ranks per dataset and aggregating across all spectra. Write the per-dataset ranking statistics into a single comma-separated table (e.g. verification_rankings.csv). Correctness is judged by whether the optimized weights' performance on the original dataset falls within the range observed on independent validation datasets.

## Related tools

- **MetFrag2.2** (Command-line candidate ranking engine run over all spectra of each dataset to produce candidate rankings evaluated at Top 1/3/5) — http://c-ruttkies.github.io/MetFrag/
- **MetFragCommandLine** (Command-line wrapper used to execute MetFrag2.2 across all spectra per dataset) — http://c-ruttkies.github.io/MetFrag/
- **MetFragLib** (Core Java library implementing in silico fragmentation, scoring and InChIKey filtering behind the ranking evaluation) — https://github.com/ipb-halle/MetFragRelaunched
- **xcms** (R package providing mzClust_hclust used to merge replicate spectra (eppm=5e-6, eabs=0.001 Da) before ranking)
- **OpenBabel** (Converts SMILES to InChIKeys when preparing suspect lists for candidate filtering)

## Examples

```
mvn clean install -pl MetFragCommandLine -am && java -jar MetFragCommandLine/Matrix/MetFragCommandLine-*.jar -ParameterPath MetFragConfig.txt
```

## Evaluation signals

- Top 1 rank percentage on each independent validation dataset falls within a coherent range (e.g. 63–82 %) with the original training dataset (71 %) inside that range.
- Ranking counts are consistent when aggregated across all merged spectra (e.g. 824 spectra across UF, EQex, EQexPlus datasets).
- Per-dataset Top 1/Top 3/Top 5 counts are non-decreasing (Top1 ≤ Top3 ≤ Top5) for every dataset in the output table.
- Verification rankings table contains exactly one row (or column set) per dataset with Top 1/3/5 statistics, with no missing datasets among those configured.
- InChIKey filtering leaves only the best-scored stereoisomer per first-InChIKey-block group, so duplicate stereoisomer rankings do not inflate Top-k counts.

## Limitations

- Mass spectra dominated by one or only a few fragments (e.g. a water loss) that can be explained by most candidates lack information to distinguish candidates, limiting Top-k performance.
- Increasing database size (e.g. PubChem doubling since 2009) makes candidate ranking harder over time, so absolute Top-k percentages may not transfer to current database snapshots.
- Parameter files, spectral data, candidate, result and ranking files for the EA, UF, EQEx, EQExPlus and HILL datasets are only available as external ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/, which may break reproducibility if the deposit moves.
- Retention-time scoring depends on an RT–log P model established per dataset via tenfold cross-validation; it requires retention times and is unsuitable for datasets lacking RT information.

## Evidence

- [results] Spectra merged with mzClust_hclust retaining only those with a fragment peak besides precursor: [evidence span withheld — anchor status: not_found]
- [results] Weight optimization constraint summing to 1: [evidence span withheld — anchor status: not_found]
- [abstract] Verification of optimal parameters on three additional datasets totaling 824 spectra: "The optimal parameters and weights were verified using three additional datasets of 824 merged HR-MS/MS spectra in total."
- [results] Top 1 performance improvement enabling ranking evaluation context: "the median rank improved from 18.5 to 14.5, while the number of correct ranked candidates in the top 1, 3 and 5 improved from 8 to 9, 20 to 24 and 28 to 34"
- [readme] Command-line MetFrag2.2 availability for batch ranking runs: "MetFrag commandline version depends on MetFragLib"

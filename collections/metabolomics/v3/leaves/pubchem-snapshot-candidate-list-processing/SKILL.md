---
name: pubchem-snapshot-candidate-list-processing
description: Use when reproducing or benchmarking candidate ranking on a fixed compound
  dataset against historical database snapshots, e.g. quantifying how PubChem growth
  (338 vs 2508 average candidates per compound) degrades MetFrag ranks.
when_to_use_negative:
- When the goal is MetFrag2.2 evaluation with reference-count or retention-time (RT–log
  P) scoring — the 2006-vs-2009 snapshot comparison must use fragment-explanation
  scoring only to isolate the database-size effect
- When candidate lists are drawn from a current database download or ChemSpider/KEGG
  rather than the fixed historical PubChem snapshots — ranks are then not comparable
  to the published benchmarks
- When the dataset is not a fixed compound list (e.g. EA, UF, EQEx datasets) without
  per-snapshot candidate files, since the comparison depends on identical spectra
  against differing candidate pools
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2258
tools:
- name: MetFragCommandLine
  role: Batch-scores the 102 spectra against each snapshot's candidate lists using
    the MetFrag2010 bond-dissociation fragmenter; built with Maven from MetFragRelaunched
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFrag
  role: Original 2010 in silico fragmenter whose bond-dissociation fragmentation and
    fragment-explanation scoring define the ranking under test
  repo: http://c-ruttkies.github.io/MetFrag/
- name: Maven
  role: Build system used to compile the MetFrag command line tool (`mvn clean install
    -pl MetFragCommandLine -am`)
- name: PubChem
  role: Source of the Feb. 2006 and 2009 candidate structure snapshots whose growth
    (doubled since 2009) is the variable under study
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:29a6e673a0fcf28176d2e65e6701cf6e8bdaa971e37d464c207a405976ff504a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - Maven
  - MetFrag
  - MetFrag2.2
  - MetFragCommandLine
  - PubChem
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pubchem-snapshot-candidate-list-processing

## Summary

Prepares and compares versioned PubChem candidate-structure snapshots (e.g. Feb. 2006 vs 2009) for a fixed compound set, so that database-growth effects on in silico fragmentation ranking (MetFrag2010) can be quantified per snapshot.

## When to use

Use when reproducing or benchmarking candidate ranking on a fixed compound dataset against historical database snapshots, e.g. quantifying how PubChem growth (338 vs 2508 average candidates per compound) degrades MetFrag ranks.

## When NOT to use

- When the goal is MetFrag2.2 evaluation with reference-count or retention-time (RT–log P) scoring — the 2006-vs-2009 snapshot comparison must use fragment-explanation scoring only to isolate the database-size effect
- When candidate lists are drawn from a current database download or ChemSpider/KEGG rather than the fixed historical PubChem snapshots — ranks are then not comparable to the published benchmarks
- When the dataset is not a fixed compound list (e.g. EA, UF, EQEx datasets) without per-snapshot candidate files, since the comparison depends on identical spectra against differing candidate pools

## Inputs

- HILL dataset spectral data files for the 102 compounds (from CHIN-D-15-00088 ZIP archives)
- MetFrag parameter files for the HILL dataset
- February 2006 PubChem candidate structure snapshot for the 102 compounds
- 2009 PubChem candidate structure snapshot for the 102 compounds
- MetFrag2010 fragment-explanation scoring configuration (no reference or retention score terms)

## Outputs

- Per-compound rank of the correct candidate for each PubChem snapshot
- Median rank and average candidates per compound per snapshot (expected: 8 / 338 for 2006; 31.5 / 2508 for 2009)
- metfrag2010_hill_ranking_results.csv comparing the two snapshots

## How to apply

Build the MetFrag command line tool from source with Maven (`mvn clean install -pl MetFragCommandLine -am`, optionally with `-DskipTests`). Obtain the HILL dataset parameter, spectral data, candidate, result and ranking files from the CHIN-D-15-00088 ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/, together with the February 2006 and 2009 PubChem candidate structure snapshots for the 102 Hill compounds. Run the MetFragCommandLine batch scoring with the original MetFrag2010 bond-dissociation fragmenter over the 102 spectra against each snapshot's candidate lists using fragment-explanation scoring only (no MetFrag2.2 reference or retention terms). For each snapshot, compute the rank of the correct candidate per compound, the median rank, and the average number of candidates per compound, then compare against the reported benchmarks (median rank 8 with 338 average candidates for 2006; median rank 31.5 with 2508 for 2009). The rationale is that PubChem roughly doubled since 2009, making candidate ranking harder over time for the same spectra.

## Related tools

- **MetFragCommandLine** (Batch-scores the 102 spectra against each snapshot's candidate lists using the MetFrag2010 bond-dissociation fragmenter; built with Maven from MetFragRelaunched) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFrag** (Original 2010 in silico fragmenter whose bond-dissociation fragmentation and fragment-explanation scoring define the ranking under test) — http://c-ruttkies.github.io/MetFrag/
- **Maven** (Build system used to compile the MetFrag command line tool (`mvn clean install -pl MetFragCommandLine -am`))
- **PubChem** (Source of the Feb. 2006 and 2009 candidate structure snapshots whose growth (doubled since 2009) is the variable under study)

## Examples

```
mvn clean install -pl MetFragCommandLine -am -DskipTests
```

## Evaluation signals

- Median rank against the 2006 snapshot reproduces the published value of 8 with ~338 average candidates per compound
- Median rank against the 2009 snapshot reproduces the published value of 31.5 with ~2508 average candidates per compound
- The 2009 snapshot yields a larger candidate pool per compound than 2006 (≈2508 vs ≈338), consistent with PubChem doubling since 2009
- Ranking uses fragment-explanation scoring only — results must match the MetFrag2010 configuration, not MetFrag2.2's improved median rank of 14.5 on the same HILL dataset
- Ranks are computed for exactly 102 compounds in both runs

## Limitations

- Database growth makes candidate ranking more challenging over time; results are snapshot-specific and not transferable to current PubChem releases
- MetFrag2.2-era features (InChIKey stereoisomer filtering, reference counts, retention-time/log P terms) must be disabled for a faithful MetFrag2010 baseline
- Parameter files, spectral data, candidate, result and ranking files for the HILL dataset are only available as external ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/, so reproduction depends on that archive remaining accessible

## Evidence

- [intro] Database growth effect on ranking: "As PubChem is now double the size of the 2009 version, the candidate ranking becomes more challenging over time"
- [results] MetFrag2.2 vs MetFrag2010 baseline distinction on HILL dataset: "the median rank improved from 18.5 to 14.5, while the number of correct ranked candidates in the top 1, 3 and 5 improved from 8 to 9"
- [results] Reproduction benchmark for the two snapshots: [evidence span withheld — anchor status: not_found]
- [discussion] External archive availability of dataset files: "The parameter files including the spectral data, the candidate, result and ranking files of the used EA, UF, EQEx, EQExPlus and HILL datasets are available at"
- [readme] Command line build procedure: "mvn clean install -pl MetFragCommandLine -am"

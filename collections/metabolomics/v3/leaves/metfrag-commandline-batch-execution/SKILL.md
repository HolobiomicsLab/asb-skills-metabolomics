---
name: metfrag-commandline-batch-execution
description: Use when you need to score many tandem mass spectra against pre-retrieved
  candidate lists (e.g. SDF/PubChem snapshot SDFs for ~102 compounds) and compute
  candidate ranks in a scripted, reproducible batch run.
when_to_use_negative:
- When the goal is to use the MetFrag2.2 extensions (reference/retention scoring,
  element/substructure filters, InChIKey filtering) rather than reproduce the fragment-explanation-only
  MetFrag2010 baseline
- When only a handful of spectra need interactive evaluation — the MetFragWeb interface
  or MetFragR (metfRag) is simpler than configuring batch parameter files
- When candidate structures have not yet been retrieved — the command line tool scores
  supplied candidate lists, it does not perform the PubChem/ChemSpider/KEGG retrieval
  itself
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetFragCommandLine
  role: Batch execution of MetFrag scoring over multiple spectra and candidate lists;
    built via Maven from the cloned repository
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFrag
  role: Underlying in silico fragmenter performing the bond-dissociation fragmentation
    and fragment-explanation candidate scoring
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFragR (metfRag)
  role: Alternative R interface to MetFrag for running candidate ranking from R
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:0ceda58eb04e9fc8235f8eeb408d60e50702efa01b7a01f31c31d575e73191a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ipb-halle/MetFragRelaunched
  tools:
  - Maven
  - MetFrag
  - MetFrag2.2
  - MetFragCommandLine
  - MetFragR (metfRag)
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metfrag-commandline-batch-execution

## Summary

Builds the MetFrag command line tool with Maven and runs batch scoring of HR-MS/MS spectra against candidate structure lists (e.g. PubChem snapshots) using the in silico bond-dissociation fragmenter, enabling reproducible large-scale candidate ranking without the web interface.

## When to use

Use when you need to score many tandem mass spectra against pre-retrieved candidate lists (e.g. SDF/PubChem snapshot SDFs for ~102 compounds) and compute candidate ranks in a scripted, reproducible batch run.

## When NOT to use

- When the goal is to use the MetFrag2.2 extensions (reference/retention scoring, element/substructure filters, InChIKey filtering) rather than reproduce the fragment-explanation-only MetFrag2010 baseline
- When only a handful of spectra need interactive evaluation — the MetFragWeb interface or MetFragR (metfRag) is simpler than configuring batch parameter files
- When candidate structures have not yet been retrieved — the command line tool scores supplied candidate lists, it does not perform the PubChem/ChemSpider/KEGG retrieval itself

## Inputs

- MetFrag parameter files including spectral data (peak lists of HR-MS/MS spectra, e.g. the HILL dataset files from the CHIN-D-15-00088 ZIP archives)
- Candidate structure files (SDF) per compound, e.g. February 2006 and 2009 PubChem snapshot candidate lists for 102 compounds
- Compiled MetFragCommandLine jar built from the ipb-halle/MetFragRelaunched sources (Java 21, Apache Maven 3.8)

## Outputs

- Per-compound MetFrag result and ranking files listing scored candidates
- Computed rank of the correct candidate per compound (e.g. metfrag2010_hill_ranking_results.csv)
- Summary statistics: median rank and average candidates per compound (baseline: median rank 8 / 338 avg candidates on 2006 PubChem; median rank 31.5 / 2508 avg candidates on 2009 PubChem)

## How to apply

Clone the MetFragRelaunched repository and build the command line module with Maven (`mvn clean install -pl MetFragCommandLine -am`, optionally with `-DskipTests`); the command line version depends on MetFragLib. Supply the MetFrag parameter file containing the peak list (spectral data) and the candidate file (e.g. a PubChem snapshot SDF for the compounds of interest), selecting the original MetFrag2010 bond-dissociation fragmenter and fragment-explanation scoring only when reproducing the 2010 baseline. Run the identical batch configuration once per candidate set (e.g. the Feb. 2006 vs. 2009 PubChem snapshots) so that rank differences are attributable to database growth alone — PubChem having roughly doubled since 2009 makes ranking harder over time. For each compound, compute the rank of the correct candidate, then summarize with the median rank and the average number of candidates per compound.

## Related tools

- **MetFragCommandLine** (Batch execution of MetFrag scoring over multiple spectra and candidate lists; built via Maven from the cloned repository) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFrag** (Underlying in silico fragmenter performing the bond-dissociation fragmentation and fragment-explanation candidate scoring) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFragR (metfRag)** (Alternative R interface to MetFrag for running candidate ranking from R) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
git clone https://github.com/ipb-halle/MetFragRelaunched.git && mvn clean install -pl MetFragCommandLine -am -DskipTests
```

## Evaluation signals

- Reproduced median rank of 8 with ~338 average candidates per compound against the Feb. 2006 PubChem snapshot on the Hill 102-compound dataset
- Reproduced median rank of 31.5 with ~2508 average candidates per compound against the 2009 PubChem snapshot
- Maven build completes for MetFragCommandLine (and MetFragLib dependency) without errors, with or without -DskipTests
- Each of the 102 compounds yields a result/ranking file with a computed rank for the correct candidate; no compounds dropped from the batch
- Ranking uses fragment-explanation scoring only — no MetFrag2.2 reference-count or retention (RT–logP) terms should influence scores

## Limitations

- Parameter files including spectral data, candidate, result and ranking files for the EA, UF, EQEx, EQExPlus and HILL datasets are only available as external ZIP archives (http://msbi.ipb-halle.de/download/CHIN-D-15-00088/), not in the repository
- Larger candidate databases (PubChem is now double its 2009 size) make fragment-explanation-only ranking substantially harder, inflating median ranks
- Spectra dominated by one or few fragments (e.g. a water loss) explainable by most candidates do not contain enough information to distinguish candidates

## Evidence

- [readme] Build MetFragCommandLine via Maven: "mvn clean install -pl MetFragCommandLine -am
```

- skipping test during build by<br>
```bash
mvn clean install -pl MetFragCommandLine -am -DskipTests"
- [methods] Run MetFragCommandLine batch scoring: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark comparison target: [evidence span withheld — anchor status: not_found]
- [results] Baseline benchmark values: [evidence span withheld — anchor status: not_found]
- [intro] Database growth increases ranking difficulty: "As PubChem is now double the size of the 2009 version, the candidate ranking becomes more challenging over time"
- [discussion] External dataset archives: "The parameter files including the spectral data, the candidate, result and ranking files of the used EA, UF, EQEx, EQExPlus and HILL datasets are available at"

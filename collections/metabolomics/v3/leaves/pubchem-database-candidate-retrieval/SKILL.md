---
name: pubchem-database-candidate-retrieval
description: 'Use when ranking unknowns from HR-MS/MS spectra requires candidate structures
  from a large open compound database: you have a precursor m/z (e.g. 199.0428) and
  need PubChem candidates for fragmentation-based scoring.'
when_to_use_negative:
- The unknown is already covered by a small, specific database — the article stresses
  that smaller, specific databases (KEGG, HMDB, STOFF-IDENT, NORMAN suspects, MassBank)
  greatly improve the chance of ranking the correct answer highly, so prefer those
  over full PubChem when available.
- You need retention-time-aware ranking from the start — PubChem retrieval alone supplies
  structures; RT–logP weighting and user-defined scores must be added afterwards in
  MetFrag2.2.
- The input is a reference-spectrum library match (e.g. MassBank) rather than an unknown
  requiring de novo candidate retrieval.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0157
tools:
- name: MetFrag2.2
  role: Command-line tool that performs the PubChem database search by precursor mass,
    applies InChIKey first-block filtering, and scores the retrieved candidates by
    in silico bond dissociation fragmentation before ranking.
  repo: http://c-ruttkies.github.io/MetFrag/
- name: MetFragWeb
  role: Web interface of the relaunched MetFrag stack that queries PubChem (optionally
    via a configured LocalPubChemDatabase) and exposes the candidate retrieval step
    interactively.
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: MetFragCommandLine
  role: Command-line module of MetFragRelaunched used to run the PubChem candidate
    retrieval and InChIKey-filtered ranking in batch over the 473-spectrum dataset.
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:925a010bd2da0e44ff06d73d245d61d000e30413ce9a711c66d08bd6214be342
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MetFrag2.2
  - MetFragCommandLine
  - MetFragWeb
  - PubChem
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# pubchem-database-candidate-retrieval

## Summary

Retrieves candidate structures from the PubChem compound database by precursor formula mass (e.g. ppm-mass windows around 199.04284) and optionally applies InChIKey first-block filtering to collapse stereoisomers, supplying a reduced SDF candidate set to downstream in silico fragmenters such as MetFrag2.2 and CFM-ID.

## When to use

Use when ranking unknowns from HR-MS/MS spectra requires candidate structures from a large open compound database: you have a precursor m/z (e.g. 199.0428) and need PubChem candidates for fragmentation-based scoring.

## When NOT to use

- The unknown is already covered by a small, specific database — the article stresses that smaller, specific databases (KEGG, HMDB, STOFF-IDENT, NORMAN suspects, MassBank) greatly improve the chance of ranking the correct answer highly, so prefer those over full PubChem when available.
- You need retention-time-aware ranking from the start — PubChem retrieval alone supplies structures; RT–logP weighting and user-defined scores must be added afterwards in MetFrag2.2.
- The input is a reference-spectrum library match (e.g. MassBank) rather than an unknown requiring de novo candidate retrieval.

## Inputs

- Precursor m/z or exact mass of the unknown (e.g. 199.04284 from a merged Orbitrap HR-MS/MS spectrum)
- Optional local PubChem database configuration (MySQL/PostgreSQL with mass, formula, InChI, InChIKey1/2, CID, SMILES, compound-name columns) for MetFragWeb
- Optional suspect list of InChIKeys (e.g. STOFF-IDENT SMILES converted to InChIKeys with OpenBabel)
- Optional SMARTS element/substructure filters (e.g. elements = C, S; SMARTS S(=O)(=O)O)

## Outputs

- Set of PubChem candidate structures (SDF or identifier list) matching the precursor mass for each spectrum
- Stereoisomer-reduced candidate ranking (one best candidate per first InChIKey block) feeding Top 1/Top 5 rank counts
- Candidate file with formula, InChIKey, and SMILES per entry suitable for MetFrag2.2 / CFM-ID fragmentation scoring

## How to apply

Query PubChem web services with the precursor formula mass (e.g. 199.04284) to retrieve all candidate structures in an appropriate ppm mass window, as done in the June 2015 snapshot for the 473 merged Orbitrap HR-MS/MS spectra. Because PubChem's doubling in size since 2009 makes ranking harder, apply the article's redundancy control: score candidates, then keep only the best-scored candidate per first InChIKey block to collapse stereoisomers. If local speed is needed, MetFragWeb supports pointing LocalPubChemDatabase at a local MySQL/PostgreSQL mirror with mass, formula, InChI, InChIKey, SMILES and CID columns rather than hitting the live web service. Pass the resulting candidate list (SDF or database identifiers) to MetFrag2.2 or CFM-ID for fragmentation-only scoring, optionally combined with MetFrag2.2's element/substructure filters (e.g. elements = C, S when isotope signals are detected) before fragmentation.

## Related tools

- **MetFrag2.2** (Command-line tool that performs the PubChem database search by precursor mass, applies InChIKey first-block filtering, and scores the retrieved candidates by in silico bond dissociation fragmentation before ranking.) — http://c-ruttkies.github.io/MetFrag/
- **MetFragWeb** (Web interface of the relaunched MetFrag stack that queries PubChem (optionally via a configured LocalPubChemDatabase) and exposes the candidate retrieval step interactively.) — https://github.com/ipb-halle/MetFragRelaunched
- **MetFragCommandLine** (Command-line module of MetFragRelaunched used to run the PubChem candidate retrieval and InChIKey-filtered ranking in batch over the 473-spectrum dataset.) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
Rscript -e 'install_github("ipb-halle/MetFragR/metfRag")'
```

## Evaluation signals

- With HR-MS/MS fragmentation-only scoring on the 473 merged PubChem-candidate spectra, Top 1 counts must reproduce 30 (MetFrag2.2) and 43 (CFM-ID).
- After InChIKey filtering, no two retained candidates for a spectrum may share the same first InChIKey block.
- Every retrieved candidate's monoisotopic mass must lie within the ppm window defined around the precursor mass (e.g. 199.04284 ± 5 ppm, consistent with the eppm = 5×10-6 used for spectral merging).
- Suspect-list candidates (e.g. STOFF-IDENT entries converted to InChIKeys via OpenBabel) must be present in the supplied candidate list before scoring.
- Median rank on the Hill et al. subset should improve relative to MetFrag2010 when the retrieved PubChem candidates are re-scored with MetFrag2.2 (18.5 → 14.5 in the article).

## Limitations

- PubChem has roughly doubled in size since the 2009 MetFrag version, so the same precursor mass now returns more candidates and the ranking task becomes harder over time.
- Candidate retrieval alone is insufficient when spectra are dominated by one or a few fragments (e.g. a water loss) explained by most candidates — additional reference/retention information or filtering is needed.
- Candidate, result and ranking files for the EA, UF, EQEx, EQExPlus and HILL datasets used in the paper are only available as external ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/, so exact candidate sets may not be reproducible from the article alone.
- MetFragWeb's PubChem web-service queries require a ChemSpider token and, behind proxies, explicit proxy settings for the database services in settings.properties.

## Evidence

- [intro] Retrieval of PubChem candidates by mass precedes fragmentation scoring: [evidence span withheld — anchor status: not_found]
- [results] InChIKey first-block filtering collapses stereoisomers after scoring: "Stereoisomers were filtered to keep only the best scored candidate based on the comparison of the first part of the candidates’ InChIKeys"
- [abstract] Candidates can be pre-filtered by elements or substructures before fragmentation: "Candidates can be filtered or scored differently based on criteria like occurence of certain elements and/or substructures prior to fragmentation"
- [results] Element filter example tied to isotope evidence: "filter included elements = C, S (as isotope signals were detected in the full scan)"
- [results] Suspect lists are supplied to MetFrag as InChIKey lists: "the SMILES were converted to InChIKeys using OpenBabel and given as input to MetFrag as a suspect list"
- [abstract] Top 1 benchmark counts for PubChem candidates with fragmentation-only scoring: "Using HR-MS/MS information only, MetFrag2.2 and CFM-ID had 30 and 43 Top 1 ranks, respectively, using PubChem as a database."
- [intro] Larger PubChem makes ranking harder over time: "As PubChem is now double the size of the 2009 version, the candidate ranking becomes more challenging over time"
- [readme] MetFragWeb supports a local PubChem mirror for faster queries: "to speed up database queries you can set up local repositories by setting up MySQL or PostgreSQL databases"

---
name: candidate-structure-retrieval-from-pubchem-chemspider
description: Use when tandem-MS spectra need candidate structures scored by in silico
  fragmentation and no candidate list exists yet; trigger on a precursor neutral mass
  or molecular formula plus a chosen compound database (PubChem, ChemSpider).
when_to_use_negative:
- Candidate lists are already provided (e.g. the candidate files shipped in the EA/UF/EQEx/EQExPlus/HILL
  dataset ZIP archives) — skip retrieval and use them directly
- The target space is a small, specific database (KEGG, HMDB, STOFF-IDENT, NORMAN
  suspects) rather than the large general databases — the article notes smaller, specific
  databases greatly improve top-rank chances
- The identification goal is a direct spectral library match (e.g. MassBank combined
  query) rather than de novo candidate ranking
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3318
tools:
- name: MetFrag
  role: Executes the candidate retrieval from PubChem/ChemSpider/KEGG web services
    or local databases before in silico fragmentation
  repo: http://c-ruttkies.github.io/MetFrag/
- name: metfRag (MetFragR)
  role: R package wrapper for running MetFrag candidate retrieval and scoring from
    R
  repo: https://github.com/ipb-halle/MetFragRelaunched
- name: OpenBabel
  role: Converts suspect-list SMILES (e.g. STOFF-IDENT) to InChIKeys for candidate
    restriction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:b7af17a93e03be07e19a0a9deb4a503e8166fd78bc191237a8dc109c64c49faf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag
  - MetFrag2.2
  - MetFragCommandLine
  - MetFragLib
  - OpenBabel
  - metfRag (MetFragR)
  - xcms
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# candidate-structure-retrieval-from-pubchem-chemspider

## Summary

Retrieves candidate compound structures from the PubChem and ChemSpider databases (or a user-supplied SDF) for a given precursor mass/formula, producing the candidate lists that MetFrag2.2 scores against HR-MS/MS spectra. Database choice and size directly control ranking difficulty, so retrieval must be configured consistently with the scoring step.

## When to use

Use when tandem-MS spectra need candidate structures scored by in silico fragmentation and no candidate list exists yet; trigger on a precursor neutral mass or molecular formula plus a chosen compound database (PubChem, ChemSpider).

## When NOT to use

- Candidate lists are already provided (e.g. the candidate files shipped in the EA/UF/EQEx/EQExPlus/HILL dataset ZIP archives) — skip retrieval and use them directly
- The target space is a small, specific database (KEGG, HMDB, STOFF-IDENT, NORMAN suspects) rather than the large general databases — the article notes smaller, specific databases greatly improve top-rank chances
- The identification goal is a direct spectral library match (e.g. MassBank combined query) rather than de novo candidate ranking

## Inputs

- Precursor neutral monoisotopic mass or molecular formula per spectrum
- Choice of compound database (PubChem, ChemSpider, KEGG) as set in the MetFrag parameter file
- ChemSpider security token (for ChemSpider web service queries)
- Optional suspect list (InChIKeys or SMILES converted via OpenBabel, e.g. from STOFF-IDENT)
- Optional user-supplied structure data file (SDF)

## Outputs

- Per-spectrum candidate structure lists (candidate files with structures, InChIKeys, database identifiers)
- Candidate lists consumed by MetFrag2.2 for in silico fragmentation and ranking

## How to apply

Configure the database source in the MetFrag parameter file (LocalMetFragDatabase / web-service choice), supplying a ChemSpiderToken in settings.properties when querying ChemSpider web services. Retrieve candidates by precursor neutral monoisotopic mass or molecular formula from PubChem or ChemSpider; the reference study retrieved candidate structures from PubChem and ChemSpider in June 2015, so record the retrieval date because database growth changes ranking difficulty over time. Keep candidate lists aligned with the dataset's parameter, spectral, result and ranking files (as in the EA, UF, EQEx, EQExPlus and HILL dataset ZIP archives), and optionally restrict candidates to a suspect list supplied as InChIKeys (e.g. STOFF-IDENT SMILES converted with OpenBabel). Retrieved candidates are then fragmented in silico by MetFrag2.2's bond-dissociation approach and ranked against the measured product ions.

## Related tools

- **MetFrag** (Executes the candidate retrieval from PubChem/ChemSpider/KEGG web services or local databases before in silico fragmentation) — http://c-ruttkies.github.io/MetFrag/
- **metfRag (MetFragR)** (R package wrapper for running MetFrag candidate retrieval and scoring from R) — https://github.com/ipb-halle/MetFragRelaunched
- **OpenBabel** (Converts suspect-list SMILES (e.g. STOFF-IDENT) to InChIKeys for candidate restriction)

## Examples

```
library(devtools); install_github("ipb-halle/MetFragR/metfRag"); library(metfRag)
```

## Evaluation signals

- Each spectrum's retrieved candidate list contains the known correct compound before scoring (verifiable against known identities in the validation datasets)
- Retrieval date is documented (reference: PubChem and ChemSpider retrieval in June 2015) so ranking results are reproducible
- Candidate lists sum consistently across the 824 merged spectra of the validation datasets and match the counts in the dataset's parameter/result files
- Suspect-list restriction reduces candidate counts to only InChIKeys present in the supplied list (e.g. STOFF-IDENT-derived set)

## Limitations

- Database growth (PubChem had doubled since 2009) makes candidate ranking progressively harder, so retrieval date affects reproducibility of Top 1/Top 3/Top 5 percentages
- MetFrag2.2 with HR-MS/MS information only achieved only 30 Top 1 ranks with PubChem, versus 336 (71%) with retention and reference information — retrieval alone does not guarantee ranking success
- Candidate files for the EA, UF, EQEx, EQExPlus and HILL datasets are only available as external ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/, not regenerated on demand

## Evidence

- [intro] Candidate retrieval workflow: [evidence span withheld — anchor status: not_found]
- [results] Retrieval date matters: "candidate structures were retrieved from the compound databases PubChem and ChemSpider in June, 2015"
- [readme] ChemSpider token configuration: "## define chemspider token to query ChemSpider database
ChemSpiderToken = ..."
- [abstract] Web-service scope: "retrieval of reference, data source and patent information via ChemSpider and PubChem web services"

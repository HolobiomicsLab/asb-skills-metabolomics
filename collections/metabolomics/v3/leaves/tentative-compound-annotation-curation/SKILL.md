---
name: tentative-compound-annotation-curation
description: Use when a GC-MS library-hit table (columns Component.RT, Component.Area,
  Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) needs per-compound public-database
  annotation and curation before peak-area extraction.
when_to_use_negative:
- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms depend on published
  spectra in PubChem and do not yet apply to those acquisition modes
- Data were acquired on instruments other than GC-MS (e.g. Q-TOF, Orbitrap, TimsTOF
  HT, Astral or LC-MS) — uafR has not been tested in those contexts
- Compounds already carry resolved PubChem CIDs or curated annotations — the CID-resolution
  and CACTUS fallback steps would be redundant
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: uafR
  role: R package providing spreadOut(), mzExacto(decontaminate = TRUE), categorate(detail
    = "research"), exactoThese(), and validateCategorateResult() to expand, annotate,
    filter, and export tentative compound annotations
  repo: github.com/castratton/uafR
- name: webchem
  role: Supplies get_cid() to convert tentative compound names into PubChem CIDs,
    exact monoisotopic masses, and preferred names; NCI CACTUS resolver used for canonical
    SMILES fallback
- name: R
  role: Runtime environment (R >= 4.2) with BiocManager dependencies ChemmineR and
    fmcsR for the cheminformatics steps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:3b50d5b92f3e19363f91348e8014216902f86c7d0d8c08a684e3739877830f79
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - uafR
  - webchem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tentative-compound-annotation-curation

## Summary

Enriches tentative GC-MS compound identities from an Agilent Unknowns Analysis hit table by retrieving PubChem CIDs, exact monoisotopic masses, and preferred names via webchem's get_cid(), falling back to NCI CACTUS canonical SMILES for compounds absent from PubChem, then curating hits with match-factor and structural-match thresholds.

## When to use

Use when a GC-MS library-hit table (columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) needs per-compound public-database annotation and curation before peak-area extraction.

## When NOT to use

- Input is MS2, MS3, or ion mobility + MS2 data — uafR's algorithms depend on published spectra in PubChem and do not yet apply to those acquisition modes
- Data were acquired on instruments other than GC-MS (e.g. Q-TOF, Orbitrap, TimsTOF HT, Astral or LC-MS) — uafR has not been tested in those contexts
- Compounds already carry resolved PubChem CIDs or curated annotations — the CID-resolution and CACTUS fallback steps would be redundant

## Inputs

- Agilent Unknowns Analysis library-hit CSV (e.g. gcms_hit_table.csv) with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- Character vector of target compound names (user-specified or Match.Factor > 80 subset)
- Optional chemical library data frame (e.g. library_data, wide format) for categorate() annotation

## Outputs

- spread database (list-structured object serialized as spread_database.rds) with unique IDs per datapoint
- mzExacto() result: compound identity, optimal retention time, exact mass, best match factor, aggregated component area across samples
- categorate(detail = "research") result: reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch tables plus PubChemProperties, ChemicalTraits, KEGGPathways, SourceCoverage
- Exported CSV/Excel bundle with manifest via exportCategorateWorkbook()

## How to apply

Load the vendor CSV without renaming columns — uafR requires 'Component.RT', 'Component.Area', 'Base.Peak.MZ', 'File.Name', 'Compound.Name', and 'Match.Factor' — and reject duplicate peak keys before expanding the table with spreadOut() into one record per datapoint with unique identifiers. For each tentative Compound.Name, query PubChem with webchem's get_cid() to retrieve the CID, exact monoisotopic mass, and preferred compound name; for compounds missing a CID, resolve a canonical SMILES via the NCI CACTUS resolver and simulate mass and structure data from it. Apply a match-factor threshold to subset candidates (the README example uses Compound.Name[Match.Factor > 80]; the article also describes filtering on Tanimoto similarity > 0.95 as a strong or > 0.85 as a moderate structural match). Run mzExacto(spread, query_compounds, decontaminate = TRUE) to remove chemicals lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES, returning identities, optimal retention time, exact mass, best match factor, and aggregated component area across samples. For deeper annotation, call categorate(detail = "research") to append normalized PubChem and KEGG tables (PubChemProperties, KEGGPathways, ChemicalTraits, SourceCoverage, etc.) on top of the eight standard tables, and validate the result with validateCategorateResult() before export.

## Related tools

- **uafR** (R package providing spreadOut(), mzExacto(decontaminate = TRUE), categorate(detail = "research"), exactoThese(), and validateCategorateResult() to expand, annotate, filter, and export tentative compound annotations) — github.com/castratton/uafR
- **webchem** (Supplies get_cid() to convert tentative compound names into PubChem CIDs, exact monoisotopic masses, and preferred names; NCI CACTUS resolver used for canonical SMILES fallback)
- **R** (Runtime environment (R >= 4.2) with BiocManager dependencies ChemmineR and fmcsR for the cheminformatics steps)

## Examples

```
spread <- spreadOut(raw_gcms); targets <- unique(raw_gcms$Compound.Name[raw_gcms$Match.Factor >= 80]); extracted <- mzExacto(spread, targets, decontaminate = TRUE)
```

## Evaluation signals

- All six required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) are present in the input table before spreadOut() runs
- Every tentative compound name resolves to a PubChem CID or, failing that, a canonical SMILES from NCI CACTUS; decontaminate = TRUE removes any chemical missing both
- mzExacto() output contains the five expected fields: compound identity, optimal retention time, exact mass, best match factor, and aggregated component area
- Standard categorate() returns exactly eight tables (reactives, LOTUS, KEGG, FEMA, FDA_SPL, FMCS, FunctionalGroups, BestChemMatch); detail = "research" preserves these and appends PubChem/KEGG tables
- validateCategorateResult() audit (Summary, TableQuality, SourceDiagnostics, Issues) reports no blocking issues before export

## Limitations

- Compounds absent from PubChem require the NCI CACTUS fallback; chemicals with no strong cross-sample match, no PubChem entry, and no canonical SMILES are removed by decontamination
- MS2/MS3 and ion mobility + MS2 functionality is deferred until spectra are published for most chemicals in PubChem's database
- Researchers should still validate that compound areas identified by the algorithm make chemical and/or biological sense in their study system
- uafR requires R >= 4.2 and Bioconductor packages ChemmineR and fmcsR; Windows users may need Rtools and macOS users the Xcode Command Line Tools
- No changelog or DOI was available in the GitHub source record for the castratton/uafR repository

## Evidence

- [intro] Public-database resolution of tentative names: "communicates with public chemistry utilities (including PubChem and the National Cancer Institute) to sort and process the aggregated set of all tentatively identified molecules"
- [intro] Decontamination filter criteria: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [intro] Structural match thresholds: "whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match"
- [readme] Required input schema: [evidence span withheld — anchor status: external_doc]
- [readme] mzExacto output contract: [evidence span withheld — anchor status: not_found]

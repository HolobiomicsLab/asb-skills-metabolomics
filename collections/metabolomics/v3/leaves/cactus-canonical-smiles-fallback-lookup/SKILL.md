---
name: cactus-canonical-smiles-fallback-lookup
description: Use when a GC-MS hit-table compound (e.g. from Agilent Unknowns Analysis)
  has no PubChem CID after webchem get_cid() lookup, and a canonical SMILES plus simulated
  mass/structure data is needed to keep the compound in the spread database.
when_to_use_negative:
- The compound already resolves to a PubChem CID with exact monoisotopic mass — the
  CACTUS fallback is only for CID-less compounds
- Data are MS2, MS3, or ion mobility + MS2 — uafR's published-spectrum-dependent algorithms
  (and hence this annotation step) do not yet apply
- Input is not a vendor peak table with the strict uafR column schema (e.g. already
  a processed feature table)
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: uafR
  role: R package whose spreadOut() function implements the annotation pipeline and
    invokes the PubChem/CACTUS fallback for each tentative compound
  repo: github.com/castratton/uafR
- name: webchem
  role: Provides get_cid() to convert compound names to PubChem CIDs; compounds failing
    this step are routed to the CACTUS fallback
- name: NCI CACTUS
  role: Public chemical resolver used to generate a canonical SMILES for compounds
    without a PubChem CID
- name: PubChem
  role: Primary source for CID, exact monoisotopic mass, and preferred compound name
    before the fallback is triggered
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:045181b17974a94fe7205a47431af9af5f422e3b85636019bc8c7ce3a24b2fb9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NCI CACTUS
  - PubChem
  - R
  - uafR
  - webchem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# CACTUS canonical SMILES fallback lookup

## Summary

A fallback resolution step in the uafR spreadOut() pipeline: compounds from the Agilent Unknowns Analysis hit table that lack a PubChem CID are sent to the NCI CACTUS resolver to obtain a canonical SMILES, which is then used to simulate mass and structure data so every tentative compound can be represented in the spread database.

## When to use

Use when a GC-MS hit-table compound (e.g. from Agilent Unknowns Analysis) has no PubChem CID after webchem get_cid() lookup, and a canonical SMILES plus simulated mass/structure data is needed to keep the compound in the spread database.

## When NOT to use

- The compound already resolves to a PubChem CID with exact monoisotopic mass — the CACTUS fallback is only for CID-less compounds
- Data are MS2, MS3, or ion mobility + MS2 — uafR's published-spectrum-dependent algorithms (and hence this annotation step) do not yet apply
- Input is not a vendor peak table with the strict uafR column schema (e.g. already a processed feature table)

## Inputs

- Expanded per-datapoint GC-MS records from spreadOut() with tentative Compound.Name, Component.RT, Component.Area, Base.Peak.MZ, File.Name, and Match.Factor
- Per-compound PubChem lookup results from webchem::get_cid(), identifying which names lack a CID

## Outputs

- Canonical SMILES strings for compounds absent from PubChem
- Simulated exact mass and structure data derived from the canonical SMILES
- List-structured spread database (serialized as spread_database.rds) containing all per-compound annotations

## How to apply

Within spreadOut(), first query PubChem with webchem's get_cid() to retrieve the CID, exact monoisotopic mass, and preferred name for each Compound.Name in the expanded peak table (which must contain Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor). For any compound with no CID, call the NCI CACTUS resolver to generate a canonical SMILES; this SMILES is then used to simulate mass and structure data so the compound is not lost. Store each compound's resolved CID or SMILES, mass, name, and metadata as a list entry in the spread database and serialize it (e.g. spread_database.rds). Note the downstream rationale: the mzExacto(decontaminate = TRUE) step removes chemicals that were unable to be found in PubChem AND unable to have a canonical SMILES generated, so the CACTUS fallback directly protects borderline identifications from being dropped.

## Related tools

- **uafR** (R package whose spreadOut() function implements the annotation pipeline and invokes the PubChem/CACTUS fallback for each tentative compound) — github.com/castratton/uafR
- **webchem** (Provides get_cid() to convert compound names to PubChem CIDs; compounds failing this step are routed to the CACTUS fallback)
- **NCI CACTUS** (Public chemical resolver used to generate a canonical SMILES for compounds without a PubChem CID)
- **PubChem** (Primary source for CID, exact monoisotopic mass, and preferred compound name before the fallback is triggered)

## Examples

```
input_spread = spreadOut(input_dat)
```

## Evaluation signals

- Every Compound.Name lacking a PubChem CID in the spread database carries a non-empty canonical SMILES obtained from CACTUS
- The spread database contains one record per datapoint with unique identifiers, retaining component area, match factor, base-peak m/z, retention time, and sample ID
- Compounds with neither a PubChem entry nor a generated canonical SMILES are the only ones removed by decontaminate = TRUE
- Simulated masses derived from CACTUS SMILES are within the expected monoisotopic-mass range for the tentative compound's formula
- Serialized spread_database.rds reloads as the documented list structure without missing annotation slots

## Limitations

- Mass and structure data for CID-less compounds are simulated from the CACTUS canonical SMILES rather than taken from a curated database entry, so they carry lower confidence
- Chemicals that cannot be found in PubChem and cannot have a canonical SMILES generated are removed by decontaminate = TRUE and are unrecoverable
- Manual validation of compound areas and identities is still required from researchers
- uafR is untested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) and LC-MS data

## Evidence

- [intro] Stream published chemical information from PubChem and NCI CACTUS: "communicates with public chemistry utilities (including PubChem and the National Cancer Institute) to sort and process the aggregated set of all tentatively identified molecules"
- [intro] Decontamination removes chemicals without public-database entry or canonical SMILES: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [readme] spreadOut() strict input column requirements: [evidence span withheld — anchor status: external_doc]

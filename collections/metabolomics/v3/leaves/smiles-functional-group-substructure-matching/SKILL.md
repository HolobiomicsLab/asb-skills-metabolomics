---
name: smiles-functional-group-substructure-matching
description: Use when annotating MALDI-MSI m/z features from a derivatizing matrix
  (e.g. FMP-10) and you must filter a structure database such as HMDB or LIPID MAPS
  down to compounds containing the derivatization agent's target functional groups,
  including multi-site compounds.
when_to_use_negative:
- The matrix is conventional (non-derivatizing) so the expected ion is [M+H]+ or [M-H]-
  and no functional-group prefiltering is needed.
- Input is already an annotated peak/feature table rather than raw structures — this
  skill operates on SMILES/SDF-level chemical structure, not on m/z features.
- You need MS2 spectral similarity rather than MS1 candidate filtering; Met-ID's MS2
  database comparison is a separate step.
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_3172
tools:
- name: RDKit
  role: Parses and sanitizes HMDB/LIPID MAPS SMILES into Mol objects and performs
    SMARTS substructure matching to count derivatizable sites
  repo: https://www.rdkit.org/
- name: Met-ID
  role: Reference implementation whose derivatized-target subsetting (HMDB(All) default,
    FMP-10 matrix) this skill reproduces; results feed its MS1 identify() mass matching
  repo: https://github.com/pbjarterot/Met-ID
- name: HMDB
  role: Openly available target metabolite structure database supplying the SMILES
    that are filtered by functional-group matching
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:09aef443ded1ac4cc6b0268deb037e9768bf5a2fd5512ec56219a9818e15ca07
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMDB
  - Met-ID
  - RDKit
  - SQLite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# smiles-functional-group-substructure-matching

## Summary

Enumerates derivatization-reactive functional groups (e.g. phenolic hydroxyls, primary/secondary amines for the FMP-10 reactive MALDI matrix) in structure-database molecules by parsing HMDB/LIPID MAPS SMILES with RDKit and applying SMARTS substructure matching, mirroring how Met-ID generates its derivatized-target subset for MS1 annotation.

## When to use

Use when annotating MALDI-MSI m/z features from a derivatizing matrix (e.g. FMP-10) and you must filter a structure database such as HMDB or LIPID MAPS down to compounds containing the derivatization agent's target functional groups, including multi-site compounds.

## When NOT to use

- The matrix is conventional (non-derivatizing) so the expected ion is [M+H]+ or [M-H]- and no functional-group prefiltering is needed.
- Input is already an annotated peak/feature table rather than raw structures — this skill operates on SMILES/SDF-level chemical structure, not on m/z features.
- You need MS2 spectral similarity rather than MS1 candidate filtering; Met-ID's MS2 database comparison is a separate step.

## Inputs

- HMDB (All) structure records as SMILES or SDF
- LIPID MAPS structure records as SMILES or SDF
- SMARTS patterns for the derivatization agent's target functional groups (e.g. FMP-10 reactive groups)

## Outputs

- derivatization_site_counts.csv with columns accession, compound name, derivatization-site count
- Filtered derivatized-target subset retaining only compounds with ≥1 matching functional group

## How to apply

Download the public HMDB/LIPID MAPS structure records (SMILES/SDF) that Met-ID parses into its SQLite databases. Sanitize each structure into an RDKit Mol object, neutralizing salts and skipping unparseable entries rather than failing the run. Apply SMARTS substructure matching that mirrors Met-ID's functional-group definitions — phenolic hydroxyls and primary/secondary amines for FMP-10 — to count derivatizable sites per compound. Exclude zero-match compounds (e.g. the cyanoacrylate enbucrilate) from the derivatized-target subset but retain their site counts (0) for validation; keep multi-site compounds since Met-ID 'is able to identify multiple derivatization sites on the same molecule'. Emit a table of accession, compound name, and derivatization-site count, then spot-check against a known four-isomer set (enbucrilate = 0, p-octopamine = 2, norfenefrine = 2, dopamine = 3).

## Related tools

- **RDKit** (Parses and sanitizes HMDB/LIPID MAPS SMILES into Mol objects and performs SMARTS substructure matching to count derivatizable sites) — https://www.rdkit.org/
- **Met-ID** (Reference implementation whose derivatized-target subsetting (HMDB(All) default, FMP-10 matrix) this skill reproduces; results feed its MS1 identify() mass matching) — https://github.com/pbjarterot/Met-ID
- **HMDB** (Openly available target metabolite structure database supplying the SMILES that are filtered by functional-group matching)

## Examples

```
from rdkit import Chem; mol = Chem.MolFromSmiles(smi); n = len(mol.GetSubstructMatches(Chem.MolFromSmarts('[OX2H][c],[NX3;H2,H3;!$(N[*]=O)]')))
```

## Evaluation signals

- Four-isomer validation: enbucrilate must score 0 derivatizable sites, p-octopamine 2, norfenefrine 2, and dopamine 3.
- Every compound with a zero site count must be excluded from the derivatized-target subset while remaining present in derivatization_site_counts.csv.
- Unparseable or salt-containing entries are skipped/neutralized without aborting the batch, and the accession-to-site-count table covers all parsed compounds.
- Multi-site compounds (e.g. dopamine with three sites) survive the filter, matching Met-ID's stated multi-site capability.
- Functional-group SMARTS reproduce the FMP-10 reactive groups used by Met-ID's defaults (HMDB(All) + FMP-10 matrix).

## Limitations

- The FMP-10 derivatizing matrix was developed in-house, so its reactive-group definitions may need re-deriving from the literature for other users' workflows.
- rdkit installation is described as slightly more difficult than an average PyPI project (often via anaconda) and no pinned dependency versions are stated.
- Reinstalling newer versions of Met-ID does not update bundled database files; the com.farmbio.metid folder must be removed manually before reinstalling.
- Adding functional groups currently has known issues on macOS.

## Evidence

- [abstract] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [methods] The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix.: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [discussion] Met-ID is also dependent on rdkit, a C++ framework for computational chemistry: [evidence span withheld — anchor status: not_found]
- [readme] Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode.: "Met-ID has a particular focus on derivatizing matrices leading to other ions than the common [M+H]+ in positive mode and [M-H]- in negative mode."
- [discussion] contains files parsing the HMDB and LIPID MAPS into SQLite databases that Met-ID can read: [evidence span withheld — anchor status: not_found]

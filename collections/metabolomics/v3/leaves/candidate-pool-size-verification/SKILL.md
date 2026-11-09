---
name: candidate-pool-size-verification
description: Use when reproducing or validating the FMP-10 candidate-pool reduction
  over HMDB — i.e., before metabolite identification in MALDI-MSI, to confirm the
  derivatization-filter subset counts (26,202 all vs 4,951 endogenous) against reported
  values.
when_to_use_negative:
- The goal is metabolite identification/annotation of m/z features from MALDI-MSI
  data rather than verifying the candidate-pool sizes upstream of identification
- The data were acquired with a conventional (non-derivatizing) matrix whose expected
  adducts are [M+H]+ / [M-H]- and no FMP-10 functional-group filter applies
- The metabolite database is a custom or user-supplied target list rather than the
  Met-ID-parsed HMDB (or LIPID MAPS) SQLite database, so the 26,202 / 4,951 reference
  counts are not meaningful
edam_operation: http://edamontology.org/operation_3695
edam_topics: []
tools:
- name: Met-ID
  role: Provides MetIDServer (context-manager local server), the client's list_metabolites()
    loader for the parsed HMDB SQLite database, and the FMP-10 matrix functional-group
    filter used to count derivatizable candidates in the full (typ='All') and endogenous-only
    subsets
  repo: github.com/pbjarterot/Met-ID
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:f30f970fec33e0d89a49f4b0fe1720ea1c75fab716beaf38bbd8d6a788e46242
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Met-ID
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# candidate-pool-size-verification

## Summary

Verifies the derivatizable-candidate pool sizes that Met-ID generates when applying the FMP-10 derivatization functional-group filter to the parsed HMDB SQLite database, confirming 26,202 candidates for all HMDB metabolites and 4,951 for the endogenous-only subset.

## When to use

Use when reproducing or validating the FMP-10 candidate-pool reduction over HMDB — i.e., before metabolite identification in MALDI-MSI, to confirm the derivatization-filter subset counts (26,202 all vs 4,951 endogenous) against reported values.

## When NOT to use

- The goal is metabolite identification/annotation of m/z features from MALDI-MSI data rather than verifying the candidate-pool sizes upstream of identification
- The data were acquired with a conventional (non-derivatizing) matrix whose expected adducts are [M+H]+ / [M-H]- and no FMP-10 functional-group filter applies
- The metabolite database is a custom or user-supplied target list rather than the Met-ID-parsed HMDB (or LIPID MAPS) SQLite database, so the 26,202 / 4,951 reference counts are not meaningful

## Inputs

- Parsed HMDB SQLite database readable by Met-ID (all metabolites, including Exogenous/Endogenous/Unspecified origin annotations)
- FMP-10 derivatizing-matrix functional-group filter definition (default in Met-ID: HMDB(All) + FMP-10 matrix)

## Outputs

- Candidate-pool size for the full HMDB database under the FMP-10 filter (expected: 26,202 metabolites)
- Candidate-pool size for the endogenous-only HMDB subset under the FMP-10 filter (expected: 4,951 metabolites)
- Tabulated comparison of both pool sizes, e.g., fmp10_candidate_pools.csv

## How to apply

Start a local Met-ID server by importing MetIDServer and using it as a context manager (default free port, one-hour idle shutdown). Load the parsed HMDB SQLite database through the client's list_metabolites() convenience method — Met-ID ships files that parse HMDB (and LIPID MAPS) into SQLite databases it can read. Apply the FMP-10 matrix's derivatization-specific functional-group filter to the full database with typ='All' so Exogenous, Endogenous, and Unspecified origin records are all retained, yielding 26,202 candidate targets. Re-apply the identical FMP-10 functional-group filter restricted to the Endogenous subset only, yielding 4,951 candidates. The reduction reflects Met-ID's core strategy of using structural information for derivatizing matrices to generate a subset of targets containing only functional groups specific to the derivatization agent. Tabulate both counts (e.g., to fmp10_candidate_pools.csv) and accept the run only if both numbers match the article's reported values exactly.

## Related tools

- **Met-ID** (Provides MetIDServer (context-manager local server), the client's list_metabolites() loader for the parsed HMDB SQLite database, and the FMP-10 matrix functional-group filter used to count derivatizable candidates in the full (typ='All') and endogenous-only subsets) — github.com/pbjarterot/Met-ID

## Examples

```
from metid import MetIDServer; with MetIDServer() as s: mets = s.client.list_metabolites()  # parsed HMDB SQLite; then apply the FMP-10 filter with typ='All' and again on the Endogenous subset
```

## Evaluation signals

- Full-HMDB FMP-10-filtered pool must equal exactly 26,202 candidate targets
- Endogenous-only FMP-10-filtered pool must equal exactly 4,951 candidate targets
- The full-database run must use typ='All' so Exogenous, Endogenous, and Unspecified records are all included before counting; an accidental origin restriction inflates or deflates the count
- Both counts are written to the tabulated output (fmp10_candidate_pools.csv) and the endogenous/all-HMDB ratio is consistent across repeated runs with a fresh MetIDServer session
- list_metabolites() on the parsed HMDB SQLite database returns records before filtering, confirming the database files loaded are the intended bundled HMDB files

## Limitations

- Reinstalling newer versions of Met-ID does not update the bundled database files; the com.farmbio.metid application-support folder must be removed manually and Met-ID reinstalled, or stale database files will change the pool counts
- HMDB metabolites with unspecified origin fall outside the endogenous-only subset, so the endogenous pool (4,951) is strictly smaller than the full pool (26,202) and comparisons between them are only meaningful under the typ='All' convention
- FMP-10 was developed in-house by the authors and features heavily in Met-ID's defaults; pool sizes are specific to the FMP-10 functional-group filter and do not transfer to other derivatizing matrices without local configuration

## Evidence

- [abstract] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent."
- [methods] The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix.: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [methods] Import ``MetIDServer`` and use it as a context manager.: "Import ``MetIDServer`` and use it as a context manager."
- [discussion] contains files parsing the HMDB and LIPID MAPS into SQLite databases that Met-ID can read: [evidence span withheld — anchor status: not_found]
- [readme] reinstalling newer versions of Met-ID does not update the database files it comes bundled with and as such the database files will have to removed manually: [evidence span withheld — anchor status: external_doc]

---
name: endogenous-metabolite-subsetting
description: Use when running Met-ID Met-ID/FMP-10 derivatization searches against
  a parsed HMDB SQLite database and the biological sample is expected to contain only
  endogenous metabolites, so exogenous and unspecified HMDB records should be excluded
  from the candidate pool.
when_to_use_negative:
- When the sample may contain exogenous compounds (e.g. drug or xenobiotic studies)
  — use the full-HMDB typ='All' pool instead
- When using a non-derivatizing conventional matrix whose adducts (e.g. [M+H]+, [M-H]-)
  do not require the FMP-10 functional-group filter
- When the annotation database is LIPID MAPS rather than HMDB, since the Endogenous/Exogenous
  typ classification is an HMDB field
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: Met-ID
  role: Provides the MetIDServer local server, the parsed HMDB SQLite database, and
    the FMP-10 functional-group filter used to restrict the candidate pool to Endogenous
    metabolites
  repo: https://github.com/pbjarterot/Met-ID
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:4372f4f620db2acc41826574012dc46250a05804dc0db751121d3b4671db0316
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - Met-ID
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# endogenous-metabolite-subsetting

## Summary

Restrict a parsed HMDB metabolite database to its Endogenous subset before applying the FMP-10 derivatization functional-group filter, shrinking the Met-ID candidate pool from 26,202 (all HMDB) to 4,951 endogenous metabolites to cut false-positive mass matches in MALDI-MSI.

## When to use

Use when running Met-ID Met-ID/FMP-10 derivatization searches against a parsed HMDB SQLite database and the biological sample is expected to contain only endogenous metabolites, so exogenous and unspecified HMDB records should be excluded from the candidate pool.

## When NOT to use

- When the sample may contain exogenous compounds (e.g. drug or xenobiotic studies) — use the full-HMDB typ='All' pool instead
- When using a non-derivatizing conventional matrix whose adducts (e.g. [M+H]+, [M-H]-) do not require the FMP-10 functional-group filter
- When the annotation database is LIPID MAPS rather than HMDB, since the Endogenous/Exogenous typ classification is an HMDB field

## Inputs

- Parsed HMDB SQLite database readable by Met-ID (via MetIDServer client list_metabolites())
- Met-ID FMP-10 derivatization functional-group filter definition

## Outputs

- Endogenous-only candidate pool of metabolites (4,951 targets) suitable for FMP-10 derivatization Met-ID annotation
- Comparison table of candidate-pool sizes, e.g. fmp10_candidate_pools.csv, for full HMDB (26,202) vs endogenous (4,951)

## How to apply

Load the parsed HMDB SQLite database through the Met-ID client's list_metabolites() convenience method after starting the local MetIDServer as a context manager. Re-run the FMP-10 derivatization-matrix functional-group filter with the typ parameter set to restrict records to the Endogenous subset only, instead of typ='All' which includes Exogenous and Unspecified records. Tabulate the filtered candidate-pool size and compare against the article's benchmark: the endogenous pool must contain 4,951 candidate targets versus 26,202 for the full HMDB. The rationale is that derivatization targets functional groups specific to the derivatizing matrix, and filtering to biologically plausible (endogenous) metabolites removes candidates that cannot occur in tissue, sharpening subsequent MS1 mass matching and MS2 spectral comparison.

## Related tools

- **Met-ID** (Provides the MetIDServer local server, the parsed HMDB SQLite database, and the FMP-10 functional-group filter used to restrict the candidate pool to Endogenous metabolites) — https://github.com/pbjarterot/Met-ID

## Examples

```
with MetIDServer() as client: mets = client.list_metabolites(db='HMDB', typ='Endogenous'); pool = apply_fmp10_functional_group_filter(mets); assert len(pool) == 4951
```

## Evaluation signals

- The endogenous-only FMP-10 filtered pool must contain exactly 4,951 candidate metabolites
- The full-HMDB (typ='All') FMP-10 filtered pool must contain 26,202 candidate metabolites
- The endogenous pool size must be strictly smaller than the full-HMDB pool size (4,951 < 26,202)
- The output table (e.g. fmp10_candidate_pools.csv) contains one row per subset with the filtered candidate-pool counts

## Limitations

- Counts depend on the bundled HMDB database version; reinstalling newer Met-ID versions does not update bundled database files, which must be removed manually (e.g. the com.farmbio.metid folder) before reinstalling
- Restricting to Endogenous records discards HMDB entries classified as Unspecified, which may exclude metabolites of uncertain origin
- The subset only limits the derivatization candidate pool; it does not by itself resolve false positives, which require recalibration and MS2 spectral comparison

## Evidence

- [methods] The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix.: "The defaults select ``HMDB(All)`` and the ``FMP-10`` matrix."
- [abstract] It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups specific to the derivatization agent.: "It utilizes structural information for derivatizing matrices to generate a subset of targets that contain only functional groups"
- [discussion] contains files parsing the HMDB and LIPID MAPS into SQLite databases that Met-ID can read: [evidence span withheld — anchor status: not_found]
- [methods] Import ``MetIDServer`` and use it as a context manager.: "Import ``MetIDServer`` and use it as a context manager."
- [readme] reinstalling newer versions of Met-ID does not update the database files it comes bundled with: [evidence span withheld — anchor status: external_doc]

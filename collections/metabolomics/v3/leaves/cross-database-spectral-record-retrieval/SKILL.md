---
name: cross-database-spectral-record-retrieval
description: Use when the same physical mass spectrum must be reconciled across MassBank,
  MoNA, GNPS, HMDB, MetaboLights, or mzCloud entries whose accession IDs differ, and
  a database-independent identifier is needed to confirm they refer to the same spectrum.
when_to_use_negative:
- When the goal is to distinguish two spectra that each contain only a single ion
  of the same mass — the SPLASH was observed to collide for such spectra, so the identifier
  cannot discriminate them.
- When the input is not a mass spectrum but a chemical structure identifier — the
  article positions InChIKey as the analogous identifier for structures, so InChI-based
  tools are the correct choice there.
- When spectra come from libraries like NIST that cannot be publicly released and
  are therefore absent from the MoNA-hosted SPLASH dataset — no hash can be computed
  without access to the underlying peak list.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3360
tools:
- name: spectra-hash (Java reference implementation)
  role: Reference SPLASH implementation used to compute the four-block spectral hash
    for each cross-database record
  repo: https://github.com/berlinguyinca/spectra-hash
- name: MassBank
  role: Source spectral library whose record PR100026 is the canonical example cross-referenced
    against other databases
- name: MoNA
  role: Public hosting repository where 'all but the NIST spectra are available on
    MoNA' for cross-database retrieval
- name: validation tool (validation-1.8.jar)
  role: Verifies that SPLASHes generated from different records/databases are consistent
    with the reference implementation
  repo: http://gose.fiehnlab.ucdavis.edu:55000/content/groups/public/edu/ucdavis/fiehnlab/splash/validation/1.8/validation-1.8.jar
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nbt.3689
    title: SPLASH, a hashed identifier for mass spectra
  source_content_hash: sha256:84810a17330481563b8c3bffd031ccc548ed9cde07c02946b451ece397e26c05
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/berlinguyinca/spectra-hash
  tools:
  - Java
  - MassBank
  - MoNA
  - Python
  - spectra-hash (Java reference implementation)
  - validation tool (validation-1.8.jar)
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# cross-database-spectral-record-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Resolves identical mass spectra recorded under different database-specific identifiers (e.g. PR100026 in MassBank/MoNA, 5464 in HMDB, CCMSLIB00000222858 in GNPS) by applying the SPLASH spectral hash to each record and matching on the resulting four-block identifier rather than on database accession numbers.

## When to use

Use when the same physical mass spectrum must be reconciled across MassBank, MoNA, GNPS, HMDB, MetaboLights, or mzCloud entries whose accession IDs differ, and a database-independent identifier is needed to confirm they refer to the same spectrum.

## When NOT to use

- When the goal is to distinguish two spectra that each contain only a single ion of the same mass — the SPLASH was observed to collide for such spectra, so the identifier cannot discriminate them.
- When the input is not a mass spectrum but a chemical structure identifier — the article positions InChIKey as the analogous identifier for structures, so InChI-based tools are the correct choice there.
- When spectra come from libraries like NIST that cannot be publicly released and are therefore absent from the MoNA-hosted SPLASH dataset — no hash can be computed without access to the underlying peak list.

## Inputs

- Mass spectrum peak lists in single-line 'ion:intensity ion:intensity ...' string format (e.g. '138.0641:71.59 195.0815:261.7')
- Spectral records from MassBank (e.g. PR100026), HMDB (e.g. 5464), GNPS (e.g. CCMSLIB00000222858), MoNA, MetaboLights, mzCloud, or the spectra-hash GitHub repo datasets
- CSV files with columns for spectrum ID/origin, the mass spectrum string, and (optionally) a precomputed SPLASH, for use with the validation tool

## Outputs

- Full four-block SPLASH identifier per spectrum (e.g. splash10-0002-0900000000-b112e4e059e1ecf98c5f)
- Cross-database record mapping linking differing accession IDs (PR100026, 5464, CCMSLIB00000222858) to a shared SPLASH
- Validation-tool output confirming consistent SPLASH generation across candidate records

## How to apply

Parse each candidate spectrum's peak list into the single-line 'ion:intensity' form defined by SPLASH (e.g. '138.0641:71.59 195.0815:261.7' for the caffeine spectrum PR100026). Compute the SPLASH using the reference Java implementation via SplashFactory or SplashUtil with SpectraType.MS, which produces the identifier block (splash10), two canonical summary blocks (built through the canonicalization and summary-bin rules), and a fourth hash block derived with SHA-256. Cross-database resolution then reduces to exact string comparison of the full SPLASH, so that PR100026, 5464 and CCMSLIB00000222858 collapse to the single identifier splash10-0002-0900000000-b112e4e059e1ecf98c5f. For bulk matching, run the validation tool (java -jar validation-1.8.jar) with the -c, -s (spectrum column), -k (SPLASH column) and -o (spectrum origin/ID column) flags to ensure every candidate record in the cross-database set was hashed with the same reference implementation. Rationale: because the hash block has a theoretical collision probability of ~10^-31 for 10^9 spectra and identical SPLASHes were only observed for single-ion spectra of the same mass, an exact match is strong evidence the underlying spectra are the same compound measurement.

## Related tools

- **spectra-hash (Java reference implementation)** (Reference SPLASH implementation used to compute the four-block spectral hash for each cross-database record) — https://github.com/berlinguyinca/spectra-hash
- **MassBank** (Source spectral library whose record PR100026 is the canonical example cross-referenced against other databases)
- **MoNA** (Public hosting repository where 'all but the NIST spectra are available on MoNA' for cross-database retrieval)
- **validation tool (validation-1.8.jar)** (Verifies that SPLASHes generated from different records/databases are consistent with the reference implementation) — http://gose.fiehnlab.ucdavis.edu:55000/content/groups/public/edu/ucdavis/fiehnlab/splash/validation/1.8/validation-1.8.jar

## Examples

```
String splash = SplashUtil.splash("138.0641:71.59 195.0815:261.7", SpectraType.MS);
```

## Evaluation signals

- The computed SPLASH for the caffeine spectrum PR100026 must be byte-for-byte identical to splash10-0002-0900000000-b112e4e059e1ecf98c5f.
- Running the validation tool with -c -s <col> -t ms against the reference implementation must report zero mismatches between generated and expected SPLASH columns.
- Any two records sharing a SPLASH identifier should correspond to the same spectrum; conversely, differing SPLASHes must correspond to different peak lists.
- The identifier must consist of exactly four dash-separated blocks: splash10, two summary blocks, and a SHA-256-derived hash block.
- When reconciling PR100026 (MassBank/MoNA), 5464 (HMDB) and CCMSLIB00000222858 (GNPS), all three must resolve to one shared SPLASH, confirming they are the same caffeine spectrum.

## Limitations

- Identical SPLASHes were only observed for spectra containing a single ion of the same mass, so the identifier cannot discriminate between such spectra.
- Theoretical collision probability is approximately 10^-31 for a database of 10^9 spectra, but this is an estimate, not a proof of uniqueness.
- Not all spectra in the validation set are annotated with structures, so structure-level cross-checking may be unavailable for some records.
- NIST library spectra cannot be publicly released and are absent from the MoNA-hosted dataset, limiting cross-database coverage.

## Evidence

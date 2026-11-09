---
name: cross-ionization-mode-compound-set-intersection
description: Use when flow injection–Orbitrap MSn data have been acquired in both
  positive and negative ionization modes, per-mode feature lists (e.g. from mzmine)
  and cleaned metadata exist, and the goal is to quantify how many unique compounds
  are detected in positive-only, negative-only, and both modes.
when_to_use_negative:
- Input is already a feature table with ionization-mode labels and a compound identifier
  column — no join or set-intersection work is needed
- Data were acquired in a single ionization mode only — the cross-mode intersection
  is undefined and per-mode counts alone suffice
- Metadata have not passed the metadata cleanup flow and lack consistent column names
  (unique sample id, plate_id, well location) — the feature-to-compound join would
  be ambiguous
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzmine
  role: Processes the .mzML flow injection–Orbitrap acquisitions into per-ionization-mode
    feature lists that are joined against the cleaned metadata to assign unique compound
    identifiers
  repo: https://github.com/mzmine/mzmine
- name: ThermoRawFileParser
  role: Converts Thermo .raw files from the Orbitrap ID-X into open formats (mzML/MGF/Parquet)
    so mzmine can process both ionization-mode acquisitions
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: Prefect
  role: Runs the metadata_cleanup_prefect.py flow that standardizes metadata column
    names and minimum needed information, enabling a reliable feature-to-compound
    join
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: Orbitrap ID-X
  role: Instrument that acquires the flow injection MSn data whose per-mode compound
    detections feed the intersection statistics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:5603dd2155bb0207989109dcadeee9cf7848baf5b6704c1da3af3288d2a14fcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - Orbitrap ID-X
  - Prefect
  - ThermoRawFileParser
  - mzmine
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - direct-infusion-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-ionization-mode-compound-set-intersection

## Summary

Compute the distribution of unique compound detections across positive-only, negative-only, and both ionization modes by joining mzmine feature lists against cleaned flow injection–Orbitrap MSn metadata and intersecting the per-mode compound identifier sets; this yields the 30,008-unique-compound coverage breakdown central to the MSnLib resource.

## When to use

Use when flow injection–Orbitrap MSn data have been acquired in both positive and negative ionization modes, per-mode feature lists (e.g. from mzmine) and cleaned metadata exist, and the goal is to quantify how many unique compounds are detected in positive-only, negative-only, and both modes.

## When NOT to use

- Input is already a feature table with ionization-mode labels and a compound identifier column — no join or set-intersection work is needed
- Data were acquired in a single ionization mode only — the cross-mode intersection is undefined and per-mode counts alone suffice
- Metadata have not passed the metadata cleanup flow and lack consistent column names (unique sample id, plate_id, well location) — the feature-to-compound join would be ambiguous

## Inputs

- Per-ionization-mode mzmine feature lists from flow injection–Orbitrap MSn acquisitions (.mzML or converted .raw via ThermoRawFileParser)
- Cleaned metadata table with unique sample id, plate_id, and well/vial location following the shared template
- Compound-to-feature assignments from PubChem Name search or provided structure identifiers (LOTUS, DrugBank, DrugCentral, Dictionary of Natural Products, Broad Drug Repurposing Hub)

## Outputs

- detection_statistics.csv with unique compound counts per ionization mode and the set intersection
- Count breakdown: total unique compounds (30,008), positive-only (>12,200), negative-only (>3,400), both modes (~14,300)

## How to apply

Start from per-file mzmine feature lists derived from the .mzML/.raw flow injection–Orbitrap ID-X acquisitions (converting Thermo .raw with ThermoRawFileParser where needed) and the cleaned metadata table produced by the metadata cleanup flow. Join each detected feature to its unique compound identifier using the cleaned metadata's unique sample id, plate_id, and well/vial location fields, treating distinct ionization modes as separate feature lists. Assign every unique compound identifier a membership label for positive mode and negative mode (detected / not detected), then count positives-only, negatives-only, and the set intersection of compounds present in both ionization modes. Validate against the reported statistics: 30,008 unique small molecules in total, >12,200 exclusively positive, >3,400 exclusively negative, and ~14,300 in both modes, and write the breakdown to detection_statistics.csv. The Prefect 2 metadata cleanup flow (prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work) supplies the standardized metadata that makes this join unambiguous.

## Related tools

- **mzmine** (Processes the .mzML flow injection–Orbitrap acquisitions into per-ionization-mode feature lists that are joined against the cleaned metadata to assign unique compound identifiers) — https://github.com/mzmine/mzmine
- **ThermoRawFileParser** (Converts Thermo .raw files from the Orbitrap ID-X into open formats (mzML/MGF/Parquet) so mzmine can process both ionization-mode acquisitions) — https://github.com/CompOmics/ThermoRawFileParser
- **Prefect** (Runs the metadata_cleanup_prefect.py flow that standardizes metadata column names and minimum needed information, enabling a reliable feature-to-compound join) — https://github.com/corinnabrungs/msn_tree_library
- **Orbitrap ID-X** (Instrument that acquires the flow injection MSn data whose per-mode compound detections feed the intersection statistics)

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- Total unique compound count equals 30,008 after deduplicating across both ionization modes
- Positive-only count exceeds 12,200; negative-only count exceeds 3,400; both-modes count is approximately 14,300
- detection_statistics.csv contains rows for each of the three membership categories (positive-only, negative-only, both) with counts summing to the total
- Every detected feature in the joined table maps to a unique compound identifier sourced from the cleaned metadata (no orphan features)
- The intersection is computed as the set overlap of compound identifiers detected in positive mode and those detected in negative mode, not as a per-feature count

## Limitations

- Counts rely on metadata cleanup having been performed; inconsistent column names or missing well/vial locations break the join
- Databases needing a local file or special access (Dictionary of Natural Products, DrugBank) must be set appropriately in jobs.py or excluded, which can affect the compound identifier universe
- Structure information not provided is queried from PubChem by Name search, so name ambiguities propagate into the compound identifier assignment

## Evidence

- [abstract] MSnLib provides >2 million spectra in MSn trees covering 30,008 unique small molecules: "a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
- [readme] Metadata cleanup flow standardizes column names and minimum needed information for the query: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
- [abstract] High-throughput pipeline in mzmine builds the MSn library: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
- [methods] Sequence creation is set up for the Orbitrap ID-X (Xcalibur): "The sequence creation is setup for the Orbitrap ID-X (Xcalibur)."
- [readme] Databases needing local files or special access are toggled in jobs.py: "some need a local file and/or special access otherwise set it to False in the jobs.py"

---
name: bioassay-profile-normalization
description: Use when raw primary-screening assay results (BioMAP-style inhibition/no-inhibition
  columns, growth assays, etc.) must be assembled into normalized biological profiles
  before import into NP Analyst for Compound Activity Mapping.
when_to_use_negative:
- Input is already a normalized NP Analyst-ready profile table or the assay columns
  are all on an identical scale
- Input is mass spectrometry data (mzML, MZmine 2 peak lists, GNPS .graphml) rather
  than bioassay results — those go through replicate comparison and basketing instead
- The assays cannot be meaningfully placed on a common scale (e.g., heterogeneous
  assay types with no shared inhibition/no-inhibition semantics)
tools:
- name: NP Analyst
  role: Consumes the normalized biological profiles via the npanalyst CLI import step
    to build Compound Activity Maps
  repo: https://github.com/liningtonlab/npanalyst
- name: Docker
  role: Runtime container for the npanalyst CLI import command with a bind-mounted
    data volume
  repo: https://www.docker.com/get-started
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.1c01108
    title: 'NP Analyst: An Open Online Platform for Compound Activity Mapping'
  source_content_hash: sha256:e32f06aa4c308e00d7e1f2e8f0d546dc7af878aecaac046f3a4581b96ac08e79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/liningtonlab/npanalyst
  tools:
  - Docker
  - MZmine 2
  - NP Analyst
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# bioassay-profile-normalization

## Summary

Assembles individual bioassay results from a screening campaign (e.g., 15-pathogen BioMAP inhibition/no-inhibition tables for a 925-prefraction marine Actinobacterial library) into biological profiles whose columns share a common scale, so NP Analyst can compute comparable Activity Scores across features.

## When to use

Use when raw primary-screening assay results (BioMAP-style inhibition/no-inhibition columns, growth assays, etc.) must be assembled into normalized biological profiles before import into NP Analyst for Compound Activity Mapping.

## When NOT to use

- Input is already a normalized NP Analyst-ready profile table or the assay columns are all on an identical scale
- Input is mass spectrometry data (mzML, MZmine 2 peak lists, GNPS .graphml) rather than bioassay results — those go through replicate comparison and basketing instead
- The assays cannot be meaningfully placed on a common scale (e.g., heterogeneous assay types with no shared inhibition/no-inhibition semantics)

## Inputs

- Per-prefraction bioassay result tables (e.g., BioMAP 15-pathogen inhibition/no-inhibition CSV for 925 prefractions)
- Assay metadata specifying each column's scale/units for normalization

## Outputs

- Normalized biological-profile table (rows = extracts/prefractions, columns = assays on a common scale) suitable for npanalyst import

## How to apply

Gather all assay results measured on the same extract or prefraction library (e.g., the 925-prefraction BioMAP results across 15 pathogens) and assemble them into a profile table where each extract is a row and each assay a column. Normalize every column so all assays are on the same scale, since NP Analyst scores each MS feature against the combined profile and mixed scales would bias Activity and Cluster Scores. Import the normalized profiles into NP Analyst via the npanalyst CLI using the -t BioMAP type flag (Docker workflow described in the README); detailed normalization instructions are in the project documentation site. Validation is by checking that the imported profile preserves the inhibition/no-inhibition semantics of the source assay and that downstream feature scoring produces the expected community structure (e.g., 79 prefractions sharing activity against the same 6 pathogens, 23 of them in the micromonolactam-containing cluster).

## Related tools

- **NP Analyst** (Consumes the normalized biological profiles via the npanalyst CLI import step to build Compound Activity Maps) — https://github.com/liningtonlab/npanalyst
- **Docker** (Runtime container for the npanalyst CLI import command with a bind-mounted data volume) — https://www.docker.com/get-started

## Examples

```
docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/BioMAP.csv -o /data -t BioMAP -v
```

## Evaluation signals

- Every assay column in the output profile table is on the same scale (per-article requirement for NP Analyst input)
- Row count matches the source library size (925 prefractions for the BioMAP dataset)
- Imported profiles yield reproducible community structure downstream, e.g., the 79 prefractions with activity against the same 6 pathogens of which 23 fall in the micromonolactam-containing cluster
- npanalyst import with -t BioMAP completes without errors and downstream feature scoring produces parseable cluster CSVs

## Limitations

- Normalization must be performed before import; NP Analyst does not rescale heterogeneous assay columns itself
- Detailed normalization procedures are documented on the external documentation site rather than fully in the article

## Evidence

- [discussion] these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale: "these data are suitable for assembly into biological profiles for use in NP Analyst, provided that the data are normalized so that each column uses the same scale"
- [discussion] Detailed instructions for data normalization are included in the Web site documentation (https://liningtonlab.github.io/npanalyst_documentation/).: "Detailed instructions for data normalization are included in the Web site documentation (https://liningtonlab.github.io/npanalyst_documentation/)."
- [abstract] We now present NP Analyst, an open online platform for Compound Activity Mapping that accepts bioassay data of almost any type: "We now present NP Analyst, an open online platform for Compound Activity Mapping that accepts bioassay data of almost any type"
- [readme] docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v: "docker run -it -v PATH/TO/DATA:/data npanalyst-cli import -i /data/GNPS.graphml -o /data -t GNPS -v"

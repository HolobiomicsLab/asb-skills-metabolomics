---
name: compound-list-precursor-mass-calculation
description: Use when a validated compound list (CSV) plus a high-resolution mzML
  file must be converted into a set of precursor ion masses for extraction, and no
  explicit precursor m/z column is supplied.
when_to_use_negative:
- When the compound list already contains a precursor m/z column supplied from an
  external tool and the analyst wants to preserve those exact values instead of recalculating
  from structure.
- When screening masses rather than named compounds (mass list-based screening), because
  there is no structural information to convert to a precursor ion.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Shinyscreen
  role: Executes the adduct-aware precursor mass calculation automatically from the
    compound list's structural information before extraction
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: MSnbase
  role: Consumes the computed precursor masses to retrieve EICs and MS2 spectral scans
    from the mzML file within the m/z tolerance
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:1be909d075e5e0aa5e9798eabd611fa7af4731a6d7df92dcb407ee5bab57e6d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - mzR
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# compound-list-precursor-mass-calculation

## Summary

Automatically computes the precursor ion m/z values needed to extract MS1 and MS2 data for every compound in a validated CSV compound list, by applying the user's adduct setting to each compound's structural formula, so extraction targets match the actual ion species in the mzML file.

## When to use

Use when a validated compound list (CSV) plus a high-resolution mzML file must be converted into a set of precursor ion masses for extraction, and no explicit precursor m/z column is supplied.

## When NOT to use

- When the compound list already contains a precursor m/z column supplied from an external tool and the analyst wants to preserve those exact values instead of recalculating from structure.
- When screening masses rather than named compounds (mass list-based screening), because there is no structural information to convert to a precursor ion.

## Inputs

- Validated compound list CSV with structural information (formula / InChI / SMILES) for each compound
- Adduct setting chosen by the analyst (e.g. [M+H]+, [M-H]-, [M+Na]+)

## Outputs

- Set of precursor ion m/z values aligned one-to-one with the compound list entries, ready for the extraction and prescreening stage

## How to apply

Read the compound list (which already carries structural information such as formula, InChI or SMILES) and the user's chosen adduct setting (e.g. [M+H]+ or [M-H]-) as inputs. Shinyscreen converts each compound's neutral formula into the charged precursor ion mass by adding or removing the adduct atoms and correcting for electron mass, yielding the m/z that the extraction step then searches for inside the specified m/z tolerance. This automatic calculation means the analyst only supplies an adduct setting, not hand-computed m/z values, reducing transcription errors across large lists. The computed masses feed directly into the extraction and prescreening stage where EICs are retrieved within the experimental m/z error tolerance. Validate by spot-checking a few computed m/z values against manual calculation or against known reference masses in MassBank or PubChem.

## Related tools

- **Shinyscreen** (Executes the adduct-aware precursor mass calculation automatically from the compound list's structural information before extraction) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **MSnbase** (Consumes the computed precursor masses to retrieve EICs and MS2 spectral scans from the mzML file within the m/z tolerance)

## Examples

```
R: Shinyscreen::init(compound_list = 'compounds.csv', adduct = '[M+H]+', mzML_file = 'run.mzML')  # precursor masses are computed automatically
```

## Evaluation signals

- Spot-check 3–5 computed precursor m/z values against independent calculation (e.g. PubChem molecular weight + adduct mass + proton mass) within the experimental m/z error tolerance
- Cross-reference computed masses against known precursor ions in MassBank or PubChemLite for the same compounds and adducts
- Confirm that every compound-list entry yields exactly one precursor m/z (no missing or duplicated entries after adduct conversion)
- Verify the downstream extraction EICs actually contain signal at the computed m/z, and flag compounds whose EIC is empty as candidate mismatches between structure and adduct

## Limitations

- Precursor masses are only as correct as the structural information in the compound list; wrong formulas propagate silently into extraction targets
- Only one adduct per compound is applied per run; analysts must rerun with different adduct settings to cover alternative ion species
- Requires the mzML data to actually contain the ion species corresponding to the chosen adduct — signals from other adducts will be missed

## Evidence

- [methods] Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [methods] Shinyscreen can automatically convert the 'simple chemical notation' of the compound list entries into SMILES structures for computation: [evidence span withheld — anchor status: not_found]
- [methods] its precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans are retrieved from the data: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans are retrieved from the"

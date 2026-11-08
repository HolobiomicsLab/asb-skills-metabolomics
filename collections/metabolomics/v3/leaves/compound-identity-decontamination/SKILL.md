---
name: compound-identity-decontamination
description: Use when a GC-MS library-hit table contains tentative Compound.Name entries
  with varying Match.Factor values and you must drop unreliable IDs (no strong cross-sample
  match, no PubChem record, or no canonical SMILES) before aggregating Component.Area
  across samples.
when_to_use_negative:
- Input is already a validated, manually curated compound-area table where every ID
  has a PubChem CID and canonical SMILES — decontamination would add no value.
- 'Data are MS2, MS3, or ion mobility + MS2 spectra: uafR depends on published data
  and its algorithms ''do not yet apply'' to these modes.'
- You need to keep low-match-factor tentative IDs for exploratory review rather than
  quantitative aggregation — the strong-match and SMILES filters would silently discard
  them.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: uafR
  role: R package providing spreadOut() table validation and mzExacto(decontaminate
    = TRUE) identity filtering and cross-sample area aggregation
  repo: https://github.com/castratton/uafR
- name: R
  role: Runtime environment for installing and executing uafR
- name: PubChem
  role: Public chemical database consulted during decontamination; chemicals lacking
    a PubChem entry are removed
- name: Agilent Unknowns Analysis
  role: Upstream peak deconvolution and library search producing the hit table that
    decontamination filters
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:b834e6231e893ddc5e2f653c6f84f6dac0b96f2244951d164a325cc138c27d73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castratton/uafR
  tools:
  - Agilent Unknowns Analysis
  - PubChem
  - R
  - uafR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# compound-identity-decontamination

## Summary

A uafR filtering step (mzExacto with decontaminate = TRUE) that removes GC-MS tentative identifications lacking a strong cross-sample match, a PubChem entry, or a canonical SMILES, so only verifiable compounds are carried into area aggregation and quantification.

## When to use

Use when a GC-MS library-hit table contains tentative Compound.Name entries with varying Match.Factor values and you must drop unreliable IDs (no strong cross-sample match, no PubChem record, or no canonical SMILES) before aggregating Component.Area across samples.

## When NOT to use

- Input is already a validated, manually curated compound-area table where every ID has a PubChem CID and canonical SMILES — decontamination would add no value.
- Data are MS2, MS3, or ion mobility + MS2 spectra: uafR depends on published data and its algorithms 'do not yet apply' to these modes.
- You need to keep low-match-factor tentative IDs for exploratory review rather than quantitative aggregation — the strong-match and SMILES filters would silently discard them.

## Inputs

- GC-MS library-hit table (CSV from Agilent Unknowns Analysis) with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- User-defined chemical query list (vector of compound names to extract)
- spreadOut()-processed data frame (spread object)

## Outputs

- Decontaminated compound list (chemicals with strong cross-sample match, PubChem entry, and canonical SMILES)
- Per-compound aggregated Component.Area values across samples with identity, optimal RT, exact mass, and best Match.Factor

## How to apply

First run spreadOut() on the Agilent Unknowns Analysis CSV export, which validates the required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and rejects duplicate peak keys before any live lookups. Then call mzExacto() with the spread object and your chemical query list, setting decontaminate = TRUE: this removes chemicals that did not have a strong match across samples, could not be found in PubChem, and/or could not have a canonical SMILES generated. Optionally pre-subset candidates by Match.Factor threshold (the paper illustrates Match.Factor > 80 for 'top hits') or by structural Tanimoto strength (strong > 0.95, moderate > 0.85) before extraction. mzExacto then aggregates candidate Component.Area values across every GC-MS sample for the surviving compounds and returns identity, optimal retention time, exact mass, best match factor, and aggregated area. Judge success by confirming removed IDs genuinely lack cross-sample support or database records, and that retained compounds (e.g. methyl salicylate in the internal-standards data) yield plausible aggregated areas correlated with known quantities (R2 0.8273-0.9998 for the standards).

## Related tools

- **uafR** (R package providing spreadOut() table validation and mzExacto(decontaminate = TRUE) identity filtering and cross-sample area aggregation) — https://github.com/castratton/uafR
- **R** (Runtime environment for installing and executing uafR)
- **PubChem** (Public chemical database consulted during decontamination; chemicals lacking a PubChem entry are removed)
- **Agilent Unknowns Analysis** (Upstream peak deconvolution and library search producing the hit table that decontamination filters)

## Examples

```
spread <- spreadOut(input_dat); extracted <- mzExacto(data_in = spread, chemicals = targets, decontaminate = TRUE)
```

## Evaluation signals

- Required-column check passes: spreadOut() validates Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor and rejects duplicate peak keys.
- Every retained compound has a strong cross-sample match, a PubChem entry, and a canonical SMILES; removed chemicals lack at least one of these.
- On the internal-standards dataset (ethyl hexanoate, methyl salicylate, octanal, undecane), per-compound R2 of aggregated area vs. injected quantity falls within the reported 0.8273-0.9998 range across 1e-5 to 1e3 ng/ul.
- Retained compounds return best Match.Factor, exact mass, and optimal retention time alongside aggregated Component.Area.
- Tanimoto-based match grades are consistent with the paper's thresholds (> 0.95 strong, > 0.85 moderate) when structural filtering is layered on.

## Limitations

- Manual validation remains necessary: researchers should still check that compound areas 'make chemical and/or biological sense in their study system'.
- Untested on other mass spectrometers (Q-TOF, Orbitrap, TimsTOF HT, Astral) and on LC-MS data.
- Does not apply to MS2, MS3, or ion mobility + MS2 data; support is deferred until such spectra are published in PubChem.
- Decontamination depends on live PubChem/NCBI lookups, so records missing from public databases are dropped even if the ID might be real.

## Evidence

- [intro] removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated: "removes any chemicals that did not have a strong match across samples, were unable to be found in public databases (i.e. PubChem), and/or were unable to have a canonical SMILES generated"
- [methods] selected = mzExacto(spread, query_compounds, decontaminate = TRUE): "selected = mzExacto(spread, query_compounds, decontaminate = TRUE)"
- [results] Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998 (Fig 2): "Peak areas calculated by uafR for the set of standards correlated with the volume of the standards injected, with R2 values ranging from 0.8273 to 0.9998 (Fig 2)"
- [intro] A simple and effective approach is to subset the search chemicals by setting a minimal match factor on the raw output of Unknowns Analysis: "A simple and effective approach is to subset the search chemicals by setting a minimal match factor on the raw output of Unknowns Analysis"
- [intro] whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match: "whether it has a strong (Tanimoto similarity greater than 0.95) or moderate (greater than 0.85) structural match"

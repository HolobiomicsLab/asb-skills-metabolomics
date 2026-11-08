---
name: precursor-mass-tolerance-benchmarking
description: Use when benchmarking dereplication tools (DEREPLICATOR+, DEREPLICATOR-G,
  DEREPLICATOR) on spectra with known metabolite annotations and you need to measure
  how precursor mass tolerance settings (e.g. 0.5 Da vs 0.02 Da) change top-rank identification
  rates.
when_to_use_negative:
- The goal is discovery of unannotated 'dark matter' spectra or novel variants via
  spectral networks, not controlled benchmarking of identification rates on known-annotation
  ground truth.
- Compounds fall outside the classes the fragmentation-graph scoring supports — the
  article states DEREPLICATOR+ is not applicable to all classes of compounds.
- Spectra are not centroided or not in an open format (MGF, mzXML, mzML, mzData) —
  preconvert first with msconvert rather than running the benchmark directly.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: DEREPLICATOR+
  role: Primary database-search tool for dereplicating spectra against diverse metabolites
    (peptidic and non-peptidic); run at both tolerance settings
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR-G
  role: Comparison search tool included in the benchmark to baseline top-rank rates
    against
- name: DEREPLICATOR
  role: PNP-only baseline search tool for identifying peptidic natural products in
    the benchmark comparison
  repo: https://github.com/ablab/npdtools
- name: GNPS
  role: Source of spectral library structures merged with DNP to form the reference
    structure database, and online platform for running the pipelines
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:c784b3e06d25a89bc20553733f4afd4882b517f0785800f51ef41f09db078f66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - DEREPLICATOR-G
  - GNPS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# precursor-mass-tolerance-benchmarking

## Summary

Benchmark protocol for quantifying how precursor mass tolerance affects top-rank identification rates of spectral database-search dereplication tools (DEREPLICATOR+, DEREPLICATOR-G, DEREPLICATOR) on spectra with known ground-truth metabolite annotations.

## When to use

Use when benchmarking dereplication tools (DEREPLICATOR+, DEREPLICATOR-G, DEREPLICATOR) on spectra with known metabolite annotations and you need to measure how precursor mass tolerance settings (e.g. 0.5 Da vs 0.02 Da) change top-rank identification rates.

## When NOT to use

- The goal is discovery of unannotated 'dark matter' spectra or novel variants via spectral networks, not controlled benchmarking of identification rates on known-annotation ground truth.
- Compounds fall outside the classes the fragmentation-graph scoring supports — the article states DEREPLICATOR+ is not applicable to all classes of compounds.
- Spectra are not centroided or not in an open format (MGF, mzXML, mzML, mzData) — preconvert first with msconvert rather than running the benchmark directly.

## Inputs

- Set of centroided LC-MS/MS spectra with known metabolite annotations as ground truth (e.g. 5473 SpectraLibrary spectra; MGF, mzXML, mzML, or mzData format)
- Chemical structure database of known natural products (DNP merged with GNPS library structures; 86,665 structures)
- Executable binaries for DEREPLICATOR+, DEREPLICATOR-G, and DEREPLICATOR (NPDtools bin directory)

## Outputs

- top_rank_percentages.csv tabulating top-rank correct-hit counts and percentages per tool per precursor tolerance (0.5 Da and 0.02 Da)
- Per-spectrum ranked metabolite-spectrum matches (MSMs) scored by unique matched masses

## How to apply

Compile the benchmark spectra set (e.g. 5473 SpectraLibrary spectra whose annotated metabolites serve as ground truth) and merge a reference structure database by combining the Dictionary of Natural Products (DNP) with GNPS spectral library structures (86,665 structures in the reference benchmark). Ensure spectra are centroided and converted to MGF (via ProteoWizard msconvert if starting from mzXML/mzML), then run each search tool (DEREPLICATOR+, DEREPLICATOR-G, DEREPLICATOR) against the merged structure library twice, once at 0.5 Da and once at 0.02 Da precursor mass tolerance, keeping all other parameters fixed. Note that the tools preprocess spectra by retaining only the six top-intensity peaks per 50 Da window, and score each metabolite-spectrum match (MSM) as the number of unique masses in the spectrum that annotate nodes of the metabolite's fragmentation graph. For each tool and tolerance, rank candidate metabolite-spectrum matches by score and count how many spectra have the correct annotated metabolite as the top-ranked hit. Tabulate top-rank percentages and absolute counts per tool per tolerance (reference result: DEREPLICATOR+ 34%/1878 at 0.5 Da rising to 44%/2414 at 0.02 Da; DEREPLICATOR-G 30%/1647 to 40%/2216; DEREPLICATOR 17%/964 to 25%/1362) into a top_rank_percentages.csv file.

## Related tools

- **DEREPLICATOR+** (Primary database-search tool for dereplicating spectra against diverse metabolites (peptidic and non-peptidic); run at both tolerance settings) — https://github.com/ablab/npdtools
- **DEREPLICATOR-G** (Comparison search tool included in the benchmark to baseline top-rank rates against)
- **DEREPLICATOR** (PNP-only baseline search tool for identifying peptidic natural products in the benchmark comparison) — https://github.com/ablab/npdtools
- **GNPS** (Source of spectral library structures merged with DNP to form the reference structure database, and online platform for running the pipelines)

## Examples

```
msconvert spectra.mzML --mgf -o mgf/ ; python <npdtools_installation_dir>/bin/dereplicator+.py --spectra mgf/spectra.mgf --database dnp_gnps_structures
```

## Evaluation signals

- Top-rank counts at 0.5 Da / 0.02 Da match published values over the 5473-spectrum set: DEREPLICATOR+ 1878/2414, DEREPLICATOR-G 1647/2216, DEREPLICATOR 964/1362.
- Corresponding percentages match: 34%/44% (DEREPLICATOR+), 30%/40% (DEREPLICATOR-G), 17%/25% (DEREPLICATOR).
- Tightening tolerance from 0.5 Da to 0.02 Da monotonically increases top-rank identification rates for all three tools (deltas of +10, +10, and +8 percentage points).
- Merged reference library contains exactly 86,665 chemical structures after combining DNP and GNPS library structures.
- All denominators equal 5473 spectra, confirming every benchmark spectrum was searched at both tolerances.

## Limitations

- The article states no version numbers for DEREPLICATOR+, DEREPLICATOR, VarQuest, or GNPS, so benchmark results are version-dependent and hard to reproduce exactly.
- DEREPLICATOR+ uses a simple shared-peaks scoring scheme that results in a high false-negative rate; improved scoring was not implemented in the paper.
- DEREPLICATOR+ is not applicable to all classes of compounds; DEREPLICATOR is limited to peptidic natural products and cannot identify polyketides or terpenes.

## Evidence

- [intro] Recently, we introduced DEREPLICATOR32 for searching spectral datasets against the database of peptidic natural products (PNPs): "Recently, we introduced DEREPLICATOR32 for searching spectral datasets against the database of peptidic natural products (PNPs)"
- [readme] Peak filtering and spectrum conversion prerequisites: [evidence span withheld — anchor status: external_doc]
- [discussion] Spectrum preprocessing step applied before scoring: "DEREPLICATOR+ processes all spectra by retaining only six top intensity peaks in each window of size 50 Da."
- [discussion] Definition of the MSM scoring function used to rank candidates: "Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] Decoy generation underpins statistical significance / FDR evaluation of matches: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [methods] Format conversion path for input spectra: "uses msconvert utility from the ProteoWizard package to convert spectra in other formats to MGF"

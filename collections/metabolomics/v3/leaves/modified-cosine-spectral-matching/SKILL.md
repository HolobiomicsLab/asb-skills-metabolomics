---
name: modified-cosine-spectral-matching
description: Use when propagating MS/MS library annotations (e.g., drugs and their
  metabolites/analogs from the GNPS Drug Library) into untargeted LC-MS/MS metabolomics
  datasets, where peaks may have different precursor m/z than the reference but share
  fragment and neutral-loss structure.
when_to_use_negative:
- When spectra are expected to share an identical precursor m/z and full fragment
  pattern (e.g., direct library identification rather than analog/metabolite propagation)
  — plain cosine scoring suffices and modified cosine's mass-shift tolerance is unnecessary.
- When the reference library does not contain pharmacologic metadata needed to propagate
  a meaningful annotation (e.g., a drug library lacking drug class or indication fields),
  as matches cannot be interpreted beyond spectral identity.
- When input spectra are not LC-MS/MS fragmentation data (e.g., GC-EI-MS or MS1-only
  data), as the modified cosine score depends on paired MS/MS fragment ion lists.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Hosts the GNPS Drug Library of drug/drug-metabolite MS/MS reference spectra
    and provides the FastMASST / Fast Search API against which batch modified-cosine
    analog searches are executed
- name: microbe_masst (domainMASSTs)
  role: Provides jobs.py batch-search code that leverages the Fast Search API to run
    modified-cosine searches of multiple spectra against indexed GNPS/MassIVE, Metabolomics
    Workbench, Metabolights, and NORMAN data and produce _matches.tsv, _library.tsv,
    _datasets.tsv, and per-domain HTML/JSON outputs
  repo: https://github.com/robinschmid/microbe_masst
- name: Manuscript_GNPS_Drug_Library
  role: Repository associated with the source article for the GNPS Drug Library used
    as the reference spectral library in modified-cosine matching
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:f53f6919b58200bc959597ace2ec43ce24f9ecf4006326d152db7eb3b37ae48d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - Manuscript_GNPS_Drug_Library
  - microbe_masst (domainMASSTs)
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Modified cosine spectral matching

## Summary

Alignment of MS/MS reference spectra against untargeted metabolomics peaks using modified cosine scoring (accounting for neutral-loss differences between precursor masses) to propagate drug metabolite and analog annotations from a curated drug spectral library into public datasets.

## When to use

Use when propagating MS/MS library annotations (e.g., drugs and their metabolites/analogs from the GNPS Drug Library) into untargeted LC-MS/MS metabolomics datasets, where peaks may have different precursor m/z than the reference but share fragment and neutral-loss structure.

## When NOT to use

- When spectra are expected to share an identical precursor m/z and full fragment pattern (e.g., direct library identification rather than analog/metabolite propagation) — plain cosine scoring suffices and modified cosine's mass-shift tolerance is unnecessary.
- When the reference library does not contain pharmacologic metadata needed to propagate a meaningful annotation (e.g., a drug library lacking drug class or indication fields), as matches cannot be interpreted beyond spectral identity.
- When input spectra are not LC-MS/MS fragmentation data (e.g., GC-EI-MS or MS1-only data), as the modified cosine score depends on paired MS/MS fragment ion lists.

## Inputs

- Reference MS/MS spectra library (e.g., GNPS Drug Library drug/drug-metabolite spectra, 103,209 entries)
- Peak lists or MGF files from untargeted LC-MS/MS metabolomics datasets (GNPS/MassIVE, MetaboLights, Metabolomics Workbench)
- List of unique spectrum identifiers (USIs) or .mgf file for batch search
- Search parameter settings: minimum cosine score, m/z tolerance, minimum matched peaks

## Outputs

- Matched reference-to-dataset spectral pair table (e.g., drug_analog_spectral_pairs.tsv) with USI mappings
- Per-spectrum _matches.tsv listing scans matching searched spectrum
- _library.tsv of matching GNPS library spectra enabling Level 2 MSI annotation
- _datasets.tsv with unique sample counts per dataset
- Interactive HTML trees and JSON files per domain-specific MASST (e.g., _microbe.html, _microbe.json)

## How to apply

Load reference MS/MS spectra (e.g., the 103,209 drug and drug-metabolite spectra in the GNPS Drug Library) and peak lists from public repositories such as GNPS/MassIVE, MetaboLights, and Metabolomics Workbench (~3500 LC-MS/MS datasets). For each reference-versus-peak pair, compute modified cosine similarity, which scores matched fragment ions while tolerating precursor-mass shifts attributable to analogs or metabolites (differences arising from neutral losses between the two precursor m/z values). Retain pairs whose modified cosine score is at least 0.8 AND with at least 6 matched fragment ions — these dual thresholds suppress spurious low-information matches while still capturing spectra of analogs/metabolites whose precursor differs from the reference. Assign each surviving match the library spectrum's pharmacologic metadata (exposure source, pharmacologic class, therapeutic indication) and record both spectra's unique spectrum identifiers (USIs) and dataset of origin. Rerun batch searches via the Fast Search API until no new matches appear, since API calls can intermittently fail (skip_existing=True).

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Hosts the GNPS Drug Library of drug/drug-metabolite MS/MS reference spectra and provides the FastMASST / Fast Search API against which batch modified-cosine analog searches are executed)
- **microbe_masst (domainMASSTs)** (Provides jobs.py batch-search code that leverages the Fast Search API to run modified-cosine searches of multiple spectra against indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data and produce _matches.tsv, _library.tsv, _datasets.tsv, and per-domain HTML/JSON outputs) — https://github.com/robinschmid/microbe_masst
- **Manuscript_GNPS_Drug_Library** (Repository associated with the source article for the GNPS Drug Library used as the reference spectral library in modified-cosine matching) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Examples

```
python jobs.py  # with entries ('input.mgf', 'output/drugmatch') and adjusted min cosine, m/z tolerance, min matched peaks
```

## Evaluation signals

- Every retained spectral pair must satisfy modified cosine score >= 0.8 AND matched ion count >= 6 — pairs below either threshold indicate misapplied thresholds.
- The output drug-analog pair table row count should be in the order of >19 million when searching ~3500 public LC-MS/MS datasets with 103,209 reference spectra; drastically fewer rows suggest failed API calls that were not re-run.
- Each output pair must carry a resolvable unique spectrum identifier (USI) for both the reference spectrum and the dataset peak, plus the dataset of origin.
- Re-running jobs.py with skip_existing=True should produce no substantially new matches; persistent new output on re-runs indicates incomplete indexing.
- Library-level matches reported in _library.tsv should permit Level 2 Metabolomics Standards Initiative annotation of the corresponding peaks.

## Limitations

- Fast Search API calls can fail intermittently; jobs.py must be re-run (with skip_existing=True) until no new output is generated to capture all matches.
- Requires Python 3.10 for jobs.py batch execution.
- The GNPS Drug Library is not intended for immediate clinical decision-making; matched drug annotations support research, not clinical action.
- Modified cosine matching tolerates precursor mass shifts, so hits may correspond to analogs/metabolites rather than the exact reference drug; downstream interpretation requires pharmacologic metadata.
- Matches depend on what has been currently indexed in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN; newly deposited datasets are not searchable until indexed.

## Evidence

- [methods] FastMASST analog search threshold: [evidence span withheld — anchor status: not_found]
- [readme] Fast Search API batch search description: [evidence span withheld — anchor status: external_doc]
- [readme] jobs.py re-run caveat: [evidence span withheld — anchor status: external_doc]
- [readme] Search parameters adjustment: [evidence span withheld — anchor status: external_doc]
- [discussion] Clinical use disclaimer: "While the GNPS Drug Library is not intended for immediate clinical decision-making, this tool lays the groundwork for future integration of exposure data into precision medicine frameworks"

---
name: spectral-library-matching
description: Use when query MS/MS spectra (.mgf from MZmine or USI lists in .csv/.tsv)
  need library annotation and taxonomic attribution against microbial monoculture
  data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN.
when_to_use_negative:
- The input is already a feature table or annotated peak list with no MS/MS peak lists
  available — library matching requires fragment spectra.
- The goal is de novo structural elucidation of molecules absent from all reference
  libraries; spectral matching only assigns hits present in GNPS libraries or indexed
  datasets.
- Searches against non-microbial domains (plants, tissues, food) as the target of
  interest — use the corresponding plantMASST/tissueMASST/foodMASST domain tool instead
  of microbeMASST.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: microbeMASST
  role: Taxonomically informed domain-specific MASST whose curated microbial monoculture
    index the spectra are searched against
  repo: https://masst.gnps2.org/microbemasst/
- name: Fast Search API
  role: Backend API invoked by jobs.py to execute the batch spectral search against
    indexed repositories
  repo: https://fasst.gnps2.org/fastsearch/
- name: jobs.py (microbe_masst)
  role: Batch runner that submits spectra, sets cosine/mz/peak-match parameters, and
    writes _matches.tsv, _library.tsv and count outputs
  repo: https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- name: MZmine
  role: Generates the single .mgf peak-list file used as batch search input
  repo: https://github.com/mzmine/mzmine
- name: GNPS libraries
  role: Reference spectral library whose matches in _library.tsv enable Level 2 Metabolomics
    Standards Initiative annotations
  repo: https://library.gnps2.org/
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41564-023-01575-9
    title: 'microbeMASST: a taxonomically informed mass spectrometry search tool for
      microbial metabolomics data'
  source_content_hash: sha256:26b99cbd569b2d62c59cefadb1851d9fe4e2055244c01c248cba0668f23160f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fast Search API
  - GNPS libraries
  - MZmine
  - MassBank/MassBank-data
  - jobs.py (microbe_masst)
  - microbeMASST
  - mzmine
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spectral-library-matching

## Summary

Match query MS/MS spectra against curated spectral reference libraries and domain-specific MASST databases (microbeMASST) via the GNPS Fast Search API to assign Level 2 Metabolomics Standards Initiative annotations and taxonomic provenance to microbial metabolites.

## When to use

Use when query MS/MS spectra (.mgf from MZmine or USI lists in .csv/.tsv) need library annotation and taxonomic attribution against microbial monoculture data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN.

## When NOT to use

- The input is already a feature table or annotated peak list with no MS/MS peak lists available — library matching requires fragment spectra.
- The goal is de novo structural elucidation of molecules absent from all reference libraries; spectral matching only assigns hits present in GNPS libraries or indexed datasets.
- Searches against non-microbial domains (plants, tissues, food) as the target of interest — use the corresponding plantMASST/tissueMASST/foodMASST domain tool instead of microbeMASST.

## Inputs

- MZmine-generated single-entry .mgf file of MS/MS spectra
- USI list in .csv or .tsv format
- GNPS library MS/MS spectra of reference compounds (e.g. lovastatin, salinosporamide A, commendamide)
- Indexed spectra in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN (server-side)

## Outputs

- _matches.tsv listing all matched scans per searched spectrum
- _library.tsv listing GNPS library spectra matched to each query (Level 2 MSI annotation evidence)
- _datasets.tsv with counts of unique matching samples per dataset
- _count_domain.tsv / _count_microbe.tsv taxonomic match counts
- interactive _domain.html trees and _domain.json files per domain-specific MASST

## How to apply

Prepare query spectra as a single MZmine-generated .mgf file or a USI list in .csv/.tsv, then run the batch jobs.py runner from robinschmid/microbe_masst, which calls the Fast Search API (https://fasst.gnps2.org/fastsearch/) against all indexed GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN data. Apply microbeMASST default search parameters — modified cosine score >= 0.7, 0.05 Da precursor/fragment m/z tolerance, and at least 3 matched ions — adjusting them per the research question. Restrict returned hits to the curated domain (e.g. bacterial, fungal, archaeal monoculture extracts) and cross-reference the generated _library.tsv against GNPS library spectra to claim Level 2 MSI annotations. Re-run jobs.py with skip_existing=True a couple of times until no new output appears, because individual Fast Search API calls can silently fail. Parse _matches.tsv (matched scans, including samples outside the curated domain), _datasets.tsv (unique matching samples per dataset), and _count_domain.tsv / _count_microbe.tsv to aggregate per-compound taxonomic hits such as lovastatin uniquely to Aspergillus or salinosporamide A only to Salinispora tropica.

## Related tools

- **microbeMASST** (Taxonomically informed domain-specific MASST whose curated microbial monoculture index the spectra are searched against) — https://masst.gnps2.org/microbemasst/
- **Fast Search API** (Backend API invoked by jobs.py to execute the batch spectral search against indexed repositories) — https://fasst.gnps2.org/fastsearch/
- **jobs.py (microbe_masst)** (Batch runner that submits spectra, sets cosine/mz/peak-match parameters, and writes _matches.tsv, _library.tsv and count outputs) — https://github.com/robinschmid/microbe_masst/blob/master/code/jobs.py
- **MZmine** (Generates the single .mgf peak-list file used as batch search input) — https://github.com/mzmine/mzmine
- **GNPS libraries** (Reference spectral library whose matches in _library.tsv enable Level 2 Metabolomics Standards Initiative annotations) — https://library.gnps2.org/

## Examples

```
python code/jobs.py  # entries: ("input/spectra.mgf", "output/lovastatin"), skip_existing=True; Fast Search API at https://fasst.gnps2.org/fastsearch/
```

## Evaluation signals

- Re-run jobs.py with skip_existing=True until no new output is generated; no empty result sets should remain after convergence.
- _matches.tsv is non-empty and each query spectrum has at least one matched scan meeting cosine >= 0.7 and >= 3 matched ions at 0.05 m/z tolerance.
- _library.tsv contains GNPS library hits for queries expected to be known compounds, supporting Level 2 MSI annotation claims.
- For the nine-compound microbeMASST validation, the aggregated table has exactly nine compound rows, lovastatin matches Aspergillus exclusively, and salinosporamide A maps only to Salinispora tropica with no human cell line matches.
- Taxonomic counts in _count_microbe.tsv fall within microbeMASST's documented coverage (8 kingdoms, 20 phyla, 278 families, 561 genera, 1379 species, 542 strains).

## Limitations

- Some Fast Search API entries silently fail; sequential re-runs with skip_existing=True are needed to catch all possible matches.
- _matches.tsv includes samples that are not part of the curated domain-specific MASSTs, so hits must be filtered to the target domain.
- microbeMASST coverage is bounded (561 genera, 1379 species, 542 strains), so negative matches do not prove absence of a microbial producer.
- Must use Python 3.10 with jobs.py; other versions are not supported per the README.

## Evidence

- [results] microbeMASST leverages the Fast Search Tool to search a user-input MS/MS spectrum against all indexed MS/MS spectra in GNPS/MassIVE, restricting the search to spectra likely to achieve high modified cosine similarity under configurable precursor/fragment tolerances and score thresholds.: [evidence span withheld — anchor status: not_found]
- [methods] Apply the microbeMASST default search parameters: [evidence span withheld — anchor status: not_found]
- [readme] A _library.tsv file will be generated. This contains a list of spectra from the [GNPS libraries] found to match your spectrum of interest. This enables a Level 2 annotation according the Metabolomics Standards Initiative.: "A _library.tsv file will be generated. This contains a list of spectra from the [GNPS libraries](https://library.gnps2.org/) found to match your spectrum of interest. This enables a Level 2"
- [readme] Make sure to run jobs.py a couple of times, until no new output is generated by having the option: skip_existing=True. Due to the Fast Search API some of the entries will fail.: "Make sure to run [jobs.py] **_a couple of times_**, until no new output is generated by having the option: `skip_existing=True`. Due to the Fast Search API some of the entries will fail."
- [readme] This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed. This includes also samples that are not part of the curated domain-specific MASSTs.: "This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed. This includes also samples that are not part of the curated domain-specific"

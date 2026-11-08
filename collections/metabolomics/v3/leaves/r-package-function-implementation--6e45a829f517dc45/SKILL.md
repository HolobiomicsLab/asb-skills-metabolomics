---
name: r-package-function-implementation
description: Use when a geoRge result object of labeled base-peak m/z values (s2 from
  basepeak_finder) must be mapped to putative metabolites in a database with explicit
  adduct annotations (e.g. -H, +Cl).
when_to_use_negative:
- The data has not been processed by XCMS (peak picking, alignment and grouping) into
  an XCMSet, or no basepeak_finder s2 object exists to annotate
- The candidate metabolites are not represented in the supplied database, or the measured
  ions are not explained by the adducts in the provided adduct list
- The task is the upstream detection of isotopically labeled pairs itself (PuInc_seeker
  / basepeak_finder scope) rather than annotating already-detected labeled features
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: geoRge
  role: Source R package providing database_query, its extdata resources (ExampleDatabase.csv,
    adducts_negative.txt) and the upstream PuInc_seeker / basepeak_finder functions
    producing s2
  repo: https://github.com/jcapelladesto/geoRge
- name: XCMS
  role: Upstream peak picking, alignment and grouping that produces the XCMSet input
    structure consumed by geoRge
  repo: https://bioconductor.org/packages/release/bioc/html/xcms.html
- name: R
  role: Implementation language for reconstructing the database_query function and
    running the end-to-end script
- name: devtools
  role: Used to install the geoRge package from GitHub via install_github
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5b03628
    title: 'geoRge: A Computational Tool To Detect the Presence of Stable Isotope
      Labeling in LC/MS-Based Untargeted Metabolomics'
  source_content_hash: sha256:942f6f2389cd7471dc7465356d4bb680be3a99ede64251cc3589c3428aaa0d85
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/geoRge
  tools:
  - R
  - XCMS
  - devtools
  - geoRge
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# r-package-function-implementation

## Summary

Reimplement or wrap the geoRge database_query function, which annotates isotope-labeled base-peak features (from basepeak_finder's s2 object) with putative metabolite identities by matching measured m/z against a metabolite database under negative-mode adduct hypotheses.

## When to use

Use when a geoRge result object of labeled base-peak m/z values (s2 from basepeak_finder) must be mapped to putative metabolites in a database with explicit adduct annotations (e.g. -H, +Cl).

## When NOT to use

- The data has not been processed by XCMS (peak picking, alignment and grouping) into an XCMSet, or no basepeak_finder s2 object exists to annotate
- The candidate metabolites are not represented in the supplied database, or the measured ions are not explained by the adducts in the provided adduct list
- The task is the upstream detection of isotopically labeled pairs itself (PuInc_seeker / basepeak_finder scope) rather than annotating already-detected labeled features

## Inputs

- geoRge result object s2 produced by basepeak_finder (labeled base-peak m/z values)
- XCMS XCMSet object (e.g. the mtbls213 dataset) used upstream by PuInc_seeker and basepeak_finder
- Metabolite database CSV, e.g. ExampleDatabase.csv from geoRge extdata
- Negative-mode adduct table, e.g. adducts_negative.txt

## Outputs

- Data frame of annotated hits mapping labeled features to database metabolites with adduct annotations
- hits.csv file containing the annotated feature-to-metabolite mappings including adduct information

## How to apply

First reproduce the upstream geoRge chain: run PuInc_seeker on the XCMS XCMSet (e.g. the mtbls213 example data) with ULtag 'CELL_Glc12', Ltag 'CELL_Glc13', then basepeak_finder with UL.atomM=12.0, L.atomM=13.003355, ppm.s=6.5 and Basepeak.minInt=2000 to obtain s2. Load the annotation resources shipped in the package extdata directory: ExampleDatabase.csv as the metabolite database and adducts_negative.txt as the negative-mode adduct table. Implement the function with the exported signature database_query(geoRgeR, adducts, db, ppm.db), which matches each labeled base-peak m/z in the geoRge result object against metabolite masses in the database within the ppm.db mass tolerance and reports the matching metabolite together with the negative-mode adduct (e.g. -H, +Cl) explaining the measured m/z. Call it with geoRgeR = s2, adducts = the negative adduct table and db = ExampleDatabase.csv, then write the annotated feature-to-metabolite mappings including adduct information to hits.csv. The rationale is that isotopically enriched compounds shift their isotopic distribution in labeled vs unlabeled samples, and the surviving base peaks can only be identified by testing database metabolite masses under plausible adduct transformations.

## Related tools

- **geoRge** (Source R package providing database_query, its extdata resources (ExampleDatabase.csv, adducts_negative.txt) and the upstream PuInc_seeker / basepeak_finder functions producing s2) — https://github.com/jcapelladesto/geoRge
- **XCMS** (Upstream peak picking, alignment and grouping that produces the XCMSet input structure consumed by geoRge) — https://bioconductor.org/packages/release/bioc/html/xcms.html
- **R** (Implementation language for reconstructing the database_query function and running the end-to-end script)
- **devtools** (Used to install the geoRge package from GitHub via install_github)

## Examples

```
db <- read.csv(system.file("extdata/ExampleDatabase.csv", package="geoRge")); hits <- database_query(geoRgeR = s2, adducts = negative, db = db)
```

## Evaluation signals

- The script runs end-to-end without errors and the output file hits.csv exists
- Every annotated hit in hits.csv contains both a database metabolite entry and an adduct annotation (e.g. -H, +Cl)
- Each reported hit's measured m/z matches the database metabolite mass transformed by the assigned adduct within the ppm.db tolerance (consistent with the ~6.5 ppm scale used in basepeak_finder)
- The set of annotated base-peak m/z values corresponds to features present in the s2 basepeak_finder result object
- Installation and behavior match the package's documented API (check help(function) usage, since geoRge 1.0 reduced arguments and added documentation)

## Limitations

- Annotation is restricted to metabolites present in the supplied database and to the adducts in the given adduct table (here the negative-mode list), so true identities outside that space are missed
- Quality depends on upstream thresholds: basepeak_finder's Basepeak.minInt=2000 and PuInc_seeker parameters (e.g. PuInc.int.lim=4000, fc.threshold=1.5, p.value.threshold=.05 in the README example) determine which labeled features survive for annotation
- The geoRge API changed across versions (geoRge 1.0 uses 'same functions, less arguments' and label_compare() was patched), so reconstructed signatures must be checked against the installed version's help pages

## Evidence

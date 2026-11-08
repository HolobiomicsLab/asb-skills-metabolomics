---
name: untargeted-screening-identification-tallying
description: Use when a completed MS-DIAL/MS-FINDER screening yields a list of identified
  features whose library annotations and unknown IDs must be counted, cross-checked
  against a reported total (e.g. 83 compounds), and exported as a per-identification-status
  table.
when_to_use_negative:
- When the identification results are still at the feature level (no library annotation
  or in-silico assignment has been made) — tallying is premature until MS-DIAL matching
  and MS-FINDER structural elucidation have produced status labels.
- When the goal is quantification or risk ranking of detected migrants (e.g. octocrylene
  or 2-ethylhexyl-4-methoxycinnamate levels) rather than simply counting identifications
  — use the pseudo-MRM targeting or quantification workflow instead.
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-DIAL
  role: performs library matching against the in-house MS/MS library to produce the
    annotated-compound list that is tallied
  repo: https://github.com/QizhiSu/mspcompiler
- name: mspcompiler
  role: R package used to compile the in-house MS/MS libraries that enable library
    annotation, so the annotated subset can be counted
  repo: https://github.com/QizhiSu/mspcompiler
- name: MS-FINDER
  role: in-silico fragmentation tool that assigns structures to the remaining unknowns,
    which are then counted as the unknown tally
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:9d71dfada39bd5a49670680198e411aee1a4527ad5065526c65208e05e61f9a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - MS-CleanR
  - MS-DIAL
  - MS-FINDER
  - mspcompiler
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# untargeted-screening-identification-tallying

## Summary

Aggregate the feature-level identification results of an untargeted LC-HRMS screening (e.g. the rHDPE milk-bottle study) into a structured summary table that tallies total identified compounds and partitions them into library-annotated versus in-silico-only (unknown) identifications.

## When to use

Use when a completed MS-DIAL/MS-FINDER screening yields a list of identified features whose library annotations and unknown IDs must be counted, cross-checked against a reported total (e.g. 83 compounds), and exported as a per-identification-status table.

## When NOT to use

- When the identification results are still at the feature level (no library annotation or in-silico assignment has been made) — tallying is premature until MS-DIAL matching and MS-FINDER structural elucidation have produced status labels.
- When the goal is quantification or risk ranking of detected migrants (e.g. octocrylene or 2-ethylhexyl-4-methoxycinnamate levels) rather than simply counting identifications — use the pseudo-MRM targeting or quantification workflow instead.

## Inputs

- identification results table from MS-DIAL/MS-FINDER (e.g. Table 1 of the article: 66 library-annotated compounds)
- list of unknown compounds kept for further exploration (13 compounds in Appendix C)
- article abstract or results text stating the total number of identified compounds (83)

## Outputs

- identification_tallies.csv — a structured table with columns: identification status (library-annotated / unknown), compound name, and tally counts
- tally numbers: total identified compounds, number annotated by in-house MS/MS library (66), number of unknowns (13)

## How to apply

Retrieve the identification results table (in the rHDPE study, Table 1 lists 66 compounds annotated by matching against the in-house MS/MS library built with mspcompiler) and separately collect the unknown compounds kept for further exploration (13 compounds whose spectra are stored in Appendix C of the article). Combine both lists into a single results table labelled with identification status (library-annotated vs unknown) and compute the tallies: total = annotated + unknown. Cross-check the total against the article's stated number of identified compounds (83) and against the abstract's statement that 66 were annotated by the in-house MS/MS libraries and mspcompiler. Save the structured summary as identification_tallies.csv so the count can be reused for reporting or comparison across samples.

## Related tools

- **MS-DIAL** (performs library matching against the in-house MS/MS library to produce the annotated-compound list that is tallied) — https://github.com/QizhiSu/mspcompiler
- **mspcompiler** (R package used to compile the in-house MS/MS libraries that enable library annotation, so the annotated subset can be counted) — https://github.com/QizhiSu/mspcompiler
- **MS-FINDER** (in-silico fragmentation tool that assigns structures to the remaining unknowns, which are then counted as the unknown tally)

## Examples

```
read_tallies <- function(t1, unk) { annotated <- read.csv(t1); unknowns <- read.csv(unk); out <- rbind(cbind(status="library-annotated", annotated), cbind(status="unknown", unknowns)); write.csv(out, "identification_tallies.csv", row.names=FALSE); c(total=nrow(out), annotated=nrow(annotated), unknown=nrow(unknowns)) }; read_tallies("table1_annotated.csv", "appendixC_unknowns.csv")
```

## Evaluation signals

- The summed tally (library-annotated + unknown) must equal the reported total of 83 compounds.
- The library-annotated count must equal 66, matching the abstract's statement about the in-house MS/MS libraries and mspcompiler.
- The unknown count must equal 83 − 66 = 13, matching the compounds kept in Appendix C for further exploration.
- The output CSV must contain a status column that distinguishes every compound as either 'library-annotated' or 'unknown' with no third category.
- No compound should be double-counted across the two status groups.

## Limitations

- The tallies are specific to this screening configuration (rHDPE milk bottles, LC-QTOF-MS, in-house MS/MS library of >500 plastic-related chemicals); other sample matrices or libraries will yield different counts.
- Counts reflect identifications only — compounds detected but not identified are excluded, so the tally under-represents total chemical burden in the sample.
- Unknowns counted here are 'known unknowns' — the article notes the strategy only works for compounds present in existing structure databases (cppDB, volDB, genDB); 'unknown unknowns' absent from all databases cannot be tallied by this approach.

## Evidence

- [abstract] A total of 83 compounds were identified, with 66 easily annotated by making use of our in-house MS/MS libraries and the mspcompiler R package.: "A total of 83 compounds were identified, with 66 easily annotated by making use of our in-house MS/MS libraries and the mspcompiler R package."
- [abstract] In silico fragmentation combined with data obtained from gas chromatography–mass spectrometry and lists of chemicals related to plastics were used to identify those remaining unknown.: "In silico fragmentation combined with data obtained from gas chromatography–mass spectrometry and lists of chemicals related to plastics were used to identify those remaining unknown."
- [methods] For subsequent structural elucidation in MS-FINDER, both the most intense and most connected features were kept.: [evidence span withheld — anchor status: not_found]
- [discussion] employed as a structure database for the in silico fragmentation tool (MS-FINDER) and automatically found correspondences for some unknowns: "employed as a structure database for the in silico fragmentation tool (MS-FINDER) and automatically found correspondences for some unknowns"
- [readme] The goal of mspcompiler is to offer ways to compile either EI or tandem mass spectral libraries from various sources: "The goal of mspcompiler is to offer ways to compile either EI or tandem mass spectral libraries from various sources"

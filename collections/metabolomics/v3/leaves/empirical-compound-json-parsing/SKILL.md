---
name: empirical-compound-json-parsing
description: Use when a khipu preannotation run (JSON output over an LC-MS feature
  table like ecoli_pos.tsv) must be interpreted per compound — extracting isotope
  branches per adduct, inferring which isotopologue envelope a compound shows, or
  matching an inferred neutral mass to a candidate metabolite.
when_to_use_negative:
- When the input is already a tab-delimited feature table that has not been run through
  khipu preannotation — this skill consumes khipu's JSON output, not raw intensities.
- When complete, library-backed annotation with authentic standards, contaminants,
  or tandem MS data is required — khipu preannotation is explicitly a precursor step
  to those resources.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: khipu
  role: produces the JSON empirical-compound trees that this skill parses; run as
    `khipu -i testdata/ecoli_pos.tsv -o this_test`
  repo: https://github.com/shuzhao-li/khipu
- name: Python 3
  role: scripting environment used to parse and tabulate the isotope-index distribution
    from the JSON records
- name: Jupyter
  role: notebook environment for interactive inspection of the parsed empirical-compound
    trees
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:7a521f5e81a2c24027822ced7541e8023d55e0f57777391683572d05b426ce2f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Jupyter
  - Python 3
  - khipu
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# empirical-compound-json-parsing

## Summary

Parses khipu's transparent JSON empirical-compound records to extract per-tree adduct trunks and isotope branches, enabling isotope-index extraction (e.g. 13C/12C*9) from stable isotope tracing data such as the U-13C-labeled E. coli feature table.

## When to use

Use when a khipu preannotation run (JSON output over an LC-MS feature table like ecoli_pos.tsv) must be interpreted per compound — extracting isotope branches per adduct, inferring which isotopologue envelope a compound shows, or matching an inferred neutral mass to a candidate metabolite.

## When NOT to use

- When the input is already a tab-delimited feature table that has not been run through khipu preannotation — this skill consumes khipu's JSON output, not raw intensities.
- When complete, library-backed annotation with authentic standards, contaminants, or tandem MS data is required — khipu preannotation is explicitly a precursor step to those resources.

## Inputs

- khipu JSON empirical-compound annotation file (e.g. this_test.json)
- LC-MS feature table used as khipu input (tab-delimited, columns feature ID, m/z, rtime, intensities)
- isotope pattern list (13C/12C, 13C/12C*2, … at 1.0034 Da spacing)
- adduct pattern list (M+H[+], M+NH4[+], M+Na[+], M+HCl+H[+], M+K[+], M+ACN+H[+])

## Outputs

- per-compound isotope-index distribution table (e.g. isotope_index_distribution.csv with one row per empirical compound and per-condition isotopologue counts)
- selected empirical-compound tree JSON record (e.g. acetylspermidine_khipu_tree.json) for the compound of interest
- inferred neutral mass of the M0 compound per tree
- adduct trunk and isotope-branch assignments per tree

## How to apply

Load the JSON file emitted by khipu (e.g. this_test.json from `khipu -i testdata/ecoli_pos.tsv -o this_test`), which contains one empirical-compound tree per record. Each tree's isotope edges (1.0034 Da 13C/12C spacing) form groups by shared nodes, each group belonging to one adduct type on the trunk; walk these groups as 'branches' attached to the adduct trunk. Re-align the branches to the khipu grid (M0, 13C/12C, 13C/12C*2, … columns vs adduct rows like M+H[+], M+Na[+], M+K[+]) to assign an isotope index per compound. The neutral mass of the M0 compound is inferred by a regression model over the matched m/z values, so select the tree whose inferred mass matches the candidate of interest (e.g. 187.1686 for acetylspermidine, C9H21N3O). Cross-check isotopologue intensities across conditions — unlabeled samples should peak at M0 while U-13C-labeled samples peak at the isotope index equal to the carbon count (M9 for a 9-carbon compound).

## Related tools

- **khipu** (produces the JSON empirical-compound trees that this skill parses; run as `khipu -i testdata/ecoli_pos.tsv -o this_test`) — https://github.com/shuzhao-li/khipu
- **Python 3** (scripting environment used to parse and tabulate the isotope-index distribution from the JSON records)
- **Jupyter** (notebook environment for interactive inspection of the parsed empirical-compound trees)

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test  # then parse this_test.json for isotope branches per tree
```

## Evaluation signals

- Output table contains exactly 548 compound rows for the ecoli_pos.tsv dataset (548 empirical compounds annotated from 1,745 of 3,602 features).
- Multi-isotope count matches the reported 445 compounds having more than one isotopologue.
- For the tree with inferred neutral mass 187.1686, the U-13C-labeled condition's isotopologue envelope peaks at 13C/12C*9 (M9), consistent with a 9-carbon compound.
- Each isotope group in a parsed tree belongs to exactly one adduct type on the trunk, matching khipu's branch/trunk organization.
- Inferred neutral mass values are reproducible from regression over the matched m/z values (e.g. 187.1686 for the acetylspermidine tree).

## Limitations

- Complete annotation still requires additional resources — contaminants, authentic libraries, and tandem mass spectrometry data — beyond khipu preannotation.
- The isotope search pattern is often dependent on the biochemical experiment; users must overwrite the default by supplying their own search patterns when the labeling scheme differs.
- Some ions may enter the initial network by mistake or as unresolved signals and are removed from the established khipu and sent off to form a new khipu, so parsed trees are not exhaustive over the original feature table.
- Annotation of untargeted metabolomics data, including isotopic tracing data, is still not fully solved.

## Evidence

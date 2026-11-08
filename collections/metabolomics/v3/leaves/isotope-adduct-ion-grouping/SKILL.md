---
name: isotope-adduct-ion-grouping
description: Use when you have a preprocessed LC-MS feature table (feature ID, m/z,
  retention time, sample intensities) from unlabeled or isotope-labeled experiments
  and need to group related ions into empirical compounds and infer neutral masses
  before formula or metabolite-level annotation.
when_to_use_negative:
- When peaks have already been assigned to known metabolites or formulas and only
  fine-level curation (contaminants, authentic libraries, tandem MS) is needed — khipu
  is a low-level preannotation step, not a complete annotation.
- When the input is not a feature table but raw vendor/centroided MS data — khipu
  expects preprocessed features from a tool like asari.
- When the experiment uses isotope or derivatization patterns outside the default
  grid and those patterns have not yet been configured via the user-supplied search
  patterns.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: khipu
  role: Implements the Weavor/Khipu algorithms that convert mass distance networks
    into empirical compound trees and infer neutral masses; usable as pip-installable
    package, CLI, or Python library.
  repo: https://github.com/shuzhao-li/khipu
- name: asari
  role: Upstream preprocessing tool that generates the feature tables khipu consumes
    (used here at version 1.9.2) and uses khipu for its own preannotation.
  repo: https://github.com/shuzhao-li/asari
- name: mass2chem
  role: Provides the search functions khipu uses to match mass difference patterns
    between features.
  repo: https://github.com/shuzhao-li-lab/mass2chem
- name: metDataModel
  role: Defines the 'empirical compound' data model that khipu outputs.
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:2eaa59d1b630a55612ec74b5a8f481054376da3e6025969d2f1138031bf49275
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Python 3
  - asari
  - khipu
  - mass2chem
  - metDataModel
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# isotope-adduct-ion-grouping

## Summary

Groups LC-MS features into empirical compound 'khipu' trees by connecting isotope (13C/12C) and adduct (M+H, M+Na, M+K, M+NH4) edges in a mass distance network, inferring each tree's M0 neutral mass by regression — enabling preannotation of untargeted metabolomics and stable isotope tracing data without known formulas.

## When to use

Use when you have a preprocessed LC-MS feature table (feature ID, m/z, retention time, sample intensities) from unlabeled or isotope-labeled experiments and need to group related ions into empirical compounds and infer neutral masses before formula or metabolite-level annotation.

## When NOT to use

- When peaks have already been assigned to known metabolites or formulas and only fine-level curation (contaminants, authentic libraries, tandem MS) is needed — khipu is a low-level preannotation step, not a complete annotation.
- When the input is not a feature table but raw vendor/centroided MS data — khipu expects preprocessed features from a tool like asari.
- When the experiment uses isotope or derivatization patterns outside the default grid and those patterns have not yet been configured via the user-supplied search patterns.

## Inputs

- Tab-delimited LC-MS feature table (columns: feature ID, m/z, rtime, followed by intensities; e.g. testdata/ecoli_pos.tsv or yeast_neg.tsv from asari 1.9.2)
- Ionization mode (pos or neg)
- Mass tolerance in ppm (e.g. default --ppm)
- Retention-time tolerance (--rtol, unit depends on preprocessing tool)
- Optional: user-supplied isotope/adduct search patterns for tracing or derivatization experiments

## Outputs

- JSON file of empirical compounds (empCpds) with inferred neutral mass and adduct/isotope tree structure
- Tab-delimited (TSV) table of empirical compounds
- Optional: plain-text tree/grid visualizations or richer plots when loaded in Jupyter

## How to apply

Start with a tab-delimited feature table such as testdata/ecoli_pos.tsv from asari 1.9.2. Run khipu (installed via pip install khipu-metabolomics) with an ionization mode and ppm mass tolerance, or use the Python library/Jupyter interface for finer control. Khipu searches the feature list for pairs matching initial isotope patterns (e.g. 1.0034 Da for 13C/12C in successive M0..M6 steps) and adduct patterns (e.g. 1.007276 for M+H, 22.989276 for M+Na, 38.963158 for M+K), trims adduct patterns to reduce ambiguity, and connects all matched pairs into a mass distance network. Each connected subnetwork is converted into a directed acyclic graph, redundant nodes removed, and aligned to a khipu grid tree where isotopic branches attach to an adduct trunk; the M0 neutral mass is then inferred by linear regression from the matched m/z values. Output is JSON and TSV files of empirical compounds (empCpds), which can be parsed to inspect tree structure, adduct assignments, and isotope series (e.g. a tree peaking at 13C/12C*9 in a U-13C sample implies ~9 carbons).

## Related tools

- **khipu** (Implements the Weavor/Khipu algorithms that convert mass distance networks into empirical compound trees and infer neutral masses; usable as pip-installable package, CLI, or Python library.) — https://github.com/shuzhao-li/khipu
- **asari** (Upstream preprocessing tool that generates the feature tables khipu consumes (used here at version 1.9.2) and uses khipu for its own preannotation.) — https://github.com/shuzhao-li/asari
- **mass2chem** (Provides the search functions khipu uses to match mass difference patterns between features.) — https://github.com/shuzhao-li-lab/mass2chem
- **metDataModel** (Defines the 'empirical compound' data model that khipu outputs.) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test --mode pos --ppm 5
```

## Evaluation signals

- Tree's inferred neutral mass matches expected compound mass — e.g. the E. coli U-13C tree with inferred mass 187.1686 corresponds to acetylspermidine (C9H21N3O, 9 carbons).
- Tree structure must satisfy DAG and rooted-tree constraints; erroneous edges that violate this are weeded out during conversion.
- Number of features explained by empirical compounds is a quality metric — on the E. coli RP ESI+ data khipu annotated 548 empirical compounds from 1,745 of 3,602 features; on the two yeast data sets it yielded 1,775 and 908 empCpds.
- Adding extra adducts should increase explained features (yeast ESI+ went from 6,310 to 8,049; ESI- from 2,601 to 2,912 when additional adducts were included).
- Isotope-labeled samples should show shifted isotope-peak intensity maxima matching the labeled atom count (e.g. highest isotope peak at M9 for a 9-carbon compound in U-13C samples).

## Limitations

- Khipu is preannotation; complete annotation still requires contaminants, authentic libraries, and tandem MS data.
- Isotopic tracing annotation is still not fully solved; future benchmarking and improvements are expected.
- Existing network-approach tools (including khipu's context) depend on assumed base ions or formulas, so incorrect or missing adduct patterns can produce incomplete or misassigned trees.
- Only less than ~2,000 compounds were reliably measured in the examined experiments, so khipu's output does not imply that many features are truly detectable compounds.
- Erroneous edges or unresolved signals can enter the initial network; khipu removes them and re-forms new khipus, but heavily contaminated or noisy feature tables may degrade grouping.

## Evidence

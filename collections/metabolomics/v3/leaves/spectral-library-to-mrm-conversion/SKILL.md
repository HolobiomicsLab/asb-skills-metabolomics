---
name: spectral-library-to-mrm-conversion
description: Use when compounds have already been identified in untargeted LC-HRMS/DIA
  screening and you need to target them sensitively using a pseudo-MRM transition
  list derived from library MS/MS spectra.
when_to_use_negative:
- When compounds have not yet been identified in the untargeted workflow - the pseudo-MRM
  transitions can only be built for chemicals already identified with the in-house
  MS/MS libraries.
- When a true triple quadrupole MRM acquisition with a genuine MRM method is planned
  - this skill constructs a pseudo-MRM table applied to existing DIA data, not a scheduled
  MRM acquisition method.
- When library entries lack fragment/product ion records, since no precursor-product
  transition pairs (Q1->Q3) can be formed.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mspcompiler
  role: Produces the verified, curated in-house MS/MS library msp whose compound entries
    supply precursor m/z and product ion lists for transition generation
  repo: https://github.com/QizhiSu/mspcompiler
- name: MRMPROBS
  role: Consumes the pseudo-MRM transition table and performs targeted screening/quantification
    of the identified chemicals on the DIA data with automated posterior probabilistic
    data assessment
  repo: https://zenodo.org/record/4454648
- name: MS-DIAL
  role: Processed the original untargeted DIA data (version 4.38) whose identified
    chemicals feed the pseudo-MRM targeting step
- name: R
  role: Implementation language for mspcompiler and the transition-list conversion
    workflow
  repo: https://github.com/QizhiSu/mspcompiler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c05389
    title: Combination of Structure Databases, In Silico Fragmentation, and MS/MS
      Libraries for Untargeted Screening of Non-Volatile Migrants from Recycled High-Density
      Polyethylene Milk Bottles
  source_content_hash: sha256:69efdec7d1348c49b7c7ed5b183002a9aa813adff48c03d88e3829b41a949fb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QizhiSu/mspcompiler
  tools:
  - MRMPROBS
  - MS-DIAL
  - R
  - mspcompiler
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spectral-library-to-mrm-conversion

## Summary

Converts a curated in-house MS/MS library into a pseudo-MRM (MRMPROBS) transition table by ranking product ions and keeping the top 5 per compound, enabling sensitive targeted screening of previously identified migrants on existing DIA UPLC-QTOF-MS data without re-acquisition.

## When to use

Use when compounds have already been identified in untargeted LC-HRMS/DIA screening and you need to target them sensitively using a pseudo-MRM transition list derived from library MS/MS spectra.

## When NOT to use

- When compounds have not yet been identified in the untargeted workflow - the pseudo-MRM transitions can only be built for chemicals already identified with the in-house MS/MS libraries.
- When a true triple quadrupole MRM acquisition with a genuine MRM method is planned - this skill constructs a pseudo-MRM table applied to existing DIA data, not a scheduled MRM acquisition method.
- When library entries lack fragment/product ion records, since no precursor-product transition pairs (Q1->Q3) can be formed.

## Inputs

- Curated in-house MS/MS library in msp format (positive and negative polarity, e.g. 449 pos and 172 neg plastic-related chemical entries) from mspcompiler
- DIA UPLC-QTOF-MS data sets of the recycled HDPE samples previously used for untargeted screening

## Outputs

- MRMPROBS-loadable pseudo-MRM transition table (precursor m/z, product ion m/z, retention time, polarity) with top-5 product ions per compound
- MRMPROBS targeted screening/quantification results for the identified chemicals in rHDPE samples

## How to apply

Load the verified msp output of the in-house MS/MS library (449 positive-mode and 172 negative-mode plastic-related chemicals) produced by mspcompiler. For each compound entry, parse the precursor ion m/z and fragment/product ion records, rank product ions by intensity, and keep the top 5 to form precursor-product transition pairs (Q1->Q3). Write the transitions (precursor m/z, product ion m/z, retention time, polarity) into a tab/CSV table loadable by MRMPROBS as a pseudo-MRM library, then run MRMPROBS, an open-source targeted-metabolomics tool providing automated posterior probabilistic data assessment, on the original DIA data set. Rationale: the pseudo-MRM step is applied after identification so the identified chemicals can be screened sensitively in the samples without acquiring new targeted data.

## Related tools

- **mspcompiler** (Produces the verified, curated in-house MS/MS library msp whose compound entries supply precursor m/z and product ion lists for transition generation) — https://github.com/QizhiSu/mspcompiler
- **MRMPROBS** (Consumes the pseudo-MRM transition table and performs targeted screening/quantification of the identified chemicals on the DIA data with automated posterior probabilistic data assessment) — https://zenodo.org/record/4454648
- **MS-DIAL** (Processed the original untargeted DIA data (version 4.38) whose identified chemicals feed the pseudo-MRM targeting step)
- **R** (Implementation language for mspcompiler and the transition-list conversion workflow) — https://github.com/QizhiSu/mspcompiler

## Examples

```
Rscript -e "lib<-mspcompiler::read_lib('in_house.msp',type='MS2'); write_MS2_msp(lib,'pseudoMRM.csv')"
```

## Evaluation signals

- The output file exists and is a loadable MRMPROBS transition table (correct columns for precursor m/z, product ion m/z, retention time, polarity).
- Every compound entry with 5 or more fragments in the library contains exactly 5 product ions in the transition list.
- Q1 (precursor m/z) values in the transition table match the precursor ions of the corresponding in-house MS/MS library entries.
- Number of transitions corresponds to the identified chemical set (up to 5 products each, capped by available fragments) derived from the 449 positive and 172 negative library entries.
- MRMPROBS runs on the DIA data set with the imported table and yields targeted results for the identified chemicals, enabling quantification of migrants such as octocrylene and 2-ethylhexyl-4-methoxycinnamate.

## Limitations

- The strategy only works for known unknowns already represented in existing structure/MS/MS libraries; unknown unknowns absent from all databases cannot be targeted this way.
- At most 5 product ions per compound are kept; compounds with fewer than 5 fragments yield correspondingly fewer transitions.
- Pseudo-MRM on DIA data is not equivalent to a real triple quadrupole MRM acquisition; sensitivity gains depend on the posterior probabilistic assessment in MRMPROBS rather than dedicated MRM sensitivity.

## Evidence

- [other] Rank product ions by intensity and keep the top 5 per compound to form precursor-product transition pairs (Q1->Q3), matching the MRMPROBS targeted-metabolomics library format.: [evidence span withheld — anchor status: not_found]
- [abstract] pseudo-MRM applied after identification to sensitively target identified chemicals: "A pseudo-multiple reaction monitoring method was also applied to sensitively target and screen the identified chemicals in the samples."
- [methods] MRMPROBS as open-source targeted metabolomics software: "MRMPROBS
is an open-source software designed for targeted metabolomics, providing
automated posterior probabilistic data assessment"
- [readme] mspcompiler organizes libraries into a neat msp file for MS-DIAL: [evidence span withheld — anchor status: not_found]
- [discussion] high-molecular weight substances remain obstacles for food contact rHDPE: "high-molecular weight substances such as octocrylene are still obstacles for the food contact applications of rHDPE as they are not easy to remove"

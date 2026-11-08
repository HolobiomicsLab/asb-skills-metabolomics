---
name: structured-findings-tabulation
description: Use when a review article reports multiple named empirical benchmark
  results with distinct tool-condition pairings (e.g. DeepRescore, inSPIRE, MHCquant,
  MSBooster, Prosit, Percolator) and they must be captured as a machine-readable findings
  table.
when_to_use_negative:
- The article reports a single integrated method rather than several independently
  attributable benchmark results — there is no tool-condition contrast to tabulate
- The tool attribution is already provided in a structured table or supplementary
  file; re-parsing prose would only introduce transcription errors
- The goal is reproducible re-execution of the benchmarks rather than tabulating the
  review's reported claims
edam_operation: http://edamontology.org/operation_0306
tools:
- name: DeepRescore
  role: Named tool whose delta-RT plus spectral-angle benchmark result in DDA immunopeptidomics
    becomes one tabulated record
- name: inSPIRE
  role: Named tool whose binding-affinity/spectral-feature concert benchmark result
    becomes one tabulated record
- name: MHCquant
  role: Named tool whose delta-RT ineffectiveness finding (attributed to the outdated
    RTPredict retention time predictor) becomes one tabulated record
- name: MSBooster
  role: Named tool whose marginal ion-mobility benefit benchmark result becomes one
    tabulated record
- name: Percolator
  role: Named tool whose numProt bias benchmark result becomes one tabulated record
- name: Prosit
  role: Named tool whose timsTOF-finetuned and RNN-vs-transformer benchmark results
    become two tabulated records
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:493d66214d8fffca29ae8c97879436821561bc0c9301af1fc13e124027eae2db
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  tools:
  - DeepRescore
  - MHCquant
  - MS2Rescore
  - MSBooster
  - Percolator
  - Prosit
  - inSPIRE
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# structured-findings-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses the named empirical benchmark results of a review article (e.g. the 7 PSM-rescoring findings in DOI 10.1002/pmic.202300336) into structured {finding, tool, condition} records, assembles them into a single table, serializes it to JSON, and validates the output schema and record count.

## When to use

Use when a review article reports multiple named empirical benchmark results with distinct tool-condition pairings (e.g. DeepRescore, inSPIRE, MHCquant, MSBooster, Prosit, Percolator) and they must be captured as a machine-readable findings table.

## When NOT to use

- The article reports a single integrated method rather than several independently attributable benchmark results — there is no tool-condition contrast to tabulate
- The tool attribution is already provided in a structured table or supplementary file; re-parsing prose would only introduce transcription errors
- The goal is reproducible re-execution of the benchmarks rather than tabulating the review's reported claims

## Inputs

- Full text of a review article (DOI 10.1002/pmic.202300336) reporting multiple empirical benchmark results
- Named tool list extracted from the article (Percolator, Prosit, DeepRescore, inSPIRE, MHCquant, MSBooster, MS2Rescore)
- Research question framing the comparison (e.g. why MHCquant and DeepRescore reached contradictory ΔRT conclusions)

## Outputs

- findings_table.json — a JSON array of exactly 7 records, each with non-empty 'finding', 'tool', and 'condition' fields
- Expert-verified tool-condition mapping for each of the 7 benchmark results

## How to apply

Load the full text of the source review (here 'Machine learning-based peptide-spectrum match rescoring opens up the immunopeptidome', DOI 10.1002/pmic.202300336) and locate each section reporting an empirical benchmark result: DeepRescore's delta-RT plus spectral-angle dominance in DDA immunopeptidomics, inSPIRE's binding-affinity/spectral-feature concert, MHCquant's delta-RT ineffectiveness via the outdated RTPredict predictor, MSBooster's marginal ion-mobility benefit, the numProt bias in Percolator, Prosit finetuned for timsTOF, and the transformer-with-pretraining outperforming Prosit's RNN. For each result, decompose it into three structured fields: the reported result ('finding'), the named tool ('tool'), and the experimental condition such as acquisition type, predictor architecture, or feature set ('condition'). Assemble all records into one table with exactly those three fields, serialize to findings_table.json, then validate that the file exists, parses as JSON, contains exactly 7 records, and that every record has non-empty finding/tool/condition values. Finally, have each tool-condition mapping checked by expert review against the article text, since the attribution logic (e.g. MHCquant vs DeepRescore on ΔRT hinges on the >15-year-old RTPredict predictor) is the analytically fragile part of the record.

## Related tools

- **DeepRescore** (Named tool whose delta-RT plus spectral-angle benchmark result in DDA immunopeptidomics becomes one tabulated record)
- **inSPIRE** (Named tool whose binding-affinity/spectral-feature concert benchmark result becomes one tabulated record)
- **MHCquant** (Named tool whose delta-RT ineffectiveness finding (attributed to the outdated RTPredict retention time predictor) becomes one tabulated record)
- **MSBooster** (Named tool whose marginal ion-mobility benefit benchmark result becomes one tabulated record)
- **Percolator** (Named tool whose numProt bias benchmark result becomes one tabulated record)
- **Prosit** (Named tool whose timsTOF-finetuned and RNN-vs-transformer benchmark results become two tabulated records)

## Examples

```
import json; records = json.load(open('findings_table.json')); assert len(records) == 7 and all(r['finding'] and r['tool'] and r['condition'] for r in records)
```

## Evaluation signals

- findings_table.json exists and parses as valid JSON
- The parsed table contains exactly 7 records — one per empirical benchmark result reported in the article
- Every record has non-empty 'finding', 'tool', and 'condition' fields
- Each tool-condition mapping (e.g. MHCquant ↔ outdated RTPredict predictor; Prosit ↔ timsTOF finetuning) is confirmed by expert review against the article text
- The contradictory ΔRT conclusions of MHCquant and DeepRescore are represented as separate records with distinct conditions rather than merged

## Limitations

- The table captures reported claims, not reproduced results — e.g. the MHCquant/DeepRescore ΔRT contradiction is attributed to prediction-model accuracy (RTPredict is more than 15 years old), so tool-condition mappings may shift if underlying models are updated
- Tool-condition attribution requires expert review against article text; automated parsing alone cannot guarantee correct pairing
- Some sections of the source repository documentation are changelog text without scientific discussion, so evidence for tool behavior may be incomplete in the discussion section

## Evidence

- [abstract] PSM rescoring as machine learning solution: [evidence span withheld — anchor status: not_found]
- [readme] MS2Rescore scope statement: [evidence span withheld — anchor status: external_doc]

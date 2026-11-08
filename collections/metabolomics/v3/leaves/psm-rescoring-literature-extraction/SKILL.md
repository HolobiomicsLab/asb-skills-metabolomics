---
name: psm-rescoring-literature-extraction
description: Use when a review or benchmark article reports multiple tool-specific
  PSM rescoring results (DDA/timsTOF conditions, predictor architectures, feature
  sets) that must be parsed into a comparable finding/tool/condition table.
when_to_use_negative:
- The goal is to actually run PSM rescoring on spectra (use MS2Rescore/Percolator
  directly) rather than extract findings from literature
- The input document is a software changelog or README with no scientific discussion
  or evaluation results (as noted in the missing-signals section of this source)
- The findings are already available as a structured table — extraction would duplicate
  existing work
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: DeepRescore
  role: Benchmark tool whose ΔRT-plus-spectral-angle result is extracted as one of
    the 7 findings
- name: inSPIRE
  role: Benchmark tool whose binding-affinity/spectral-feature concert result is extracted
- name: MHCquant
  role: Benchmark tool whose ΔRT ineffectiveness (attributed to the outdated RTPredict
    predictor) is extracted
- name: MSBooster
  role: Benchmark tool whose marginal ion-mobility feature benefit is extracted
- name: Percolator
  role: Benchmark tool whose numProt feature bias is extracted
- name: Prosit
  role: Benchmark predictor whose timsTOF finetuning and RNN-versus-transformer comparison
    are extracted
- name: MS2Rescore
  role: Referenced modular rescoring platform whose configuration (ms2pip, deeplc,
    ristretto) contextualizes the extracted findings
  repo: https://github.com/compomics/ms2rescore
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:338370dd283c2cc01eb7df6029b252c7278f5ea206c816f5274789d88ebebaff
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/compomics/ms2rescore
  tools:
  - DeepRescore
  - MHCquant
  - MS2Rescore
  - MSBooster
  - Percolator
  - Prosit
  - inSPIRE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# psm-rescoring-literature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts empirical PSM rescoring benchmark findings from immunopeptidomics literature into structured records pairing each reported result with its named tool (e.g. DeepRescore, inSPIRE, MHCquant, Prosit) and experimental condition. Enables reconciling apparently contradictory conclusions, such as why MHCquant found ΔRT ineffective while DeepRescore ranked ΔRT among its highest-weight features.

## When to use

Use when a review or benchmark article reports multiple tool-specific PSM rescoring results (DDA/timsTOF conditions, predictor architectures, feature sets) that must be parsed into a comparable finding/tool/condition table.

## When NOT to use

- The goal is to actually run PSM rescoring on spectra (use MS2Rescore/Percolator directly) rather than extract findings from literature
- The input document is a software changelog or README with no scientific discussion or evaluation results (as noted in the missing-signals section of this source)
- The findings are already available as a structured table — extraction would duplicate existing work

## Inputs

- Full text of a peer-reviewed PSM rescoring review or benchmark article (e.g. DOI 10.1002/pmic.202300336)
- Enriched article index listing tools, workflow steps, filter steps, and traced findings with section-level evidence

## Outputs

- findings_table.json containing exactly 7 records, each with non-empty 'finding', 'tool', and 'condition' fields
- Structured record per benchmark result linking reported result, tool name, and experimental condition

## How to apply

Load the full text of the source article (here DOI 10.1002/pmic.202300336) and locate each empirically benchmarked rescoring result, such as DeepRescore's ΔRT plus spectral-angle dominance in DDA immunopeptidomics, inSPIRE's binding-affinity/spectral-feature concert, MHCquant's ΔRT ineffectiveness, MSBooster's marginal ion-mobility benefit, numProt bias in Percolator, Prosit finetuned for timsTOF, and transformer-with-pretraining outperforming Prosit's RNN. For each finding, parse out the reported result, the named tool, and the experimental condition (acquisition type, predictor architecture, feature set) into a structured record with fields 'finding', 'tool', and 'condition'. Assemble the records into a single table and serialize it to findings_table.json. Validate that the file exists, parses as JSON, contains exactly 7 records with non-empty fields, and have an expert review the tool-condition mapping against the article text. Rationale: accurate predicted peptidoform properties (fragment ion intensities, retention time, collisional cross section) determine feature informativeness, so the condition field — e.g. MHCquant's reliance on the outdated RTPredict retention-time predictor versus DeepRescore's high-accuracy ΔRT features — is essential to explaining contradictory conclusions.

## Related tools

- **DeepRescore** (Benchmark tool whose ΔRT-plus-spectral-angle result is extracted as one of the 7 findings)
- **inSPIRE** (Benchmark tool whose binding-affinity/spectral-feature concert result is extracted)
- **MHCquant** (Benchmark tool whose ΔRT ineffectiveness (attributed to the outdated RTPredict predictor) is extracted)
- **MSBooster** (Benchmark tool whose marginal ion-mobility feature benefit is extracted)
- **Percolator** (Benchmark tool whose numProt feature bias is extracted)
- **Prosit** (Benchmark predictor whose timsTOF finetuning and RNN-versus-transformer comparison are extracted)
- **MS2Rescore** (Referenced modular rescoring platform whose configuration (ms2pip, deeplc, ristretto) contextualizes the extracted findings) — https://github.com/compomics/ms2rescore

## Examples

```
import json; records = [{"finding": f, "tool": t, "condition": c} for f, t, c in parsed_findings]; json.dump(records, open("findings_table.json", "w"))
```

## Evaluation signals

- findings_table.json exists and parses as valid JSON
- The table contains exactly 7 records, one per empirical benchmark result reported in the article
- Every record has non-empty 'finding', 'tool', and 'condition' fields
- The tool-condition mapping for each of the 7 results is verified by expert review against the article text
- Extracted conditions capture the distinguishing variables (acquisition type, predictor architecture, feature set) that explain contradictory conclusions such as ΔRT effectiveness

## Limitations

- Predictor accuracy drives feature informativeness: MHCquant's reliance on the outdated RTPredict retention-time predictor (published more than 15 years ago) explains its ΔRT conclusion, so tool comparisons are only valid per-condition
- Contradictory conclusions across tools (e.g. ΔRT effectiveness) cannot be merged naively without recording the underlying prediction-model differences
- Section-level evidence must be preserved during extraction; the source notes that repository documentation may be a changelog with no methods discussion, limiting extractable context

## Evidence

- [abstract] PSM rescoring is a machine learning-based post-processing approach: [evidence span withheld — anchor status: not_found]
- [readme] ristretto rescoring input schema: [evidence span withheld — anchor status: external_doc]

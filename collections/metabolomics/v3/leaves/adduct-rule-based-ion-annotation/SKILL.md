---
name: adduct-rule-based-ion-annotation
description: Use when you have CAMERA pseudo-spectra from LC/MS data in positive or
  negative ion mode and need to label peaks as isotope, adduct or fragment species
  and derive the quasi-molecular ion mass.
when_to_use_negative:
- Data consist of raw centroided spectra or a flat XCMS peak table with no retention-time/intensity
  grouping — grouping (groupFWHM, groupCorr) must precede adduct annotation.
- The compound spectrum contains only the molecular ion and fragment ions with no
  further adducts — in this case the compound cannot be annotated directly unless
  the neutral loss is added to the rule set.
- Cross-polarity rules are being applied to data acquired in only one ion mode; the
  cross-polarity rule table can override single-polarity annotations and must not
  be used when the opposite polarity is absent.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: implements the dynamic rule table and the findAdducts step that assigns isotope/adduct/fragment
    species and proposes accurate compound masses
  repo: https://travis-ci.org/sneumann/CAMERA
- name: XCMS
  role: produces the peak list of detected features that feeds the xsAnnotate object
    prior to adduct annotation
- name: R
  role: runtime environment; CAMERA is an R/Bioconductor package
  repo: https://bioconductor.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:d54ce9708cd349bb91dc4974339981a3d633b25f24cc7057f178aba72a840191
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CAMERA
  - R
  - XCMS
  - findAdducts
  - findIsotopes
  - getPeaklist
  - groupCorr
  - groupFWHM
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 22
---

# adduct-rule-based-ion-annotation

## Summary

Annotates LC/MS feature clusters into isotope/adduct/fragment ion species by applying a dynamic rule table against pseudo-spectra produced by CAMERA, so that quasi-molecular ion masses can be assigned to each compound spectrum.

## When to use

Use when you have CAMERA pseudo-spectra from LC/MS data in positive or negative ion mode and need to label peaks as isotope, adduct or fragment species and derive the quasi-molecular ion mass.

## When NOT to use

- Data consist of raw centroided spectra or a flat XCMS peak table with no retention-time/intensity grouping — grouping (groupFWHM, groupCorr) must precede adduct annotation.
- The compound spectrum contains only the molecular ion and fragment ions with no further adducts — in this case the compound cannot be annotated directly unless the neutral loss is added to the rule set.
- Cross-polarity rules are being applied to data acquired in only one ion mode; the cross-polarity rule table can override single-polarity annotations and must not be used when the opposite polarity is absent.

## Inputs

- xsAnnotate object (grouped peak list from XCMS/CAMERA) in positive or negative ion mode
- polarity-specific adduct rule table (e.g. [M+H]+, [M+Na]+, [M+NH4]+, [M-H]-, [M+Cl]-)
- optional: cross-polarity data (feature lists from positive and negative ion mode) for annotation verification
- optional: MM39-style reference compound list with sum formulas, molar masses and PubChem IDs for validation

## Outputs

- annotated peak list where each feature carries an isotope/adduct/fragment assignment and a proposed accurate compound mass
- per-compound-spectrum quasi-molecular ion annotation (e.g. [M+H]+ or [M-H]-) enabling elemental composition calculation and tandem-MS follow-up

## How to apply

Take the xsAnnotate object produced by groupFWHM/groupCorr grouping of XCMS peak lists and run findAdducts with the polarity-specific rule set to label each peak as isotope, adduct or fragment and propose the accurate compound mass. The rule table encodes expected mass differences (e.g. [M+H]+ = +1.007825, [M+Na]+ = +22.989218, [M+NH4]+ = +18.034374) and CAMERA scores hypotheses by how many features in the pseudo-spectrum support each rule rather than assuming the most intense peak is the [M+H]+-ion. When data for both positive and negative polarity are available, enable the cross-polarity verification step so annotations confirmed in opposite ion mode are strengthened; the cross-polarity rule table should contain only common and trusted combinations because those rules can override single-polarity annotations. Evaluate by comparing derived masses against a manually defined reference annotation, as done for the MM39 mixture where 89.7% (positive) and 90.3% (negative) of annotatable compounds received the correct molecular mass.

## Related tools

- **CAMERA** (implements the dynamic rule table and the findAdducts step that assigns isotope/adduct/fragment species and proposes accurate compound masses) — https://travis-ci.org/sneumann/CAMERA
- **XCMS** (produces the peak list of detected features that feeds the xsAnnotate object prior to adduct annotation)
- **R** (runtime environment; CAMERA is an R/Bioconductor package) — https://bioconductor.org

## Examples

```
R> runCAMERA(xset, polarity="positive", ruleset=NULL)
```

## Evaluation signals

- Extraction rate of correct molecular masses against a manually defined reference (MM39 benchmark: ≈89.7% positive, ≈90.3% negative ion mode across all spike concentrations).
- Every annotated pseudo-spectrum should carry at least one adduct label with a plausible mass difference; hypotheses with tied precedence scores should be flagged rather than silently chosen.
- Cross-polarity agreement: annotations confirmed in both ion modes should outnumber single-mode-only annotations after the verification step.
- Check that no adduct label is assigned to peaks whose pseudo-spectrum contains too few explained features to support the rule.

## Limitations

- Missing molecular mass annotations occurred because compound spectra contained too few explained features, or several hypotheses had the same precedence scores.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- CAMERA does not use ad-hoc heuristics such as assuming the most intense feature is the [M+H]+-ion; accuracy therefore depends on a well-calibrated rule table.
- Sample concentration must avoid MS detector saturation, which causes m/z and retention time centroid shifts and truncated peak profiles, degrading grouping and adduct annotation quality.

## Evidence

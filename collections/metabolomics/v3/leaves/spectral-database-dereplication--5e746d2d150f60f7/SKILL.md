---
name: spectral-database-dereplication
description: Use when you have centroided LC-MS/MS spectra (MGF/mzXML/mzML/mzData)
  and want to identify known peptidic natural products or other metabolite classes
  by database search with FDR-controlled matches.
when_to_use_negative:
- Spectra are profile-mode or not centroided — NPDtools requires centroided spectra
  in open formats
- Goal is identifying polyketides, terpenes, benzenoids, alkaloids, or flavonoids
  — original DEREPLICATOR is limited to peptidic natural products (PNPs); use DEREPLICATOR+
  instead
- Aim is discovering novel variants rather than dereplicating known compounds — VarQuest
  is the modification-tolerant variant-identification tool
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: DEREPLICATOR
  role: Search spectra against the database of peptidic natural products, scoring
    MSMs by unique matched masses
  repo: https://github.com/ablab/npdtools
- name: DEREPLICATOR+
  role: Extended dereplication against diverse metabolites with fragmentation graphs
    and decoy-based FDR
  repo: https://github.com/ablab/npdtools
- name: ProteoWizard msconvert
  role: Convert spectra in other formats to MGF before running NPDtools pipelines
- name: VarQuest
  role: Modification-tolerant identification of novel variants of PNPs
  repo: https://github.com/ablab/npdtools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-06082-8
    title: Dereplication of microbial metabolites through database search of mass
      spectra
  source_content_hash: sha256:e87c3c8c304b25d78c79a11f6de2b8b48a968bfebd5a88da3abd690d7f0e2612
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/npdtools
  tools:
  - DEREPLICATOR
  - DEREPLICATOR+
  - ProteoWizard msconvert
  - VarQuest
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spectral-database-dereplication

## Summary

Dereplicate natural products by searching LC-MS/MS spectra against databases of known metabolites (peptidic natural products with DEREPLICATOR, diverse metabolites with DEREPLICATOR+), scoring compound–spectrum matches (MSMs) by the number of unique matched masses and controlling FDR via target–decoy.

## When to use

Use when you have centroided LC-MS/MS spectra (MGF/mzXML/mzML/mzData) and want to identify known peptidic natural products or other metabolite classes by database search with FDR-controlled matches.

## When NOT to use

- Spectra are profile-mode or not centroided — NPDtools requires centroided spectra in open formats
- Goal is identifying polyketides, terpenes, benzenoids, alkaloids, or flavonoids — original DEREPLICATOR is limited to peptidic natural products (PNPs); use DEREPLICATOR+ instead
- Aim is discovering novel variants rather than dereplicating known compounds — VarQuest is the modification-tolerant variant-identification tool

## Inputs

- Centroided LC-MS/MS spectra in MGF format (mzXML/mzML/mzData convertible via msconvert)
- Chemical structure database of known natural products (PNP database for DEREPLICATOR)

## Outputs

- Compound–spectrum matches (MSMs) with P-Values and FDR levels
- Counts of unique compounds and MSMs per FDR threshold (e.g., dereplicator_fdr_counts.tsv)

## How to apply

Prepare centroided spectra in MGF format, using msconvert from ProteoWizard for mzXML/mzML/mzData or other formats. Run DEREPLICATOR (NPDtools) to search spectra against the database of peptidic natural products (PNPs), scoring each compound–spectrum match by the number of unique masses in the spectrum that annotate the compound. Compute P-Values for MSMs via the target–decoy strategy (DEREPLICATOR+ generates a decoy fragmentation graph for each target) and evaluate FDR at each P-Value level. Filter MSMs at 0%, 1%, and 3% FDR thresholds and count unique compounds and total MSMs at each level (benchmark: 73 compounds/166 MSMs at 1% FDR, 66/148 at 0% FDR on SpectraActiSeq). Output counts to dereplicator_fdr_counts.tsv and verify reproduction of reported values.

## Related tools

- **DEREPLICATOR** (Search spectra against the database of peptidic natural products, scoring MSMs by unique matched masses) — https://github.com/ablab/npdtools
- **DEREPLICATOR+** (Extended dereplication against diverse metabolites with fragmentation graphs and decoy-based FDR) — https://github.com/ablab/npdtools
- **ProteoWizard msconvert** (Convert spectra in other formats to MGF before running NPDtools pipelines)
- **VarQuest** (Modification-tolerant identification of novel variants of PNPs) — https://github.com/ablab/npdtools

## Examples

```
python bin/dereplicator.py --spectra-files spectra_acti.mgf --pnps-db pnps_db --output dereplicator_fdr_counts.tsv
```

## Evaluation signals

- Reproduced counts match reported values on SpectraActiSeq: 73 compounds (166 MSMs) at 1% FDR and 66 compounds (148 MSMs) at 0% FDR, 75 compounds at 3% FDR
- dereplicator_fdr_counts.tsv is produced with counts at all three thresholds (0%, 1%, 3%)
- Input spectra are centroided and in MGF (or successfully converted via msconvert)
- FDR estimates derive from target–decoy P-Values rather than raw scores
- Successfully installed NPDtools binaries show dereplicator.py, varquest.py, dereplicator+.py and the dereplicate core binary in the bin directory

## Limitations

- Original DEREPLICATOR cannot identify non-peptidic classes such as polyketides and terpenes
- Shared-peaks scoring yields a high false-negative rate; improved scoring not yet implemented
- Vast majority of GNPS spectra remain uninterpreted (dark matter of metabolomics); no version numbers stated for DEREPLICATOR/DEREPLICATOR+/VarQuest/GNPS
- NPDtools requires 64-bit Linux or macOS with Python; joblib is needed for parallel processing of multiple spectra files

## Evidence

- [methods] target-decoy scoring rationale: [evidence span withheld — anchor status: not_found]
- [discussion] MSM scoring definition: "Given a metabolite M and spectrum S, we define score(M,S) as the number of unique masses in S that annotate a node in the fragmentation graph of M."
- [discussion] decoy generation: "DEREPLICATOR+ generates a decoy fragmentation graph for each target fragmentation graph."
- [readme] input format requirements: [evidence span withheld — anchor status: external_doc]
- [intro] DEREPLICATOR scope limitation: "While DEREPLICATOR search of GNPS identified hundreds of peptides and their variants, it is limited to dereplicating PNPs"

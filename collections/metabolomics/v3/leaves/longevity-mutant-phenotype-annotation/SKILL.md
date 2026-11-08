---
name: longevity-mutant-phenotype-annotation
description: Use when MutMap SNP-index analysis of EMS-mutagenised long-lived non-CID
  suppressor strains has produced candidate gene calls and you must assign each causal
  nonsense mutation (e.g. gst-13, mtk-1, nsy-1, smg-1, smg-2, sur-2, rcd-1) to specific
  EMS strain numbers.
when_to_use_negative:
- Mutagenised strains that did not carry nonsense mutations in the QTL — the study
  explicitly did not analyse these further, so attempting gene–strain assignment for
  them is out of scope
- Strains whose phenotype is not a suppressor of hsf-1(sy441) CID (i.e. not long-lived
  non-CID mutants) — the mapping is specific to the CID-suppressor screen
- Assignment of CID-regulator roles requires nonsense-mutation evidence from MutMap
  SNP-index data; do not use this skill to call SNPs or de novo annotate a genome
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0219
tools:
- name: WormBase
  role: Reference database used to verify C. elegans gene symbols and sequence identifiers
    (e.g. T24E12.5/rcd-1) against curated nomenclature before adding them to the mapping
    table
  repo: wormbase.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-50111-8
    title: Regulatory mechanism of cold-inducible diapause in Caenorhabditis elegans
  source_content_hash: sha256:2bed15796ed879e2532d141f169643411cc750a7f2c30754b0754220a74f8ff4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - WormBase
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# longevity-mutant-phenotype-annotation

## Summary

Curates the mapping between candidate CID-regulator genes called from MutMap SNP-index QTL analysis (nonsense mutations in EMS-mutagenised C. elegans hsf-1(sy441) suppressor strains) and their corresponding EMS strain numbers, verified against WormBase, output as gene_ems_strain_mapping.csv.

## When to use

Use when MutMap SNP-index analysis of EMS-mutagenised long-lived non-CID suppressor strains has produced candidate gene calls and you must assign each causal nonsense mutation (e.g. gst-13, mtk-1, nsy-1, smg-1, smg-2, sur-2, rcd-1) to specific EMS strain numbers.

## When NOT to use

- Mutagenised strains that did not carry nonsense mutations in the QTL — the study explicitly did not analyse these further, so attempting gene–strain assignment for them is out of scope
- Strains whose phenotype is not a suppressor of hsf-1(sy441) CID (i.e. not long-lived non-CID mutants) — the mapping is specific to the CID-suppressor screen
- Assignment of CID-regulator roles requires nonsense-mutation evidence from MutMap SNP-index data; do not use this skill to call SNPs or de novo annotate a genome

## Inputs

- MutMap SNP-index QTL analysis results for EMS-mutagenised long-lived non-CID suppressor strains of hsf-1(sy441)
- Causal gene calls with associated EMS strain numbers (EMS#21, EMS#27, EMS#28, EMS#30, EMS#31, EMS#39, EMS#40, EMS#48, EMS#49)
- Dual-annotation note for EMS#49: T24E12.5/rcd-1 (QTL gene) and che-11(hru153) (longevity mutation)

## Outputs

- gene_ems_strain_mapping.csv — a two-column table of gene symbol vs. EMS strain covering gst-13/EMS#28, mtk-1/EMS#31, nsy-1/EMS#21 and EMS#27, smg-1/EMS#39, smg-2/EMS#40, sur-2/EMS#30 and EMS#48, and T24E12.5/rcd-1/EMS#49

## How to apply

Retrieve the MutMap SNP-index analysis results reported for the EMS-mutagenised long-lived non-CID strains (49 non-CID mutants screened, of which 16 were long-lived) and extract each causal gene call together with its linked EMS strain number(s) into a two-column (gene symbol, EMS strain) structure. For EMS#49, capture the dual assignment explicitly: T24E12.5/rcd-1 as the SNP-index QTL gene and che-11(hru153) as the longevity mutation. Verify every gene symbol and sequence identifier against the C. elegans database WormBase (wormbase.org) before committing it to the table — this matters for unusual loci such as rcd-1 (regulator of cold-inducible diapause) and its paralogs T05A8.1 and Y47G7B.2, which had not yet been studied and lack orthologues in higher animal species. Assemble the curated pairs into a CSV file saved as gene_ems_strain_mapping.csv, ensuring no EMS strain entry is left unmatched.

## Related tools

- **WormBase** (Reference database used to verify C. elegans gene symbols and sequence identifiers (e.g. T24E12.5/rcd-1) against curated nomenclature before adding them to the mapping table) — wormbase.org

## Examples

```
import csv; rows=[('gst-13','EMS#28'),('mtk-1','EMS#31'),('nsy-1','EMS#21'),('nsy-1','EMS#27'),('smg-1','EMS#39'),('smg-2','EMS#40'),('sur-2','EMS#30'),('sur-2','EMS#48'),('T24E12.5/rcd-1','EMS#49')]; csv.writer(open('gene_ems_strain_mapping.csv','w')).writerows([('gene','ems_strain')]+rows)
```

## Evaluation signals

- The finished gene_ems_strain_mapping.csv reproduces all reported gene–EMS strain pairs: gst-13/EMS#28, mtk-1/EMS#31, nsy-1/EMS#21 and EMS#27, smg-1/EMS#39, smg-2/EMS#40, sur-2/EMS#30 and EMS#48, and T24E12.5/rcd-1/EMS#49
- There are no unmatched strain entries — every EMS strain number in the input appears in the output table
- EMS#49 carries both the QTL gene (T24E12.5/rcd-1) and the longevity mutation (che-11(hru153)), and the dual assignment is explicit
- All gene symbols have been checked against WormBase (wormbase.org) nomenclature
- Candidate gene set matches the article's list of hsf-1 mutant CID regulators: rcd-1, smg-1, smg-2, mtk-1, nsy-1, gst-13, and sur-2

## Limitations

- Mutagenised strains that did not carry nonsense mutations in the QTL were not analysed further in the study, so the mapping covers only strains with causal nonsense calls
- The rcd-1 gene and its paralogs (T05A8.1, Y47G7B.2) had not yet been studied and had no reported orthologues in higher animal species, so functional annotation for the rcd-1 assignment is limited
- How smg-1 regulates hsf-1 mutant CID via nonsense-mediated decay (NMD) or in parallel with NMD remains unresolved, so the mapping records candidate associations rather than proven mechanisms

## Evidence

- [discussion] Candidate genes identified as CID regulators: [evidence span withheld — anchor status: not_found]
- [discussion] Strains excluded from analysis: "Mutagenised strains that did not carry nonsense mutations in the QTL were not analysed further in the study."
- [discussion] WormBase verification of identifiers: "no orthologues in higher animal species were reported on the C. elegans database (wormbase.org)"
- [discussion] Screening origin of the mutagenised strains: "By selecting for suppressors of the CID phenotype of hsf-1(sy441) mutant, we obtained several long-lived mutants."
- [abstract] Genetic screen scale: "successfully isolate 16 long-lived mutants among 49 non-CID mutants via genetic screening"

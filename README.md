# snakemake_BAM2CRAMandQC
Snakemake Pipeline of BAM to CRAM and SampleQC

* Docker Images: https://hub.docker.com/r/spashleyfu/snakemake_bammetrics
* Snakemake Tutorial: https://snakemake.readthedocs.io/en/stable/tutorial/tutorial.html
* Snakemake Wrappers: https://snakemake-wrappers.readthedocs.io/en/stable/wrappers.html

### Singleton WGS SNP/Indel pipeline

1. [BASH] Parabricks Germline pipeline (Functional Equivalence)
2. **[Sankemake] BAM2CRAM_splitGVCF_SampleQC (This Repo)**
3. [Sankemake ?!] Joint-calling - Hail runCombiner
4. [Sankemake ?!] QC before VQSR - kinship, sex check, and PCA
5. VQSR and Hail Annotation


### Pipeline Outline:

1. [Sample QC] `verifybamid2 ... {BAM}` form wrapper - https://snakemake-wrappers.readthedocs.io/en/stable/wrappers/verifybamid/verifybamid2.html
2. [Sample QC] `bamMetrics ... {BAM}` from built tool (executable place at `/usr/bin/bamMetrics`)
3. [BAM to CRAM] `samtools view -C ...` form wrapper - https://snakemake-wrappers.readthedocs.io/en/stable/wrappers/samtools/view.html
4. [Split GVCF] `gatk SelectVariants ...` form wrapper - https://snakemake-wrappers.readthedocs.io/en/stable/wrappers/gatk/selectvariants.html


### Steps:

1. Create Folder Structure (2022/05/11)

```
[fup@compute1-client-3 snakemake_BAM2CRAM_splitGVCF_SampleQC]$ mkdir workflow
[fup@compute1-client-3 snakemake_BAM2CRAM_splitGVCF_SampleQC]$ touch workflow/snakefile
[fup@compute1-client-3 snakemake_BAM2CRAM_splitGVCF_SampleQC]$ mkdir workflow/rules
[fup@compute1-client-3 snakemake_BAM2CRAM_splitGVCF_SampleQC]$ mkdir config
```

2. 

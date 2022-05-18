# snakemake_BAM2CRAMandQC
Snakemake Pipeline of BAM to CRAM and SampleQC

* Docker Images: https://hub.docker.com/r/spashleyfu/snakemake_bammetrics
* Snakemake Tutorial: https://snakemake.readthedocs.io/en/stable/tutorial/tutorial.html
* Snakemake Wrappers: https://snakemake-wrappers.readthedocs.io/en/stable/wrappers.html

#### Testing on cluster:

```
bsub -q general -G compute-jin810 -J snakemake -N -u fup@wustl.edu \
-R 'affinity[core(5)] span[ptile=6] rusage[mem=25GB]' \
-g /fup/jobGroup_2_snakemake \
-a 'docker(spashleyfu/snakemake_bammetrics:latest)' \
snakemake --use-conda -j 10 \
--cluster-config /storage1/fs1/jin810/Active/fup/Snakemake_WGS_pipeline_JinLab/config/cluster.json \
--cluster "bsub -q general -G compute-jin810 -oo {cluster.log} -eo {cluster.err} -M {cluster.mem} -n {cluster.core} -R {cluster.resources} -g {cluster.jobgroup} -a 'docker({cluster.image})'" \
-s /storage1/fs1/jin810/Active/fup/Snakemake_WGS_pipeline_JinLab/workflow/snakefile
```


### Singleton WGS SNP/Indel pipeline

0. [sankemake] BAM/CRAM to FASTQ for WGS
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

2. BAM2CRAM actual run:

```
(snakemake) fup@compute1-exec-132:/storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC$ snakemake --use-conda -c4
Building DAG of jobs...
Creating conda environment https://github.com/snakemake/snakemake-wrappers/raw/v1.4.0/bio/samtools/view/environment.yaml...
Downloading and installing remote packages.
Environment for https://github.com/snakemake/snakemake-wrappers/raw/v1.4.0/bio/samtools/view/environment.yaml created (location: .snakemake/conda/833aa884fddc9e3185823803aaff9c60)
Using shell: /bin/bash
Provided cores: 4
Rules claiming more threads will be scaled down.
Job stats:
job              count    min threads    max threads
-------------  -------  -------------  -------------
all                  1              1              1
samtools_view       10              2              2
total               11              1              2

Select jobs to execute...

[Tue May 17 19:00:22 2022]
rule samtools_view:
    input: /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10000/TWHJ-PNRR-10000_germline.bam, /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10000/TWHJ-PNRR-10000_germline.bam.bai
    output: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10000_germline.bam.cram
    log: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/logs/BAM2CRAM/TWHJ-PNRR-10000_BAM2CRAM.log
    jobid: 1
    wildcards: sample=TWHJ-PNRR-10000
    threads: 2
    resources: tmpdir=/tmp/289556.tmpdir

[Tue May 17 19:00:22 2022]
rule samtools_view:
    input: /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10002/TWHJ-PNRR-10002_germline.bam, /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10002/TWHJ-PNRR-10002_germline.bam.bai
    output: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10002_germline.bam.cram
    log: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/logs/BAM2CRAM/TWHJ-PNRR-10002_BAM2CRAM.log
    jobid: 2
    wildcards: sample=TWHJ-PNRR-10002
    threads: 2
    resources: tmpdir=/tmp/289556.tmpdir

Activating conda environment: .snakemake/conda/833aa884fddc9e3185823803aaff9c60
Activating conda environment: .snakemake/conda/833aa884fddc9e3185823803aaff9c60
Activating conda environment: .snakemake/conda/833aa884fddc9e3185823803aaff9c60
Activating conda environment: .snakemake/conda/833aa884fddc9e3185823803aaff9c60

...

[Tue May 17 23:22:09 2022]
localrule all:
    input: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10000_germline.bam.cram, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10002_germline.bam.cram, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10017_germline.bam.cram, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10023_germline.bam.cram, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10041_germline.bam.cram, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10049-10049_germline.bam.cram, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10052_germline.bam.cram, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10062-10062_germline.bam.cram, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10065-10065_germline.bam.cram, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/TWHJ-PNRR-10066_germline.bam.cram
    jobid: 0
    resources: tmpdir=/tmp/289556.tmpdir

[Tue May 17 23:22:09 2022]
Finished job 0.
11 of 11 steps (100%) done
Complete log: .snakemake/log/2022-05-17T185837.566825.snakemake.log
```

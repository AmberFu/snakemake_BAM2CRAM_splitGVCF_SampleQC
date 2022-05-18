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
--cluster-config /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/config/cluster.json \
--cluster "bsub -q general -G compute-jin810 -oo {cluster.log} -eo {cluster.err} -M {cluster.mem} -n {cluster.core} -R {cluster.resources} -g {cluster.jobgroup} -a 'docker({cluster.image})'" \
-s /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/workflow/snakefile
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



2. QC + BAM2CRAM

```
// Second run
(snakemake) fup@compute1-exec-135:/storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC$ snakemake --rerun-incomplete -c12 --use-conda
Building DAG of jobs...
Creating conda environment https://github.com/snakemake/snakemake-wrappers/raw/v1.5.0/bio/verifybamid/verifybamid2/environment.yaml...
Downloading and installing remote packages.
Environment for https://github.com/snakemake/snakemake-wrappers/raw/v1.5.0/bio/verifybamid/verifybamid2/environment.yaml created (location: .snakemake/conda/26630509f0b0cface7fde1b2a712e492)
Using shell: /bin/bash
Provided cores: 12
Rules claiming more threads will be scaled down.
Job stats:
job                       count    min threads    max threads
----------------------  -------  -------------  -------------
KnightBamMetrics              6              4              4
all                           1              1              1
combinedBamMetrics            1              1              1
generateBamMetricsList        1              1              1
verify_bam_id                 9              1              1
total                        18              1              4

Select jobs to execute...

[Wed May 18 20:37:10 2022]
rule KnightBamMetrics:
    input: /storage1/fs1/bga/Active/gmsroot/gc2560/core/model_data/2887491634/build21f22873ebe0486c8e6f69c15435aa96/all_sequences.fa, /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10023/TWHJ-PNRR-10023_germline.bam, /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10023/TWHJ-PNRR-10023_germline.bam.bai
    output: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/KnightBamMetrics/TWHJ-PNRR-10023_bam_metrics.txt
    log: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/logs/KnightBamMetrics/TWHJ-PNRR-10023_KnightBamMetrics.log
    jobid: 14
    wildcards: sample=TWHJ-PNRR-10023
    threads: 4
    resources: tmpdir=/tmp/312002.tmpdir, mem_mb=10000
...


// Failed...because I didn't install VerifyBamID2
(snakemake) fup@compute1-exec-135:/rdcw/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC$ snakemake -c12 --use-conda
Building DAG of jobs...
Using shell: /bin/bash
Provided cores: 12
Rules claiming more threads will be scaled down.
Job stats:
job                       count    min threads    max threads
----------------------  -------  -------------  -------------
KnightBamMetrics             10              4              4
all                           1              1              1
combinedBamMetrics            1              1              1
generateBamMetricsList        1              1              1
verify_bam_id                10              1              1
total                        23              1              4

Select jobs to execute...

[Wed May 18 18:50:02 2022]
rule KnightBamMetrics:
    input: /storage1/fs1/bga/Active/gmsroot/gc2560/core/model_data/2887491634/build21f22873ebe0486c8e6f69c15435aa96/all_sequences.fa, /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10002/TWHJ-PNRR-10002_germline.bam, /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10002/TWHJ-PNRR-10002_germline.bam.bai
    output: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/KnightBamMetrics/TWHJ-PNRR-10002_bam_metrics.txt
    log: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/logs/KnightBamMetrics/TWHJ-PNRR-10002_KnightBamMetrics.log
    jobid: 12
    wildcards: sample=TWHJ-PNRR-10002
    threads: 4
    resources: tmpdir=/tmp/306578.tmpdir, mem_mb=10000


[Wed May 18 18:50:03 2022]
rule KnightBamMetrics:
    input: /storage1/fs1/bga/Active/gmsroot/gc2560/core/model_data/2887491634/build21f22873ebe0486c8e6f69c15435aa96/all_sequences.fa, /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10017/TWHJ-PNRR-10017_germline.bam, /storage1/fs1/jin810/Active/Neuropathy_WGS_2021May/Neuropathy_batch2/pbOut_v2/pb_germline/TWHJ-PNRR-10017/TWHJ-PNRR-10017_germline.bam.bai
    output: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/KnightBamMetrics/TWHJ-PNRR-10017_bam_metrics.txt
    log: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/logs/KnightBamMetrics/TWHJ-PNRR-10017_KnightBamMetrics.log
    jobid: 13
    wildcards: sample=TWHJ-PNRR-10017
    threads: 4
    resources: tmpdir=/tmp/306578.tmpdir, mem_mb=10000

...

[Wed May 18 19:25:58 2022]
Error in rule verify_bam_id:
    jobid: 7
    output: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/verifyBamID2/TWHJ-PNRR-10052.selfSM, /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/results/verifyBamID2/TWHJ-PNRR-10052.Ancestry
    log: /storage1/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/logs/verifybamid2/TWHJ-PNRR-10052_verifybamid2.log (check log file(s) for error message)
    conda-env: /rdcw/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/.snakemake/conda/5f46c514e2aa2caac7dab83e2cddae02

RuleException:
CalledProcessError in line 23 of /rdcw/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/workflow/rules/QC_verifyBam.smk:
Command 'source /opt/conda/bin/activate '/rdcw/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/.snakemake/conda/5f46c514e2aa2caac7dab83e2cddae02'; set -euo pipefail;  /opt/conda/envs/snakemake/bin/python3.10 /rdcw/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/.snakemake/scripts/tmp2elymp1l.wrapper.py' returned non-zero exit status 1.
  File "/rdcw/fs1/jin810/Active/PCGC_CHD_IncRNA_2022/snakemake_BAM2CRAM_splitGVCF_SampleQC/workflow/rules/QC_verifyBam.smk", line 23, in __rule_verify_bam_id
  File "/opt/conda/envs/snakemake/lib/python3.10/concurrent/futures/thread.py", line 58, in run

```

3. BAM2CRAM actual run:

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

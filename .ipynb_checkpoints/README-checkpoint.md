
# Overview
DENTIST (Detecting Errors iN analyses of summary staTISTics) is a quality control (QC) tool for summary-level data from genome-wide association studies (GWASs). It leverages the difference between the observed GWAS test-statistic of a variant and its predicted value (using the neighbouring variants and linkage equilibrium (LD) data from a reference panel) to remove problematic variants. It can detect genotyping/imputation errors in either the original GWAS or the LD reference samples, allelic errors (i.e., the effect alleles of the variants are mislabelled) in the GWAS summary data, as well as heterogeneity between the GWAS and LD reference samples. As shown in [our paper](#Citations), DENTIST can significantly improve the performance of the summary-data based conditional and jointly association analysis (COJO; Yang et al. 2012 Nat Genet) especially for rare variants. It can also improve the performance of LD score regression (LDSC; Builk-Sulivant et al. 2015 Nat Genet) and SMR-HEIDI (Zhu et al. 2015 Nat Genet). We believe that DENTIST can in principle work for all analyses that use GWAS summary data, although this has not been tested extensively. Apart from its application to summary data based analyses, DENTIST can also  be used in complementary to the conventional QCs in GWAS with individual-level data.

Software License: GPLv3 

# Credits
The method is developed by Wenhan Chen, Zhihong Zhu and [Jian Yang](https://publons.com/researcher/2848531/jian-yang/) in the Yang Lab at The University of Queensland. The software is programmed and maintained by Wenhan Chen.

# Questions and Help Requests
If you meet any bugs or questions, please send an email to [Wenhan Chen](mailto:uqwche11@uq.edu.au) or [Jian Yang](mailto:jian.yang.qt@gmail.com).

# Citations
Chen, W., Wu, Y., Zheng, Z. et al. Improved analyses of GWAS summary statistics by reducing data heterogeneity and errors. Nat Commun 12, 7117 (2021). https://doi.org/10.1038/s41467-021-27438-7


# Install
For compiling yourself, I have not made a user-friendly MAKEFILE yet. For now, you can type
```
git clone https://github.com/Yves-CHEN/DENTIST
cd DENTIST
make
```
and address the dependencies yourself on MKL, BOOST and EIGEN Libraries.


# Downloads
## Pre-compiled Executable Files
The executable file below is compiled with "-static" and tested on 64-bit Linux distributions on the x86_64 CPU platform,
[DENTIST_1.3.0.0](https://www.dropbox.com/scl/fi/1yna9rizfy6s0ixek0cca/DENTIST.1.3.0.0.gz?rlkey=0ak0cdsev152jteibceit4mje&dl=0)
[DENTIST_1.2.0.0](https://www.dropbox.com/s/cq5mmfonocvdiwh/DENTIST.1.2.0.0.gz?dl=0)
[DENTIST_1.1.0.0](https://www.dropbox.com/s/1mtskir8qzqsmee/DENTIST.1.1.0.0.gz?dl=0)
[DENTIST  0.9.2.1](https://www.dropbox.com/s/37bc35azxwbzdos/DENTIST.0.9.2.1.gz?dl=0).

To download, you can run
> wget -O DENTIST_1.3.0.0.gz https://www.dropbox.com/scl/fi/1yna9rizfy6s0ixek0cca/DENTIST.1.3.0.0.gz?rlkey=0ak0cdsev152jteibceit4mje&dl=0
> wget -O DENTIST_1.2.0.0.gz https://www.dropbox.com/s/cq5mmfonocvdiwh/DENTIST.1.2.0.0.gz?dl=0

## To compile (added March 7 by Gao Wang)

```
micromamba create -n toothache mkl mkl-include libgomp eigen boost zlib
ENV_PATH="$MAMBA_ROOT_PREFIX/envs/toothache"
export LD_LIBRARY_PATH="$ENV_PATH/lib:$LD_LIBRARY_PATH"
export PATH="$ENV_PATH/bin:$PATH"
export C_INCLUDE_PATH="$ENV_PATH/include:$C_INCLUDE_PATH"
export CPLUS_INCLUDE_PATH="$ENV_PATH/include:$ENV_PATH/include:$CPLUS_INCLUDE_PATH"
```

# Basic usage
To run DENTIST with essential parameters,
> DENTIST --gwas-summary summary_data --bfile ref --out prefix

To specify the number of CPUs,
> DENTIST --gwas-summary summary_data --bfile ref --out prefix --thread-num 10

To run DENTIST at a targeted region specified by the rsID of variant (e.g. rs1234),
>DENTIST --gwas-summary summary_data --bfile ref --out prefix --thread-num 10 --target rs1234

To run DENTIST for allelic frequencies
>DENTIST --gwas-summary summary_data --bfile ref --out prefix   --freq


# Input and output

> \-\-bfile \<PLINK bed file prefix\>

Reads individual-level genotype data in PLINK bed format, e.g. test.fam, test.bim and test.bed.

> \-\-gwas-summary \<summary statistics in GCTA-COJO format\>

Reads GWAS summary data in in GCTA-COJO format (see below for an example).
```
SNP A1 A2 freq beta se p N
rs131538 A G 0.05 0.007 0.02 0.7 6000
rs140378 C G 0.05 0.007 0.02 0.7 6000
...
```
> \-\-out \<the output file prefix\>
Specifies the prefix of the output files including  \*.DENTIST.full.txt, \*.DENTIST.short.txt, \*.DENTIST.ignored.txt \*.frq.

The .DENTIST.full.txt contains the statistics for all the tested variants in the following format, e.g. 
> head -3 \<prefix\>.DENTIST.full.txt

```
rsID        chisq   -log10(P)  ifDup
rs131538    0.012   0.91       0
rs140378    14.4    2          0
...
```
As shown above, the first column contains the rsID for each variant, followed by the DENTIST test statistic (follows a <img src="https://render.githubusercontent.com/render/math?math=\chi^2"> distribution with 1 degree of freedom under the null), and the corresponding <img src="https://render.githubusercontent.com/render/math?math=-log_{10}(p-value)"> in the 3rd column. The 4th column is an indicator of whether the variant is in strong LD (|r| > 0.99) with any other variants, 0 for none and 1 for at least one.

The DENTIST.short.txt contains the rsIDs for variants that cannot pass DENTIST QC and are suggested for removal.

The DENTIST.ignored.txt contains variants that are ignored through QC for reasons including 1) inconsistently found in summmary-data and PLINK files; 2) MAF threshold; 3) MAF difference threshold between MAFs from summary and reference data; 4) alleles do not match between summary and reference data. e.g.
head -3 \<prefix\>.DENTIST.ignored.txt
```
rs141578542     notFoundInGWAS
chr22:16051722:D        notFoundInGWAS
rs139918843     notFoundInGWAS
```

The \<prefix\>.frq shows the allelic frequencies of the A1 allele specific by --freq flag, e.g. 

```
chr     RS_ID   BP      A1      A2      Freq_A1 n       nMissingGeno
1       1       1       2       1       0.04879 7703    693
1       2       2       2       1       0.05588 7703    742
1       3       3       2       1       0.04507 7703    758
```
In example above, the Freq_A1 is the allelic frequency,  n is for the sample size of the cohort,  nMissingGeno is for the number of individuals which has NA genotype and excluded for the calculation.





# Cautions
* Do not perform cutoff on GWAS p-value.
* Make sure HWE QC is performed before running DENTIST.
* DENTIST will skip over low SNP density regions, any 2Mb-region with < 1000 SNPs.


# Parameters
| **Flag**  |**Description**                          |
|--------------------|------------|
| |**INPUT and OUTPUT files**  |
|-\-gwas-summary \<STR\>&nbsp;|Reads GWAS summary data in in GCTA-COJO format, e.g.  <br> `SNP A1 A2 freq beta se p N` <br> `rs131538 A G 0.05 0.007 0.02 0.7 6000` <br>`rs140378 C G 0.05 0.007 0.02 0.7 6000` |
|-\-bfile \<STR\>  |Reads individual-level genotype data in PLINK bed format, e.g. test.fam, test.bim and test.bed|
|-\-out \<STR\>|Specifies the prefix of the output files. |
|  |**Select/filter a chromsome/region/given SNPs**  |
|-\-chrID     \<STR\>| Specifies the chromosome ID. If not provided, DENTIST will guess the chromosome ID based on if there is only one chromosome ID provided in the inputs; otherwise an error is reported.|
|-\-target    \<STR\>|  Specifies a region of 20 Mb centred at a position specified by rsID. The rsID should be present in the PLINK bim file. A warning will be reported if the target rsID is not found, and DENTIST will run across the chromosome rather than the specified region.|
|-\-target-bp \<INT\>|   has the same function as -\-target, except for that the center is defined by a bp position not rsID. |
|-\-radius    \<INT\>| Specifies the radius of a region to run DENTIST in bp unit. Together with --target or --target-bp to specify a region of interest. |
|-\-maf       \<INT\>|    Specifies a MAF threshold so that variants with MAFs smaller than this value will be precluded from the analysis. Note that this exclusion is performed after --target.|
|-\-extract   \<STR\>| Specifies a filename containing SNPs to be included for analyses. This file should contain variant identifiers (usually RS ID) in one column.|
|-\-delta-MAF  \<FLOAT\>    | Specifies a threshold for variants with MAF differences between the GWAS summary and LD reference data set. This filter is not applied by default. The commonly used thresholds include 0.1 and 0.2.|
|| **Tuning DENTIST model parameters (refer to our [paper](#Citations))** |
|-\-wind-dist    \<INT\>|  Specifies the size of a sliding window (in bp units). The default value is 2000000 bp.|
|-\-wind         \<INT\>|   has the same function as -\-wind-dist except for that this is in number of variants in a region. |
|-\-iteration-num \<INT\>|Specifies the number of iterations in the DENTIST analysis. A too large value will increase the computational costs and a too small value will increase the false discovery rate. We have experimented with this parameter and set a default value of 10 for a good trade-off.|
|-\-dup-threshold     \<FLOAT\>|    Specifies the correlation threshold for variants considered as duplicates for each other. The default value is 0.99, which corresponds to Pearson’s correlation (r2) of 0.99^2|
|-\-p-value-threshold \<FLOAT\>| Specifies the GWAS P-value threshold to group variants into null variant and significant variants. The default value is 0.01.|
|-\-no-missing-genotype             | Specifies that there are no NA genotypes in the input PLINK BED file. DENTIST can perform slightly faster for these genotype data. |
|-\-SVD-trunc-prop    \<FLOAT\>|    Specifies the degree SVD trunction preformed in DENTIST. It is a value between 0 and 1 specifying the proportion of components being retained from SVD truncation. The default value is 0.5. |
||**SPEED**|
|-\-thread-num \<INT\>|  Specifies the number of threads for parallel computing using OpenMP. The default value is 1.|
||**Others**|
| -\-debug             | turns on verbose output for debugging. |
|-\-freq     | is a flag for generating the allelic frequencies instead of running DENTIST QC. Note that the -\-bfile and -\-gwas-summary are still needed in this mode.|
|-\-write-LD                  | is yet to be implemented. |
|-\-LD-unit-in-byte \<INT\>     | is yet to be implemented. |
|-\-load-LD                   | is yet to be implemented. |
|-\-bld \<STR\>               | is yet to be implemented.|
|-\-check-LD | is yet to be implemented.  |
|-\-impute   | is yet to be implemented.  |



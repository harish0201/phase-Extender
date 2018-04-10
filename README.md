# phASE-Extender
**Extender** for the readbackphased haplotype blocks.\
***A python program to extend the ReadBackPhased haplotype blocks using markov first order transition probabilities.***

Developed by [Bishwa K. Giri](mailto:kirannbishwa01@gmail.com) in the [Remington Lab](website?) at the University of North Carolina at Greensboro, Biology department.

# Citation
Giri, B. K., Remington D. L. Haplotype phase extension and preparation of diploid genome using phase-Extender. biorxiv (2018) [not uploaded yet].


# Intro to ReadBackPhasing
**Check these links for details on readbackphasing**
- https://software.broadinstitute.org/gatk/documentation/tooldocs/3.8-0/org_broadinstitute_gatk_tools_walkers_phasing_ReadBackedPhasing.php
- https://github.com/secastel/phaser/tree/master/phaser


**Runs on `Python 3.x` and has the following dependencies:**

  - [pandas](http://pandas.pydata.org/)
  - [argparse](https://docs.python.org/3/library/argparse.html)
  - [collections](https://docs.python.org/3/library/collections.html?highlight=collections#module-collections)
  - [csv](https://docs.python.org/3/library/csv.html?highlight=csv)
  - [decimal](https://docs.python.org/3/library/decimal.html?highlight=decimal#module-decimal)
  - [functools](https://docs.python.org/3/library/functools.html?highlight=functools)
  - [itertools](https://docs.python.org/3/library/itertools.html?highlight=itertools)
  - [io](https://docs.python.org/3/library/io.html?highlight=io#module-io)
  - [multiprocessing](https://docs.python.org/3/library/multiprocessing.html?highlight=multiprocessing#)
  - [numpy](http://www.numpy.org/), `http://cs231n.github.io/python-numpy-tutorial/`
  - [resource](https://docs.python.org/3/library/resource.html?highlight=resource#module-resource)
  - [time](https://docs.python.org/3/library/time.html?highlight=time#module-time)
  - [matplotlib](https://matplotlib.org/2.2.2/index.html)



# BACKGROUND
Haplotype phasing is a second "go to" problem in bioinformatics after read alignment. The importance of haplotype phasing applies directly to the analyses of ASE (allele specific expression), preparation of extended haplotype for EHH (extended haplotype homozygosity) test, and preparation of dipolid genome which will soon be a new standard in bioinformatics in coming years, etc. The necessity for haplotype phasing (and eventuallly diploid genome) increases with the increase in heterozygosity in the genome, because higher hetetogeneity leads to larger alignment bias and complicates the reliability of the variants that are called using that alignment data (SAM, BAM files).

For haplotype phasing tools like [Beagle](https://faculty.washington.edu/browning/beagle/beagle.html), [ShapeIT](http://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html), [impute2](http://mathgen.stats.ox.ac.uk/impute/impute_v2.html), etc are used dominantly. These tools make use of the variants (SNPs, InDels) along the length of the genome by treating each variants as single observations. So, for a chain of variants with "n" heterozygote sites there exists "2^n" possible haplotypes. Several tools treat each site separately and apply identity by descent, most common haplotype method, etc. to infer the possible haplotype for a sample. The main issue with "2^n" is increased computation burden however several sampling methods and algorithms have been designed to reduce it. But, still this method requires genotype data from lots of samples, have problem phasing rare variants, is mostly applicable to human and organism with reference haplotype panel.

ReadBackphasing is slowly emerging as a replacement (or new method??) for preparation of long range haplotypes; see tools [WhatsHap] (https://whatshap.readthedocs.io/en/latest/), [hapCut] (https://github.com/vibansal/hapcut), [phaser] (https://github.com/secastel/phaser/blob/master/phaser/README.md). But, the issue with existing RBphase method is that these tools:

   - are aimed at preparing long range haplotype but not necessarily genome wide. 
   - prepare the RBPhased haplotype blocks based on multiple input "BAM" and "VCF" files for a single sample and/or trios. The increase in the size of phased haplotype blocks only depends upon multiple BAM files, or multiple sets of longer reads from the same individual.

For specimen that don't have multiple genomic and rnaseq sources, reference haplotype panel the existing RBphase method is only able to yield short range haplotypes. Given multiple bam inputs these tools are still not aimed at preparing the haplotype genome wide, but rather are limited at preparing long streches of haplotype of certain part of a genome. Additionally, these RBphase tools do not have method to solve phase state between two adjacent haplotype blocks that are not spanned by sequence reads (in a sample) by using the readbackphased haplotype data information across population of samples. 

With increase in the size of PE (paired end) reads from illumina and availability of longer sequences from PacBio, it is possible to prepare short range haplotypes that span more than 2 heterozygous site in the genome. Thus, the issue with "2^n" problem can be overcome by first preparing RBphased haplotype blocks along the genome, and next by joining the two consecutive blocks in a sample one at a time. The size of RBphased blocks also increases with increase in heterozygosity in the genome thereby making this method more suitable for highly heterozygous populations. And, there is more information in RBphased haplotype compared with a single SNP or InDel thereby overcoming the issues with switcherrors when preparing the long range haplotype in heterozygous population. 

Existing haplotype phasing tools (that use RBphase and non-RBphase method) are oriented towards organism that have large genotype data and several haplotype reference panels available. For non-model organism these methods are therefore sub-optimal. RBphasing provides some phasing benefits because the phase state along short blocks can be readily extracted. However, converting the short haplotype fragments to longer haplotype strings can be challenging due to small sample size, low coverage within each sample, lack of enough genotype data, absence of reference panels in emerging research models. **phASE-Extender** overcomes these limitations by using the short range RBphased haplotype blocks from several samples that have several haplotype break points. **phase-Extender** is aimed at filling this technical gap by using the short fragments haplotypes among several samples, where the breakpoint of one sample is bridged by several other samples. 

**phASE-Extender** starts with already prepared RBphased haplotype blocks of several individuals at once and joins the two RBphased blocks for a single sample at once using overlapping phase states in other samples of the pool. Thus it is possible to solve the haplotype breakpoints by transfering the phase information between samples that belong to same family, populaton or species. *So, if we have a pool of several samples that have RBphased haplotypes, each sample will have several non-overlapping breakpoints among the samples. We can solve the phase state between two consecutive blocks (of one sample at a time) by computing the likelyhood of the phase states (parallel vs. alternate configuration). Likelihood of each configuration is computed by using the observed phase state in other samples that bridge that breakpoint position.* In **phASE-Extender** we use first order markov chain to compute the relationship between two consecutive haplotype blocks by preparing first order transition matrix from all the nucleotides of former haplotype block to all the nucelotides in the later haplotype block. We then compute cumulative likelyhood estimates of haplotype states for both the possible configuration. The phase state of two consecutive haplotype blocks is then extended if the `computed log2 (likelihood)` of either configuration is above the `threshold log2(likelihood) cutoff`. Using RBphase method we can therefore use less sample size to accurately predict the proper haplotype state. 


**
Another problem with emerging models is - switch errors and for the specimen with  higher heterozygosity (out crossing population and hybrids) switch errors can lead to further complicative problem with genomic analyses.

So, **phase-Extender** can be used with any RBphased input haplotype file given it meets the appropriate data structure. 
**


## Input data

**phASE-Extender** can be used with the multi-sample vcf files produced by GATK pipeline or several other tools that generate readbackphased haplotype blocks in the output VCF. A haplotype file is created using the RBphased VCF and then piped into **phase-Extender**. See, this example of a [sample input haplotype file](https://github.com/everestial/phase-Extender/blob/master/example01/input_haplotype_small.txt) - a tab separated text file with `PI` and `PG_al` value for each samples.

In the several example (test) files I havve used RBphased VCF produced by `phaser` (https://github.com/secastel/phaser , https://github.com/secastel/phaser/tree/master/phaser) that consists of `Phased Genotype i.e PG` and `Phase Block Index i.e PI` values in `FORMAT` field. PI represents the unique haplotype block index (or key) in the given VCF files and PG represents the phased genotypes within that PI block. **phase-Extender** then uses the `Phased Genotype i.e PG` and `Phase Block Index i.e PI` values to prepare the transition matrix. 

So, it is crucial to run **phaser** on your VCF of interest before running phase extension or prepare the "input haplotype file" in appropriate format to pipe it in **phase-Extender**.


# Algorithm
For the **mcve** regarding the algorithm see this issue on **stackoverflow** https://codereview.stackexchange.com/questions/186396/solve-the-phase-state-between-two-haplotype-blocks-using-markov-transition-proba . 
**A more comprehensive and a neat diagrammatic version will be soon included in my wordpress account **



## Tutorial

# Setup
**phase-Extender** is written in python3 interpreter. So, it can be directly run using the **".py"** file, given all the required modules are installed.

# Usage
Requires a VCF file or a haplotype file as input and returns a extended haplotype file and other results files containing statistics on the initial vs. extended haplotype. Optionally, haplotype reference panel (with same data structure as input haplotype) and bed file can be included to gain control on the phase extension.

VCF file from `phaser` can be converted to haplotype file using the python parser `vcf_to_table-v2.py`. 

    vcf_to_table-v2.py --vcf a_vcf_file.vcf --out a_haploype_file.txt
    
`--unphased yes` can be added to above command to also included the genotypes that have unphased state. This parameter will not affect the phase extension, and is only included to keep the whole data intact.
    
**Test case 01**

Using file from [example 01](https://github.com/everestial/phase-Extender/tree/master/example01)

Run command:

    python3 phase_extender_v1-final.py --input input_haplotype_small.txt --SOI ms02g 

**Test case 02**

Using file from [example 02](https://github.com/everestial/phase-Extender/tree/master/example02)

Run command: ** complete ??

    python3 phase_extender_v1-final.py --input input_haplotype_small.txt --SOI ms02g 
    
**Additional input files **

***haplotype reference panel:*** If your goal is to use reference haplotype panel, we suggest providing the haplotype reference panel with same data structure as input haplotype file.

To convert the haplotype reference panel (from VCF to proper text format) use: ** complete ?? **

    python3 hapRefVCFtoTxt.py --input hapRef.vcf --out  
  
***bed file:*** If you goal is to limit phase extension to certain genomic regions (for eg. gene, exon or QTL boundries), we suggest that you provide appropriate bed file. Remember, **phase-Extender** is exclusively limited to bed regions.   

To convert the GTF,GFF to bed file for appropriate GTF,GFF feature use:  ** complete ??

    python3 gffToBed.py  --input myGTF.gtf  --output myBed.bed
    
# Arguments
## Required
* **--bam** - Comma separated list of BAM files containing reads. Duplicates should be marked, and files should be indexed using samtools index.

# Optional
* **--python_string** _(python2.7)_ - Command to use when calling python, required for running


## Performance Related
* **--nt** _(1)_ - Maximum number of processes to run at once
snpTh
numHets


# Output Files

## *out_prefix*.haplotypes.txt

Contains all haplotypes phased along with detailed phasing statistics.

* 1 - **contig** - Contig haplotype is found on.





## **initial**.hapstats
parameters in this file ??

## **initial**.hapstats
parameters in this file ??


## **some**.plot.png

# Recursive application of haplotype phase extension
  **phase-Extender** is designed to create long range haplotyes (and possible GW haplotype) in pool of samples. Owing to it's RBphased method which contains information from multiple SNP position, it can configure proper haplotype extension even with small number of samples which is important in emerging research models. phase-Extender also provides the benefit of allowing the user to control haplotype extension by allowing fexible values in parameters `snpTh`, `minHets`, `log2Odds cut off`. This way a user provide a more flexible means of controlling the phase extension. 
  
  ***phase-Extender maynot be able to prepare a full length genotype in one run.*** This is not the limitation but rather a intended feature in this tool. The reason is to provide flexibility and allow the user to control phase extension. A controllable haplotype extension is largely required method for phase extension in emerging research model owing to resource scarcity. The main idea is to first run **phase-Extender** with higher `log2Odds cut off` for several samples. Then merge the output of each sample to run another round of **phase extension** with concessive (lower) `log2Odds cut off`. So, when applied recursively we reduced the number of haplotypes and increase the length of haplotypes in each chromosome.
  
  So, controllable haplotype extension is a novel feature intended in **phase-Extender**. A full length recursive phase extension is illustrated in this link [phase Extender on recursive mode](some website??) using the bash script.
  
  



# Some Q/A on phase-extender: 

  **_1) What kind of algorithm does phase-extender use ?_**  
    phase-extender uses first-order-transition probabilities from each level of genotypes from former haplotype block to each level of genotypes to later haplotype block. This version (v1) uses **forward-1stOrder-markov chains** and **backward-1stOrder-markov chains** transition probabilities. Future versions will follow improvements by adding markov-chains of higher order.    
    
   **2) What is the advantage of using phase-extender ?**
   
    We generally need accurate phase state with in a gene/transcript level while doing ASE, dissecting maternal-paternal effects.  
    Long haplotypes are mostly important while preparing diploid genome, long range haplotype for testing selective sweeps, 
    within a QTL regions etc. 
    For emerging organism systems where CW,GW haplotypes are more difficult to solve, it is more important to solve haplotypes 
    within short gene regions. 
    But, most of RBphased Haplotype rarely cover full gene regions due to coverage issues, or due to short read sequence that 
    cannot cover whole gene, etc. In that situation, if we rather have short haplotype blocks from severals samples it can be 
    used to solve a full gene length haplotypes for each sample (one at a time). phase-extender comes handy at this stage; that 
    it takes the information from several haplotype blocks from other samples that are not broken at that
    point. So, we can solve haplotype configuration for SOI (sample of interest) pretty much easily. ***Also, with paired-end 
    read size getting increased in Illumina sequence data we can have more reliable full gene-level haplotypes for SOI.***
    
   **4) Does phase-extender phase InDels ?**
    Yes, but it is conditional. The InDels should already be phased to a reliable haplotype block. That way when the haplotype is
    being extended for those SNPs, InDels too get extended.
    
   **5) What is the minimal size of the haplotype block that is required? **  contd ....
    The bigger the two haplotypes, the better is the likelyhood test of which haplotype is phased with which. By, default
    I have kept this number to 3 variants (SNPs exclusive) per haplotype block that needs extension.
    
   Does phase-extender do GW (genome wide) or CW (chromosome wide) haplotype phasing?
    There are certain situation when phase-extender is able to do GW or CW haplotype phasing.
    A) If you have lots of samples where the haplotypes overlap between sample covers almost all chromosome length. In this
       case we can run phase-extender for each sample there by extending the haplotype. After this phase-extender can be 
       applied recursively on the updated data each time, there by extending the haplotypes for each sample to full chromosome
       length and possibly to to full genome wide length. This is going to be more and more possible due to samples being
       sequenced at higher coverage, increase in the size of paired-end sequence length, availability of large sequence reads
       like pac-bio reads. The method for doing is explained in this link/website ..... ??
       
    B) Another situation when GW/CW phase extension might be possible is when you have at least few samples which have haplotype 
       resolved at GW/CW level. These can include fully phased data like genome matrix file, fully phased VCF data, fully phased
       haplotype blocks. For this the fully phased sample should be provided as one single blocks in the group of sample that is
       fed to phase-extender. Here is an example.... ??
       
   Does phase-extender phase the SNPs that are not phased to any other SNPs?
    For, now phase-extender doesn't have algorithm built into it to do so. I am hoping to add that ASAP.
      
   Does phase-extender impute missing genotypes?
    No, it does not. I have not developed any algorithm so far to impute missing genotypes. Should anyone be interested in 
    developing an algorithm, and have data to do so; I would be more than happy to.  
    
   Does phase-extender use haplotype reference panel?
    No, until now. Though it should be possible in near future.
   
   Does phase-extender use recombination into account?
    No and possibly these feature will be of least importance in phase-extender. Phase-Extender mostly relies on already       
    phased haplotype block from other samples. These haplotype blocks which were phased in other sample but in SOI were used 
    to build transition probabilities. There is strong assumption that variation in other samples are not the result of 
    recombination but only mutation.
    
   Does phase-extender phase rare genotypes?
    Yes, it does. But, the rare genotype should be the part (phased to) of phased haplotype blocks. This is one of the good 
    thing about phase-extender compared to other tools when it come to rare genotype. When several SNPs are phased together 
    to extend the haplotype, rare genotypes are really hard to phase accurately - the reason being the statistical 
    significance of the rare genotype belonging to either two phase state is highly ambigous. But, if the rare genotype is 
    attached to a haplotye block supported by read-back phasing, this makes phasing of rare genotypes most accurate, since 
    the likely hoods are provided by other SNPs that are not rare.
    
   How fast is phase-extender?
    phase-extender is written in python-3, so it is slower than other tools that are built on the top of C, C++ or java.   
    Coming from a pure biology background, learning python was one of the most enduring task I have taking and then 
    building this tool was a big part of my PhD. I have optimized the part of calling VCF file using cyvcf2 (which is on 
    average 4 times faster than old pyVCF module). phase-extender is also optimized for being able to run on multiple 
    threads/process. But, if you are running phase-extender on big genome data and have very large number of samples, and    
    running on laptop I suggest running on one thread, which may be time consuming but will reduce memory burden.
    
   Who helped me during the preparation of this program?
    I have not been very fortunate to surround myself or at least get face to face help from savvy computer programmers.
    But, my heart is very thankful to people behind the web who have made me capable of working this problem out - Thanks 
    to many people on biostars, stackoverflow, seqanswer and google web searches.
    
   Does phase-extender do trio based phase extension?
   
   
   What should be the relatedness of my samples?
   
   
   Has phase-extender been tested?
   
   
   What is differece between phase extender and phase stitcher?
   
   
   
   Should I prepare my haplotype block file onlyusing phaser?
   
   
   
   
## Note:
- Basically this tool takes the paritally phased haplotypes
    - segregates them to maternal vs. paternal population haplotypes (using markov model)
    - stitches the haplotypes to create a genome wide haplotype
    
This program is exclusively designed to phase haplotypes in F1 hybrid for now and should work equally with data generated from genome reseq, RNAseq, WES or RAD given the individual is a hybrid derived from divergent strains and/or populations, but may be used to equally extended to phase F2 hybrids (work on progress).


# Future updates
## Phase SNPs 
## Imputation
## Trio based phasing
## higher order markov chain capabilities
## multiprocessing within chromosome



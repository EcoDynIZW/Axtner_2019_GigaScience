# Axtner et al. 2019 *GigaScience*

> Jan Axtner, Alex Crampton-Platt, Lisa A Hörig, Azlan Mohamed, Charles C Y Xu, Douglas W Yu & Andreas Wilting (2019) "An efficient and robust laboratory workflow and tetrapod database for larger scale environmental DNA studies" GigaScience 8(4):giz029. DOI: [10.1111/1365-2656.13555](https://doi.org/10.1093/gigascience/giz029)

The use of environmental DNA for species detection via metabarcoding is growing rapidly. We present a co-designed lab workflow and bioinformatic pipeline to mitigate the 2 most important risks of environmental DNA use: sample contamination and taxonomic misassignment. These risks arise from the need for polymerase chain reaction (PCR) amplification to detect the trace amounts of DNA combined with the necessity of using short target regions due to DNA degradation.  
Our high-throughput workflow minimizes these risks via a 4-step strategy: (i) technical replication with 2 PCR replicates and 2 extraction replicates; (ii) using multi-markers (12S,16S,CytB); (iii) a “twin-tagging,” 2-step PCR protocol; and (iv) use of the probabilistic taxonomic assignment method PROTAX, which can account for incomplete reference databases. Because annotation errors in the reference sequences can result in taxonomic misassignment, we supply a protocol for curating sequence datasets. For some taxonomic groups and some markers, curation resulted in >50% of sequences being deleted from public reference databases, owing to (i) limited overlap between our target amplicon and reference sequences, (ii) mislabelling of reference sequences, and (iii) redundancy. Finally, we provide a bioinformatic pipeline to process amplicons and conduct PROTAX assignment and tested it on an invertebrate-derived DNA dataset from 1,532 leeches from Sabah, Malaysia. Twin-tagging allowed us to detect and exclude sequences with non-matching tags. The smallest DNA fragment (16S) amplified most frequently for all samples but was less powerful for discriminating at species rank. Using a stringent and lax acceptance criterion we found 162 (stringent) and 190 (lax) vertebrate detections of 95 (stringent) and 109 (lax) leech samples.  
Our metabarcoding workflow should help research groups increase the robustness of their results and therefore facilitate wider use of environmental and invertebrate-derived DNA, which is turning into a valuable source of ecological and conservation information on tetrapods.


## ScreenForBio metabarcoding pipeline

A series of bash and R scripts used in:

Jan Axtner, Alex Crampton-Platt, Lisa A Hörig, Azlan Mohamed, Charles C Y Xu, Douglas W Yu, Andreas Wilting, An efficient and robust laboratory workflow and tetrapod database for larger scale environmental DNA studies, GigaScience, Volume 8, Issue 4, April 2019, giz029, https://doi.org/10.1093/gigascience/giz029

Provides a complete analysis pipeline for paired-end Illumina metabarcoding data, from processing of twin-tagged raw reads through to taxonomic classification with *PROTAX*.

The pipeline steps that will be of most use to other projects are those related to *PROTAX*: generating curated reference databases and associated taxonomic classification; training *PROTAX* models (weighted or unweighted); and classification of OTUs with *PROTAX*.

Steps and associated scripts:
1. Process twin-tagged metabarcoding data
  - *read_preprocessing.sh*
2. Obtain initial taxonomic classification for target taxon
  - *get_taxonomy.sh*
3. Generate non-redundant curated reference sequence database for target amplicon(s) and fix taxonomic classification
  - *get_sequences.sh*
4. Train PROTAX models for target amplicon(s)
  - *train_protax.sh* (unweighted) or *train_weighted_protax.sh* (weighted)
  - *check_protax_training.sh* (makes bias-accuracy plots)
5. Classify query sequences (reads or OTUs) with PROTAX
  - *protax_classify.sh* or *protax_classify_otus.sh* (unweighted models)
  - *weighted_protax_classify.sh* or *weighted_protax_classify_otus.sh* (weighted models)

**Note:** in some steps the ***screenforbio-mbc*** release associated with the manuscript is specific to the amplicons used in the study - primer sets and relevant settings are hard-coded in *read_preprocessing.sh* and *get_sequences.sh*. This will be generalised in a future release.

### Required software (tested versions)
Pipeline tested on Mac OSX (10.13) and Scientific Linux release 6.9 (Carbon).

Mac OSX should have GNU grep, awk and sed prioritised over BSD versions. These can be installed with homebrew.

- Processing of raw reads only
  - bcl2fastq (v2.18)
  - AdapterRemoval (v2.1.7)
  - cutadapt (v1.14)
  - usearch (v8.1.1861)
- Building databases and PROTAX
  - R (v3.5.0)
  - tabtk (r19)
  - seqtk (v1.2-r94 and v1.2-r102-dirty)
  - seqkit (v0.7.2 and seqkit v0.8.0)
  - Entrez Direct (v6.00 and v8.30)
  - usearch (v8.1.1861 and v10.0.240)
  - blast (v2.2.29 and v2.7.1)
  - mafft (v7.305b and v7.313)
  - sativa (v0.9-57-g8a99328)
  - seqbuddy (v1.3.0)
  - last (926)
  - perl (v5.10.1 and v5.26.1)

*get_sequences.sh* also requires MIDORI databases for mitochondrial target genes [Machida *et al.*, 2017](https://www.nature.com/articles/sdata201727). Download relevant MIDORI_UNIQUE FASTAs in RDP format from the [website](http://www.reference-midori.info/download.php). Manuscript used MIDORI_UNIQUE_1.1 versions of COI, Cytb, lrRNA and srRNA. Unzipped FASTAs should be placed in working directory.

*get_sequences.sh*  also requires collapsetypes_v4.6.pl to be in the ***screenforbio-mbc*** directory. Download from Douglas Chesters' [sourceforge page](https://sourceforge.net/projects/collapsetypes/).

*PROTAX* scripts are reposted here with the kind permission of Panu Somervuo. These are in the *protaxscripts* subdirectory of ***screenforbio-mbc***. This version of *PROTAX* is from [Rodgers *et al.,* 2017](https://doi.org/10.1111/1755-0998.12701), scripts were originally posted on [Dryad](https://datadryad.org/resource/doi:10.5061/dryad.bj5k0).

### Usage
All steps in the pipeline are implemented via bash scripts with similar parameter requirements. Each script includes commented usage instructions at the start and displays the same instructions if run without any or an incorrect number of parameters.

Some of the bash scripts are primarily wrappers for R scripts, all of which are assumed to be in the ***screenforbio-mbc*** directory.

*get_taxonomy.sh* example:

    mkdir ~/Documents/mbc_analysis
    cd ~/Documents/mbc_analysis
    bash ~/Documents/screenforbio-mbc/get_taxonomy.sh

You will see the following message:

    You are trying to use get_taxonomy.sh but have not provided enough information.
    Please check the following:

    Usage: bash get_taxonomy.sh taxonName taxonRank screenforbio
    Where:
    taxonName is the scientific name of the target taxon e.g. Tetrapoda
    taxonRank is the classification rank of the target taxon e.g. superclass
    screenforbio is the path to the screenforbio-mbc directory

To get the ITIS classification for Mammalia:

    bash ~/Documents/screenforbio-mbc/get_taxonomy.sh Mammalia class ~/Documents/screenforbio-mbc

When the script is running various messages will be printed to both the terminal and a log file. On completion of the script the final messages will indicate the next script that should be run and any actions the user should take beforehand.

Enjoy :-)

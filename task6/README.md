# Task6 - Resequencing: variant calling from NGS data

In this lab (based on this [tutorial](https://angus.readthedocs.io/en/2014/variant.html)) you will be exploring some resequencing data from Richard Lenski's famous E. coli long term evolution experiment.
More on Lenski's long term evolution experiment can be found in this [article](http://www.nature.com/nature/journal/v489/n7417/full/nature11514.html)

...and here: https://en.wikipedia.org/wiki/Richard_Lenski

You will be mapping the reads from a single population of E. coli at 38,000 generations that has evolved citrate-utilization capacity (Cit+). You will map the reads to the reference genome in order to identify SNPs that have occurred in this lineage (or its ancestors). You will focus on one SNP in particular that has created a "mutator" strain by disrupting the <i>mutS</i> DNA-repair gene.


### Requirements

#### Command-line tools
* Access to a linux-based OS running BASH
* [bwa](http://bio-bwa.sourceforge.net/)
* [samtools](http://samtools.sourceforge.net/)
* [bcftools](https://samtools.github.io/bcftools/bcftools.html)

#### Graphical tools

You will also need to download and install IGV on your own machine.

* [igv](http://software.broadinstitute.org/software/igv/)


## Getting Started

* Login to your linux environment and create a new folder for your task6

```
mkdir task6  #creates folder
cd task6 #enters into folder
```

## Retrieving the raw data

First, download the E. coli reference genome:

```
wget athyra.idyll.org/~t/REL606.fa.gz
gunzip REL606.fa.gz
```

Next, download the resequencing data. This is 229 MB so you may have to be patient.

```
wget http://ftp.sra.ebi.ac.uk/vol1/fastq/SRR098/SRR098038/SRR098038.fastq.gz
```


## Mapping your reads to the reference genome


Before we can map reads with `bwa` (note: `bowtie` is another option), we need to index the reference genome. This can be done with `bwa index`.

```
bwa index REL606.fa
```

Now, let's map the reads to the reference genome. This is also a fairly intensive step that may take a few minutes.

```
bwa aln REL606.fa SRR098038.fastq.gz > SRR098038.sai
```

Make a .SAM file which contains all information about where each read maps onto the reference genome

```
bwa samse REL606.fa SRR098038.sai SRR098038.fastq.gz > SRR098038.sam
```

Index the reference genome (again) so that `samtools` can work with it

```
samtools faidx REL606.fa
```

Convert .SAM file to .BAM file

```
samtools import REL606.fa.fai SRR098038.sam SRR098038.bam
rm SRR098038.sam  # remove this large file
```

Sort BAM file and index it

```
samtools sort SRR098038.bam > SRR098038.sorted.bam
samtools index SRR098038.sorted.bam
```

## Viewing your BAM file

BAM files can be viewed with `igv` or with `tablet`. But let's take a quick look in the terminal.

```
samtools tview SRR098038.sorted.bam
```

![#1589F0](https://placehold.it/15/1589F0/000000?text=+) Q1 - Can you find a read with a mutation? Paste a screenshot and indicate where the mutation is. 

![#1589F0](https://placehold.it/15/1589F0/000000?text=+) Q2 - Do you think this mutation is a real variant or an error? Why?


## Variant calling

Instead of identifying SNPs by eye, use `bcftools` to perform automated variant calling

```
bcftools mpileup -f REL606.fa SRR098038.sorted.bam | bcftools call -mv -Ob --ploidy 1 -o calls.bcf

#convert to vcf (human-readable variant call format). This file should contain all identified SNPs and other variants.
bcftools view calls.bcf > calls.vcf

```

![#1589F0](https://placehold.it/15/1589F0/000000?text=+) Q3 - How many total variants are present? Hint: `grep` for a pattern found only in your variant lines.

![#1589F0](https://placehold.it/15/1589F0/000000?text=+) Q4 - Copy and paste into your assignment a line from the VCF file containing a SNP and a line containing an indel variant.


## Locating a key SNP in Lenski's E. coli evolution experiment

This lineage of <i>E. coli</i> has a mutation in the <i>mutS</i> gene (protein sequence can be found [here](https://www.uniprot.org/uniprot/P23909.fasta)). This mutation creates a premature stop codon. Your task is to find this mutation within your sequencing data!

Find the region in the reference genome that encodes the <i>mutS</i> gene using `blast`. You may need to refer to earlier tasks to help you with this.

Now, extract the mapped regions for this region from your .bam file. The command will be something like this:

```
samtools view SRR098038.sorted.bam "rel606:START-END" > region.sam   # where START and END are position numbers
```


Download the following three files and open these files in `igv` on your home machine.

- the region.sam file you created above
- the calls.vcf file you created above
- the reference genome ([REL606.fa](http://athyra.idyll.org/~t/REL606.fa.gz))

Now, locate the region containing the <i>mutS</i> gene within `igv`.

![#1589F0](https://placehold.it/15/1589F0/000000?text=+) Q5 - Paste a screenshot of the region containing this mutation. Your `igv` screenshot should span the full region of the <i>mutS</i> gene and display two tracks: the coverage and the variants.

![#1589F0](https://placehold.it/15/1589F0/000000?text=+) Q6 - Now locate the variant that corresponds to the premature stop codon. Paste a screenshot highlighting this mutation (you will need to zoom in) and show the amino acid translation (see [here](https://software.broadinstitute.org/software/igv/sequence_track_options). What was the amino acid encoded by this codon before this mutation?


![#1589F0](https://placehold.it/15/1589F0/000000?text=+) Once you are finished, please delete the files in your task 6 folder like this:

```
cd task6
rm *
```



---

# ASSIGNMENT QUESTIONS

The questions for this task are indicated by the lines starting with ![#1589F0](https://placehold.it/15/1589F0/000000?text=+) above.
Please submit the code you used as well as the answers to the questions. Submit your assignment to a dropbox on LEARN as a .docx, .txt, or .pdf file.




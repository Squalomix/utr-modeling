# Genome sequence annotation


## Step-by-step guide for UTR modeling

This page provides a third-party guide to modifying an existing GTF file by extending 3’-ends of transcripts using the program [peaks2utr](https://github.com/haessar/peaks2utr). This process is demanded by possible data loss caused due to the biased distribution of the reads obtained in some transcriptome sequencing methods including the [Chromium Single Cell Gene Expression of 10X Genomics](https://www.10xgenomics.com/products/single-cell-gene-expression). The versions of the programs and example files employed below in this guide follows our own experience with medaka demonstrated as part of the activity of NBRP Medaka Project of Japan. <br>


### 1. Installation

**1.1.** Installing peaks2utr v1.1.2<br>
<br>
The installation requires the Python version 3.8 to 3.10 (not 3.11, the latest).

Follow the official instruction to install with ‘pip’.
```
pip install peaks2utr
```
Also see Supplementary Data of the publication by the developers ([Haese-Hill et al., Bioinformatics 2023](https://academic.oup.com/bioinformatics/article/39/3/btad112/7067741))
<br>

**1.2.** Installing cellranger v7.1.0<br>

Follow the [official instruction](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/installation).<br>
<br>
### 2. Prepare a GTF file (genemodel.gtf)

Use a GTF file from Ensembl if available for two reasons. First, 10X Genomics recommend a GTF file from Ensembl in its official page. Second, our trial of running peaks2utr with a GTF file from NCBI failed (see [Part 5](https://github.com/Squalomix/utr-modeling/tree/main#5-run-peaks2utr)).

e.g., [Oryzias_latipes.ASM223467v1.110.gtf.gz](https://ftp.ensembl.org/pub/release-110/gtf/oryzias_latipes/)

### 3. Prepare a genome assembly file (assembly.fna)

e.g., [Oryzias_latipes.ASM223467v1.dna_sm.toplevel.fa.gz](https://ftp.ensembl.org/pub/release-110/fasta/oryzias_latipes/dna/Oryzias_latipes.ASM223467v1.dna_rm.toplevel.fa.gz)

### 4. Prepare input .bam file

The program peaks2utr is meant to employ Chromium scRNA-seq data as input (Part 4.1), but it can also accept bulk RNA-seq data (Part 4.2). Choose one of these two options.

**4.1.** Use 10X Chromium single cell RNA-seq data set <br>
<br>
Format the reference and gene model using the genome assembly (assembly.fna) and the GTF file (genemodel.gtf) (see the [official guide](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/advanced/references))

```
cellranger mkref --genome=custom_ref --genes=genemodel.gtf --fasta=assembly.fna
```
Map Chromium scRNA-seq reads in the directory 'fastq_dir' with ‘cellranger count’
```
cellranger count --id=run_count --fastqs=fastq_dir --transcriptome=custom_ref
```
**4.2.** Use bulk RNA-seq data<br>
<br>
Map the trimmed reads with [hisat2](http://daehwankimlab.github.io/hisat2/) or equivalent onto the genome assembly (assembly.fna)
<br><br>

### 5. Run peaks2utr 
Use the BAM file made above in Part 4 (Part 4.1 or 4.2)
```
peaks2utr --gtf genemodel.gtf run_count/outs/possorted_genome_bam.bam -o genemodelNEW.gtf
```
Consider tweaking the parameter --max-distance ('maximum distance in bases that UTR can be from a transcript') depending on typical UTR lengths and other genomic spacing trends in the species of interest. 

We manaegd to complete this whole process using a GTF file from Ensembl (see [Part 2](https://github.com/Squalomix/utr-modeling/blob/main/README.md#2-prepare-a-gtf-file-genemodelgtf) above) and failed using a file from NCBI for an unknown reason.

### 6. Analyze the peaks2utr output<br>

 **6.1.** Open the output file 'summary_stats.txt'

 **6.2.** Use ‘agat_sp_statisctics.pl’ from [AGAT (Another GTF/GFF Analysis Toolkit)](https://agat.readthedocs.io/en/latest/index.html)


### 7. Format the reference and gene model modified by peaks2utr

```
cellranger mkref --genome=custom_refNEW --genes=genemodelNEW.gtf --fasta=assembly.fna
```

### 8. Map scRNA-seq reads

```
cellranger count --id=run_countNEW --fastqs=fastq_dir --transcriptome=custom_refNEW
```

### 9. Analyze the output 

**9.1.** Confirm an expected increase of mapping % ('Reads Mapped Confidently to Transcriptome'). Compare this with that of the product of the [Part 4.1](https://github.com/Squalomix/utr-modeling/blob/main/README.md#4-prepare-input-bam-file)



## Acknowledgments

We thank Osamu Nishimura at RIKEN BDR and Satoshi Ansai at Kyoto Univ for discussion.

## References
### Similar efforts
[Bilgic et al., bioRxiv 2023 'Truncated radial glia as a common precursor in the late corticogenesis of gyrencephalic mammals'](https://www.biorxiv.org/content/10.1101/2022.05.05.490846v3)<br>
[Lawson et al., eLife 2020 'An improved zebrafish transcriptome annotation for sensitive and comprehensive detection of cell type-specific genes'](https://elifesciences.org/articles/55792)<br>

### Tools 
[Haese-Hill et al., Bioinformatics 2023 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'](https://academic.oup.com/bioinformatics/article/39/3/btad112/7067741)<br>


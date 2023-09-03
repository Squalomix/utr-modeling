# utr-modeling

This page provides a third-party guide to modifying an existing GTF file by extending 3’-ends of transcripts using the program [peaks2utr](https://github.com/haessar/peaks2utr). This process is demanded by possible data loss caused due to the biased distribution of the reads obtained in some transcriptome sequencing methods including the 10X Chromium single cell transcriptome sequencing system.

## Step-by-step guide

1. Installation

1.1. Installing peaks2utr v1.1.2<br>
Follow the official instruction to install with ‘pip’.
```
pip install peaks2utr
```
Also see Supplementary Data of the developer’s publication ([Haese-Hill et al., Bioinformatics 2023](https://academic.oup.com/bioinformatics/article/39/3/btad112/7067741))

1.2. Installing cellranger v7.1.0<br>

Follow the [official instruction](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/installation) to install with ‘pip’.<br>
<br>
2. Prepare a GTF from Ensembl if available (genemodel.gtf)
<br>
   e.g., [Oryzias_latipes.ASM223467v1.110.gtf.gz](https://ftp.ensembl.org/pub/release-110/gtf/oryzias_latipes/)
<br><br>
3. Prepare a genome assembly (assembly.fna)
<br>
   e.g., [Oryzias_latipes.ASM223467v1.dna_sm.toplevel.fa.gz](https://ftp.ensembl.org/pub/release-110/fasta/oryzias_latipes/dna/Oryzias_latipes.ASM223467v1.dna_rm.toplevel.fa.gz)
<br><br>
4. Prepare input .bam file

4.1. Use 10X Chromium single cell RNA-seq data set <br>
      Prepare reference gene model using GTF with ‘cellranger mkref’ using the GTF (genemodel.gtf) using the genome assembly (assembly.fna) (see the official guide)
      Map the reads with ‘cellranger count’ using the fastq data in the directory (fastq_dir)
```
cellranger mkref –genome=custom_ref –genes=genemodel.gtf –fasta=assembly.fna
cellranger count –id=run_count –fastq=fastq_dir –transcriptome=custom_ref
```
4.2. Use bulk RNA-seq data<br>
<br>
Map the trimmed reads with [hisat2](http://daehwankimlab.github.io/hisat2/) or equivalent using the genome assembly (assembly.fna)
<br><br>

5. Run peaks2utr using the BAM made by cellranger
```
peaks2utr –gtf genemodel.gtf run_count/outs/possorted_genome_bam.bam -o genemodelNEW.gtf
```
Consider tweaking the parameter --max-distance ('maximum distance in bases that UTR can be from a transcript') depending on maximum distance in bases that UTR can be from a transcript


<br>
6. Analyze the peaks2utr output<br>

 6.1. Open the output file 'summary_stats.txt'

 6.2 Use agat_sp_statisctics.pl from [AGAT (Another GTF/GFF Analysis Toolkit)](https://agat.readthedocs.io/en/latest/index.html)


7. Prepare a reference gene model with ‘cellranger mkref’ inputting the GTF (genemodelNEW.gtf) and the genome assembly (assembly.fna)
```
cellranger mkref –genome=custom_refNEW –genes=genemodelNEW.gtf –fasta=assembly.fna
```

8. Map the reads with ‘cellranger count’ 
```
cellranger count –id=run_countNEW –fastq=fastq_dir –transcriptome=custom_refNEW
```

9. Analyze the cellranger count output

9.1. Confirm the increase of mapping %   (compare this with ##)





## Acknowledgments

We thank Osamu Nishimura at RIKEN BDR and Satoshi Ansai at Kyoto Univ for discussion.

## References
[Bilgic et al., bioRxiv 2023 'Truncated radial glia as a common precursor in the late corticogenesis of gyrencephalic mammals'](https://www.biorxiv.org/content/10.1101/2022.05.05.490846v3)<br>
[Lawson et al., eLife 2020 'An improved zebrafish transcriptome annotation for sensitive and comprehensive detection of cell type-specific genes'](https://elifesciences.org/articles/55792)<br>
[Haese-Hill et al., Bioinformatics 2023 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'](https://academic.oup.com/bioinformatics/article/39/3/btad112/7067741)<br>


+++
title = "1000 Genomes vs TOPMed Imputation in Samples of Diverse Ancestry"
date = 2022-10-10
weight = 20
description = "The hidden downside of better imputation results"
+++

The Stein Lab works with a unique set of human primary cell lines and donated tissue samples from a large set of donors. Each of these donors have been genotyped using standard genotyping platforms yielding over 1.7 million single nucleotide polymorphisms (SNPs). Overlapping these unique samples with individual genotypes from the HapMap3 project shows the diverse ancestry of this dataset.  

<p align="center">
  <img src="/images/imputation/sample_mds.png" alt="HapMap3 MDS Plot" width="500"/>
</p>

Before doing a genetic association analysis, it is common to increase the number of SNPs one wishes to test by first imputing the genotypes that are not measured by the genotyping arrays. In order to do this, we need a reference panel of genomes (or haplotypes) from a large set of individuals with similar ancestry to the samples we wish to impute. Since our samples are from a diverse ancestry, we need a reference panel which contains many representative populations. Two commonly used reference panels are the <a href="https://www.genome.gov/27528684/1000-genomes-project" target="_blank">1000 Genomes</a> panel and the <a href="https://topmed.nhlbi.nih.gov" target="_blank">NHLBI Trans-Omics for Precision Medicine (TOPMed)</a> panel.  

+ 1000 Genomes Phase 3 Version 5
  - 5,008 haplotypes
  - Diverse ancestry
+ TOPMed Freeze 5
  - 125,568 haplotypes
  - Diverse ancestry

Both the 1000 Genomes and TOPMed reference panels contain individuals from a diverse ancestry. However, because the TOPMed reference panel contains significantly more individuals, we expect TOPMed to better impute diverse ancestry genotypes.  

In this analysis, we used the Michigan Imputation Server to impute the genotypes of 326 diverse ancestry samples by starting with 1.7 million genotyped SNPs. At each imputed variant, the imputation quality was assessed with the <a href="https://genome.sph.umich.edu/wiki/Minimac3_Info_File" target="_blank">r<sup>2</sup> statistic</a> (R2 or Rsq in the following graphs). An r<sup>2</sup>>0.3 was considered sufficient for use in downstream association analyses. At first glance, TOPMed yielded more high quality imputed variants than 1000 Genomes:  

+ 1000 Genomes
  - 49 million imputed variants
  - 28 million with an r<sup>2</sup>>0.3
+ TOPMed
  - 239 million imputed variants
  - 32 million with an r<sup>2</sup>>0.3

Furthermore, when we look at variants across a range of minor allele frequencies (MAF), we see that TOPMed consistently yields better imputation quality than 1000 Genomes, more so for variants with an r<sup>2</sup>>0.3.  

<p align="center">
  <img src="/images/imputation/imputed_snps.png" alt="Imputation at various MAF" width="350"/>
  <img src="/images/imputation/imputed_snps_high.png" alt="Imputation at various MAF, high R2" width="350"/>
</p>

Since the TOPMed reference panel contains many more haplotypes than the 1000 Genomes panel, it appears we are getting better imputation of SNP with very low MAF.  

<p align="center">
  <img src="/images/imputation/imputed_snps_low_maf.png" alt="Imputation at low MAF" width="350"/>
</p>

With these encouraging results, we decided to continue the analysis and apply other commonly used variant filters before doing our genetic association tests. Here is where we saw something unexpected. When filtering out variants with an MAF<0.01, TOPMed no longer yielded more imputed variants than 1000 Genomes. The final filter, variants with a sufficiently high Hardy-Weinberg equilibrium p-value (HWE > 1e-6), also showed more variants from the 1000 Genomes imputation.  

<p align="center">
  <img src="/images/imputation/variants_filtered.png" alt="Filtered variants" width="400"/>
</p>

This effect is seen across the genome, the same for variants on any chromosome.  

<p align="center">
  <img src="/images/imputation/variants_filtered_by_chr.png" alt="Filtered variants by chromosome" width="700"/>
</p>

How could this be? It is possible that the TOPMed reference panel has enough genetic diversity to define more haplotypes at any give sequence in the genome. In this toy example, 1000 Genomes may have two haplotypes for the same genomic location that TOPMed defines three haplotypes. Some of the samples that were imputed to the 1000 Genomes rare haplotype may be imputed with greater accuracy to even rarer haplotypes in the TOPMed reference panel.  

<p align="center">
  <img src="/images/imputation/haplotypes.png" alt="Haplotypes" width="600"/>
</p>

In this example, the 1000 Genomes imputed A/C variant has a MAF of 0.0184, which passes the variant filter cutoff of 0.01. The TOPMed imputed variants A/C and nearby A/G both have an MAF of 0.0092, which both fail the MAF cutoff. Indeed, when we look at the TOPMed imputed variants that are not passing the MAF cutoff, they have a lower MAF than the same variants imputed with 1000 Genomes.  

But is this bad? As a data scientist, I hate to throw away data. I want to test more variants for genetic association. Indeed, that was the hope when we sought to use TOPMed instead of 1000 Genomes; we wanted to get more imputed variants by leveraging the greater genetic diversity in the TOPMed haplotypes. In the end, however, we tested fewer variants for genomic association. But this is rigorous science! The imputed variants we used in later association studies were imputed with greater confidence and contained fewer false positives. This in turn leads to fewer false positive genetic associations and a more robust data analysis.  

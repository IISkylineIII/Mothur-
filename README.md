# Complete 16S rRNA Sequencing Analysis Pipeline Using Mothur

## Description
This repository provides a complete pipeline for processing and analyzing 16S rRNA gene sequencing data using **Mothur**. The workflow covers all essential steps from raw sequence quality control and trimming, through clustering, taxonomic classification, and diversity analyses.

This pipeline is designed to facilitate reproducible microbiome studies with Mothur, following best practices for processing Illumina paired-end reads.

---

## Pipeline Overview

1. **Sequence Preparation and Quality Control**
   - Assemble paired-end reads (`make.contigs`)
   - Quality trimming and filtering (`screen.seqs`, `trim.seqs`)

2. **Sequence Dereplication and Alignment**
   - Remove duplicate sequences (`unique.seqs`)
   - Align sequences to a reference 16S rRNA database (`align.seqs`)

3. **Filtering and Pre-Clustering**
   - Remove sequences that do not align properly (`screen.seqs`)
   - Pre-cluster to reduce sequencing errors (`pre.cluster`)

4. **Chimera Detection and Removal**
   - Identify chimeric sequences (`chimera.vsearch` or `chimera.uchime`)
   - Remove chimeras (`remove.seqs`)

5. **Operational Taxonomic Unit (OTU) Clustering**
   - Cluster sequences into OTUs (`cluster.split` or `cluster`)
   - Create OTU table (`make.shared`)

6. **Taxonomic Classification**
   - Assign taxonomy using a reference database (`classify.seqs`)
   - Remove unwanted lineages (e.g., chloroplast, mitochondria)

7. **Diversity and Community Analysis**
   - Alpha diversity metrics (`summary.single`)
   - Beta diversity metrics and ordination (`dist.shared`, `pcoa`)

---

## Usage

This pipeline assumes you have paired-end FASTQ files from an Illumina sequencer.

Example Mothur commands:

```bash
# Step 1: Make contigs (assemble paired reads)
make.contigs(file=stability.files, processors=8)

# Step 2: Screen sequences to remove low quality
screen.seqs(fasta=stability.trim.contigs.fasta, group=stability.contigs.groups, maxambig=0, maxlength=275)

# Step 3: Unique sequences
unique.seqs(fasta=stability.trim.contigs.good.fasta)

# Step 4: Align sequences to SILVA reference
align.seqs(fasta=stability.trim.contigs.good.unique.fasta, reference=silva.nr_v138.align)

# Step 5: Screen sequences based on alignment
screen.seqs(fasta=stability.trim.contigs.good.unique.align, start=13862, end=23444, maxhomop=8)

# Step 6: Filter sequences
filter.seqs(fasta=stability.trim.contigs.good.unique.good.align, vertical=T, trump=.)

# Step 7: Pre-cluster sequences to reduce noise
pre.cluster(fasta=stability.trim.contigs.good.unique.good.filter.fasta, diffs=2)

# Step 8: Chimera detection and removal
chimera.vsearch(fasta=stability.trim.contigs.good.unique.good.filter.precluster.fasta, dereplicate=T)
remove.seqs(fasta=stability.trim.contigs.good.unique.good.filter.precluster.fasta, accnos=stability.trim.contigs.good.unique.good.filter.precluster.denovo.vsearch.accnos)

# Step 9: Classify sequences
classify.seqs(fasta=current, reference=trainset16_022016.pds.fasta, taxonomy=trainset16_022016.pds.tax, cutoff=80)

# Step 10: Remove unwanted lineages
remove.lineage(fasta=current, taxonomy=current, taxon=Chloroplast-Mitochondria-unknown-Archaea-Eukaryota)

# Step 11: Cluster sequences into OTUs
cluster.split(fasta=current, taxonomy=current, splitmethod=classify, taxlevel=4, cutoff=0.03)

# Step 12: Make shared OTU table
make.shared(list=current, group=stability.contigs.good.groups, label=0.03)

# Step 13: Calculate diversity metrics
summary.single(shared=current, calc=coverage-sobs-invsimpson-shannon)

# Step 14: Calculate beta diversity and ordination
dist.shared(shared=current, calc=braycurtis)
pcoa(phylip=current)

```

## Requirements
* Mothur

* Reference databases:
*   SILVA alignment (silva.nr_v138.align)
*   Training set for taxonomy (trainset16_022016.pds.fasta and .tax)

## References
*  Schloss et al. (2009) "Introducing mothur: open-source, platform-independent, community-supported software for describing and comparing microbial communities." Applied and Environmental Microbiology, 75(23), 7537-41.
*  Mothur SOP: https://mothur.org/wiki/miseq-sop/

License
This project is licensed under the MIT License.


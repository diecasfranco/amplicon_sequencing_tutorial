# Amplicon Sequencing Analysis Tutorial

This tutorial guides you through a full workflow for amplicon sequencing data (16S/18S/ITS):

1. Download SRA data
2. Quality check (FastQC)
3. Adapter & quality trimming (Trim Galore)
4. Paired-end merging (PEAR)
5. Dereplication, chimera removal, OTU/ASV clustering (VSEARCH)
6. Taxonomic classification (SINTAX in VSEARCH)

---

## 🛠 Requirements

Install via conda:


conda create -n amplicon-tutorial -c bioconda -c conda-forge fastqc trim-galore pear sra-tools vsearch
conda activate amplicon-tutorial

## 📥 Step 1: Download Data

prefetch SRR12345678
fastq-dump --split-files SRR12345678

## 🔍 Step 2: Quality Check
fastqc SRR12345678_1.fastq SRR12345678_2.fastq -o results/

##✂️ Step 3: Adapter Trimming
trim_galore --paired SRR12345678_1.fastq SRR12345678_2.fastq -o results/

## 🔗 Step 4: Pair-end assembly
pear -f results/SRR12345678_1_val_1.fq.gz \
     -r results/SRR12345678_2_val_2.fq.gz \
     -o results/merged

## 🌀 Step 5: OTU/ASV Workflow with VSEARCH

5.1 Dereplication
vsearch --derep_fulllength results/merged.assembled.fastq \
        --output results/derep.fasta \
        --sizeout

5.2 Denoising (UNOISE3 for zOTUs/ASVs) 
vsearch --cluster_unoise results/derep.fasta \
        --minsize 8 \
        --unoise3 results/zotus.fasta

Alternatively, for OTUs (97% identity):

vsearch --cluster_size results/derep.fasta \
        --id 0.97 \
        --centroids results/otus.fasta

5.3 Chimera Removal
vsearch --uchime3_denovo results/zotus.fasta \
        --nonchimeras results/zotus.nochim.fasta

🧬 Step 6: Taxonomic Classification (SINTAX)

You need a reference database (e.g., SILVA, RDP, UNITE) formatted for VSEARCH.
here you can find the databases https://zenodo.org/records/14930035

vsearch --sintax results/zotus.nochim.fasta \
        --db silva.sintax.fasta \
        --tabbedout results/zotus.sintax \
        --sintax_cutoff 0.8

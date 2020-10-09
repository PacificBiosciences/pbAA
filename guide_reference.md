Introduction
============

Amplicon Analysis with Guided Clustering (`pbaa`) requires a file of
reference sequences in FASTA format to use as a the cluster guide. The
program can accept any valid FASTA-format sequence file, but using a
well curated sequence file for the cluster guide will provide for
faster, higher quality results. This document offers some pointers on
how to curate a guide-sequence FASTA file for optimal performance, using
the construction of a reference for an 11-locus HLA panel as an example.

Protocol
--------

1.  Acquire FASTA sequences for each expected or desired Locus

The initial collection of reference sequences should ideally correspond
to the exact region being amplified for each locus, but small
differences are acceptable. The initial collection of sequences should
also span the widest possible range of template sequences, and should be
genomic rather than from cDNA, to ensure maximum sensitivity. In
particular, we recommend including any known closely related
pseudo-genes, for example HLA-DPB2 in addition to HLA-DPB1. Often there
are sequences in nature not currently in any database that are fully
functional, but are more similar to known alleles of the pseudogene, and
this ensures they too will be captured.

For HLA, we recommend using the genomic FASTA files from the public IMGT
database (<ftp://ftp.ebi.ac.uk/pub/databases/ipd/imgt/hla/fasta/)>,
which is both the most comprehensive public database of HLA alleles,
.For our purposes, we wish to cover the core MHC Class I and II genes,
so we use the following files: `A_gen.fasta, B_gen.fasta, C_gen.fasta,
DPA1_gen.fasta, DPB1_gen.fasta, DPB2_gen.fasta, DQA1_gen.fasta,
DQB1_gen.fasta, DRB1_gen.fasta, DRB345_gen.fasta`

2.  Concatenate FASTA files from different targets of interest

The final selection of sequences must be uploaded to SMRT-link or passed
to `pbaa` as a single FASTA file. In addition, it's usually more
efficient to filter all of the sequences at once, for the most
aggressive possible filtering.

```
$ cat *_gen.fasta > combined.fasta
$ grep '>' combined.fasta | head
>HLA:HLA00001 A*01:01:01:01 3503 bp
>HLA:HLA02169 A*01:01:01:02N 3291 bp
>HLA:HLA14798 A*01:01:01:03 3503 bp
>HLA:HLA15760 A*01:01:01:04 3087 bp
>HLA:HLA16415 A*01:01:01:05 3321 bp
>HLA:HLA16417 A*01:01:01:06 3097 bp
>HLA:HLA16436 A*01:01:01:07 3321 bp
>HLA:HLA16651 A*01:01:01:08 3121 bp
>HLA:HLA16652 A*01:01:01:09 3340 bp
>HLA:HLA16653 A*01:01:01:10 3121 bp
```

3.  Filter out highly similar guide sequences

Most comprehensive sequence databases, like IMGT, contain many highly
similar alleles that differ by only a few nucleotides. However even
sequence differences of \>5% are often unnecessary for correctly
clustering most data with `pbaa`, but can substantially slow down the
clustering process. We therefore recommend filtering out the most
similar sequences from the reference guide to speed up the clustering
process. We suggest using **cd-hit-est** from the CD-HIT suite of tools
(<https://github.com/weizhongli/cdhit/wiki/3.-User's-Guide>) to filter
the combined FASTA file down to a subset of representative sequences of
approximately \~90-95% similarity.

```$ cd-hit-est -c 0.9 -i combined.fasta -o filtered.fasta```

4.  Add cluster group designations to each remaining sequence

Once filtered there will still often be multiple sequences from the same
loci in the cluster guide, particularly highly diverse loci like MHC
Class I. We want some way to designate that certain sequences belong
together -- that they represent the same underlying gene -- so that `pbaa`
doesn't create duplicate results. The FASTA format specification
requires that each sequence id in a file be unique, so instead we place
the name of the locus for each sequence in the cluster guide FASTA at
the end of it's header, which `pbaa` will use if available.

```
$ sed -E 's/>([^:]*):([^ ]*) ([^\*])(.*)/>\2 \3\4|\1-\3/;s/[*: ]/_/g' filtered.fasta > guides.fasta
$ grep '>' guides.fasta | head
>HLA00001_A_01_01_01_01_3503_bp|HLA-A
>HLA02169_A_01_01_01_02N_3291_bp|HLA-A
>HLA14798_A_01_01_01_03_3503_bp|HLA-A
>HLA15760_A_01_01_01_04_3087_bp|HLA-A
>HLA16415_A_01_01_01_05_3321_bp|HLA-A
>HLA16417_A_01_01_01_06_3097_bp|HLA-A
>HLA16436_A_01_01_01_07_3321_bp|HLA-A
>HLA16651_A_01_01_01_08_3121_bp|HLA-A
>HLA16652_A_01_01_01_09_3340_bp|HLA-A
>HLA16653_A_01_01_01_10_3121_bp|HLA-A
```


5.  Upload the final cluster-guide FASTA to SMRTLink

Please see the section on "Adding Reference Guides" in the documentation
for Long Amplicon Analysis with Guided Clustering.

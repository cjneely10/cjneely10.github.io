---
title: 'MetaSanity - Output'
date: 2020-02-01
permalink: /posts/2019/10/MetaSanity-Output/
tags:
  - MetaSanity
  - demos
---

In a [previous post](https://cjneely10.github.io/posts/2019/10/MetaSanity-Demo-BioMetaDB/), we walked through several examples of using **BioMetaDB** - a Bio-focused command-line SQL wrapper package - to query the results of our **MetaSanity** pipeline runs. The contents of each project is self-contained, making it very easy for users to focus their attention on getting specific information.

<img src="https://cjneely10.github.io/files/Figure1-Workflow.png" style="border:2px solid black">

Given the complexity and volume of this application, we will walk through the raw and parsed output from each of the programs that **MetaSanity** runs. This information is also available as a <a href="https://1drv.ms/p/s!AliNbVaBWXheiagJZmV_9pGlzOoXYw" target="_blank">PowerPoint presentation</a> (views best using PowerPoint Online).

# MetaSanity output

<pre><code>out/
├── metagenome_evaluation.tsv
├── metagenome_functions.tsv
├── checkm_results/
├── fastani_results/
├── gtdbtk_results/
├── prodigal_results​/
├── prokka_results​/
├── interproscan_results​/
├── kegg_results​/
├── peptidase_results​/
├── virsorter_results/
├── mag1.metagenome_annotation.tsv
...</code></pre>

The first two `.tsv` files contain the parsed results of the **MetaSanity** pipelines. These are simple tab-delimited text files. **PhyloSanity** results are contained in `metagenome_evaluation.tsv`. **FuncSanity** results are in a set of files. `metagenome_functions.tsv` contains functional and metabolic information for all genomes that were studies. A file ending with the extension `.metagenome_annotation.tsv` will be generated for each genome that was analyzed.

The remaining directories contain the raw output from each of the programs that were incorporated into the user's annotation pipeline. The resulting file(s) are parsed for specific information to generate the workflow's results.

## PhyloSanity

### CheckM

`checkm_results/checkm_lineageWF_results.qa.txt`

### FastANI

`fastani_results/fastani_results.txt`

### GTDB-Tk

`gtdbtk_results/GTDBTK.bac120.summary.tsv`


## FuncSanity

For each MAG that is analyzed, a series or files may be generated.

### Prodigal

<pre><code>prodigal_results/
├── mag1.mrna.fna
├── mag1.protein.faa
├── mag1.txt</code></pre>

### Prokka

<pre><code>prokka_results/
├── mag1/
	├── mag1.tsv  # Original output
	├── mag1.prk.tsv.amd  # Parsed output for BioMetaDB
	├── mag1.prk.tsv.prokka.nucl  # RNA features
├── diamond/
	├── mag1.prk-to-prd.tsv  # Protein annotations</code></pre>

The remaining files in each directory are the outputs from each Prokka run.

### InterProScan

<pre><code>interproscan_results/
├── mag1.tsv  # Original output
├── mag1.amended.tsv  # Parsed output</code></pre>


### KoFamScan and KEGG-Decoder

<pre><code>kegg_results/
├── biodata_results/  # KEGG-Decoder-derived metabolic pathway estimation counts
	├── KEGG.final.tsv
	├── KEGG.expander.tsv 
	├── KEGG.decoder.tsv  
	├── decode-expand_heatmap.svg  # Heatmaps
	├── hmm_heatmap.svg
├── combined_results/
	├── combined.ko  # kofamscan matches for KEGG-Decoder input
	├── combined.protein  # gene calls for KEGG-Decoder input
├── kofamscan_results/
	├── mag1.tsv  # KO matches per protein
	├── mag1.detailed  # Detailed kofamscan output
	├── mag1.amended.tbl  # Parsed output for BioMetaDB</code></pre>

### MEROPS

<pre><code>peptidase_results/
├── mag1.merops.tsv  # MEROPS matches
├── mag1.pfam.tsv  # MEROPS matches by PFam id
├── merops/
	├── mag1.merops.protein.faa  # MEROPS protein matches
	├── hmmconvert_data/  # HMM prep
	├── hmmsearch_results/  # HMMSearch output</code></pre>

### CAZy

<pre><code>peptidase_results/
├── cazy/
	├── hmmsearch_results/  # HMMSearch output
	├── mag1.cazy_assignments.tsv  # Raw counts of CAZy HMM matches
	├── mag1.cazy_assignments.byprot.tsv  # Protein annotations</code></pre>

### PSORTb and SignalP

<pre><code>peptidase_results/
├── psortb_results/
	├── mag1.tbl  # Raw PSORTb output
├── signalp_results/
	├── mag1.signalp.tbl  # Raw SignalP output</code></pre>

### Extracellular Peptidase

<pre><code>peptidase_results/
├── mag1.pfam.by_prot.tsv  # PFam annotations for peptidase results</code></pre>

### VirSorter

<pre><code>virsorter_results/
├── mag1
	├── virsorter_results/
		├── VIRSorter_global-phage-signal.csv  # Original output
		├── mag1.VIRSorter_adj_out.tsv  # Parsed output for BioMetaDB</code></pre>

The remaining files in each `virsorter_results` directory are the raw outputs of the VirSorter run for each MAG.

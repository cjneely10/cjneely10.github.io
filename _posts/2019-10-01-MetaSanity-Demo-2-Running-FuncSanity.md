---
title: 'MetaSanity Demo - Running FuncSanity'
date: 2019-10-01
permalink: /posts/2019/10/MetaSanity-Demo-2-Running-FuncSanity/
tags:
  - MetaSanity
  - demos
---

In this blog, we will continue our work from the [first demo blog](https://cjneely10.github.io/posts/2019/10/MetaSanity-Demo-1-Running-PhyloSanity/). Previously, we had evaluated a set of metagenomic assembled genomes (MAGs) for completion and contamination. We then queried our dataset for high quality, non-redundant genomes, and stored these to a separate directory.

Now, we can begin annotating each genome.

Part 2 - FuncSanity
======

We are continuing our work in the directory `~/test-run`.

Config file preparation
------
Copy the default configuration file from your program package into your project directory.

`cd ~/test-run && cp ~/MetaSanity/Config/Docker/FuncSanity.ini ./`

The config file `FuncSanity.ini` can be used as-is; however, users may add additional flags or edit existing flags as needed. Again, we'll edit the config file using `nano FuncSanity.ini`. The file `Complete-FuncSanity.ini` in the project package contains the currently supported flags that are available to each program in the **FuncSanity** pipeline.

<pre><code># Default config file for running the FuncSanity pipeline
# DO NOT edit any PATH, DATA, or DATA_DICT variables
# Users are recommended to edit copies of this file only

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# The following pipes **MUST** be set

[PRODIGAL]
PATH = /usr/bin/prodigal
-p = meta
FLAGS = -m

[HMMSEARCH]
PATH = /usr/bin/hmmsearch
-T = 75

[HMMCONVERT]
PATH = /usr/bin/hmmconvert

[HMMPRESS]
PATH = /usr/bin/hmmpress

[BIOMETADB]
--db_name = MSResults
FLAGS = -s

[DIAMOND]
PATH = /usr/bin/diamond


# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# The following sections may optionally be set
# Ensure that the entire section is valid,
# or deleted/commented out, prior to running pipeline


# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Peptidase annotation

[CAZY]
DATA = /home/appuser/Peptidase/dbCAN-fam-HMMs.txt

[MEROPS]
DATA = /home/appuser/Peptidase/MEROPS.pfam.hmm
DATA_DICT = /home/appuser/Peptidase/merops-as-pfams.txt

#[SIGNALP]
#PATH = /home/appuser/signalp/signalp

#[PSORTB]
#PATH = /usr/bin/psortb

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# KEGG pathway annotation

[KOFAMSCAN]
PATH = /usr/bin/kofamscan
--cpu = 1

[BIODATA]
PATH = /home/appuser/BioData/KEGGDecoder

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# PROKKA

[PROKKA]
PATH = /usr/bin/prokka
FLAGS = --addgenes,--addmrna,--usegenus,--metagenome,--rnammer,--force
--evalue = 1e-10
--cpus = 1

# # - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# # InterproScan

#[INTERPROSCAN]
#PATH = /usr/bin/interproscan
## Do not remove this next flag
#--tempdir = /home/appuser/interpro_tmp
#--applications = TIGRFAM,SFLD,SMART,SUPERFAMILY,Pfam,ProDom,Hamap,CDD,PANTHER
#--cpu = 1
#FLAGS = --goterms,--iprlookup,--pathways

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# VirSorter

[VIRSORTER]
PATH = /home/appuser/virsorter-data
--db = 2
--ncpu = 1</code></pre>

As before, I will now edit this configuration file to match the specifications of my system. For this analysis, I am not interested in predicting protein domains, so I will leave InterProScan commented out.

I interested in predicting extracellularity, so I will also remove the `#` characters that precede the two PSORTb lines. 
I will also uncomment the SignalP section by removing the `#` characters preceding the two SignalP lines. 

I will raise the thread count on each applicable program to better match my system, but no other flags need to be changed.

<pre><code># Docker/FuncSanity.ini
# Default config file for running the FuncSanity pipeline
# DO NOT edit any PATH, DATA, or DATA_DICT variables
# Users are recommended to edit copies of this file only

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# The following **MUST** be set

[PRODIGAL]
PATH = /usr/bin/prodigal
-p = meta
FLAGS = -m

[HMMSEARCH]
PATH = /usr/bin/hmmsearch
-T = 75

[HMMCONVERT]
PATH = /usr/bin/hmmconvert

[HMMPRESS]
PATH = /usr/bin/hmmpress

[BIOMETADB]
--db_name = MSResults
FLAGS = -s

[DIAMOND]
PATH = /usr/bin/diamond


# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# The following sections may optionally be set
# Ensure that the entire section is valid,
# or deleted/commented out, prior to running pipeline


# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Peptidase annotation

[CAZY]
DATA = /home/appuser/Peptidase/dbCAN-fam-HMMs.txt

[MEROPS]
DATA = /home/appuser/Peptidase/MEROPS.pfam.hmm
DATA_DICT = /home/appuser/Peptidase/merops-as-pfams.txt

[SIGNALP]
PATH = /home/appuser/signalp/signalp

[PSORTB]
PATH = /usr/bin/psortb

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# KEGG pathway annotation

[KOFAMSCAN]
PATH = /usr/bin/kofamscan
--cpu = 10

[BIODATA]
PATH = /home/appuser/BioData/KEGGDecoder

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# PROKKA

[PROKKA]
PATH = /usr/bin/prokka
FLAGS = --addgenes,--addmrna,--usegenus,--metagenome,--rnammer,--force
--evalue = 1e-10
--cpus = 10

# # - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# # InterproScan

#[INTERPROSCAN]
#PATH = /usr/bin/interproscan
## Do not remove this next flag
#--tempdir = /home/appuser/interpro_tmp
#--applications = TIGRFAM,SFLD,SMART,SUPERFAMILY,Pfam,ProDom,Hamap,CDD,PANTHER
#--cpu = 1
#FLAGS = --goterms,--iprlookup,--pathways

# - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# VirSorter

[VIRSORTER]
PATH = /home/appuser/virsorter-data
--db = 2
--ncpu = 10</code></pre>

Running FuncSanity
------
The project directory `test-run` should resemble the following structure, substituting your own values:

<pre><code>test-run/
├── genomes/
├── FuncSanity.ini
├── eval.log
├── out/
├── MSResults/
└── PhyloSanity.ini</code></pre>

At this point, we can choose to annotate the entire dataset, or just the high-quality, non-redundant dataset. For the purposes of this blog, we will annotate the entire dataset.

For the purpose of protein localization predictions and Prokka annotations, by default **FuncSanity** assumes that each genome is derived from a Gram-negative bacteria. Users can define genomes as Gram-positive (for both PSORTb and SignalP archaea are assumed to be Gram-positive). This info should be provided in a separate file and passed to `MetaSanity` from the command line using the `-t` flag. As long as each genome is defined separately, **FuncSanity** can run any combination of gram +/- and bacteria/archaea. The format of this file should include the following info, separated by tabs, with one line per relevant FASTA file:

<pre><code>[fasta-file]\t[Bacteria/Archaea]\t[gram+/gram-]\n</code></pre> 

Example:

`example-fasta-file.fna\tArchaea\tgram+\n`

Fortunately, this file can be automatically generated using the `generate-typefile.py` script in the `MetaSanity/Accessories/` folder of the **MetaSanity** installation. Assuming that the directory contents are on your path, generate the type-file `typefile.list` by using:

`generate-typefile.py MSResults`

From within this directory, run the **FuncSanity** pipeline. You may redirect log messages to a separate file.

`MetaSanity -d genomes/ -c FuncSanity.ini FuncSanity 2>annot.log`

Depending on your system, this may run for several hours.

Once **FuncSanity** is complete, the project directory structure will resemble the following:

<pre><code>test-run/
├── annot.log	
├── genomes/
├── FuncSanity.ini
├── eval.log
├── out/
├── MSResults/
└── PhyloSanity.ini</code></pre>

**FuncSanity** added completed outputs into the existing `out/` directory and added its output to that directory. At this point, the **MetaSanity** pipeline is complete. If I decided to later incorporate PSORTb or InterProScan, I can simply update my `FuncSanity.ini` config file and rerun with the same command as above.

Within the `out/` directory, each input genome will have its own tab-delimited output file that ends in `.annotation.tsv`. If the peptidase annotation + extracellular prediction were run, then summary data is in the `peptidase_results/combined_results` folder. Also, if the KEGG pathway annotation was run, then summary data is in `kegg_results/biodata_results`, including heatmap visualizations of the metabolic pathways calculated as part of KEGG-Decoder. Any additional information will be in the file `functions.tsv`. All of this data is populated into the **BioMetaDB** project stored in the `MSResults` directory.

We can generate a quick summary of this data using a command-line query:

`dbdm SUMMARIZE -c MSResults`

Note that we can omit the `-c MSResults` if we are working in a directory that has no other **BioMetaDB** projects.

`dbdm SUMMARIZE`

Depending on the number of genomes, a good deal of output will display on your terminal. This is great, but it would be very helpful to be able to work with and manipulate this data. **BioMetaDB** steps in to provide a simple command-line interface for accessing your data quickly and easily.

Summary of **BioMetaDB** project output
------

Below is an example of a complete annotation, involving all programs running default options. InterProScan counts are based on total times each domain was identified. Phage and prophage summaries are present, indicating at least one viral contig or prophage. If VirSorter identified no viral matches, then these columns would not be present.

<pre><code>SUMMARIZE:	View summary of all tables in database
 Project root directory:	MSResults
 Name of database:		MSResults.db

*******************************************************************************************
	     Table Name:	tobg-cpc-96 
  Number of Records:	1957/2656      

	        Database	Average             	Std Dev     

	  phage_contig_1	0.000               	0.000       
	  phage_contig_2	0.000               	0.000       
	  phage_contig_3	0.000               	0.000       
	      prophage_1	0.000               	0.000       
	      prophage_2	0.000               	0.000       
	      prophage_3	0.000               	0.000       
-------------------------------------------------------------------------------------------

	        Database	Most Frequent       	Number    	Total Count 

	            cazy	GT2_Glycos_transf...	7         	53          
	             cdd	cd02440             	26        	808         
	           hamap	MF_00600 IPR00184...	3         	331         
	is_extracellular	False               	73        	77          
	              ko	K02456 general se...	21        	1072         
	     merops_pfam	PF00082             	7         	77          
	         panther	PTHR43289           	15        	1631        
	            pfam	PF00005 IPR003439...	32        	1764        
	          prodom	PD001179            	5         	34          
	          prokka	ybhF_1 COG1131 pu...	11        	847
	            sfld	SFLDS00029 IPR007...	6         	23          
	           smart	SM00028 IPR019734...	79        	415         
	     superfamily	SSF52540 IPR02741...	154       	1628        
	         tigrfam	TIGR02532 IPR0129...	23        	567         
-------------------------------------------------------------------------------------------</code></pre>

**MetaSanity** generates a single summary database based on the results of the entire pipeline. The complete output contains elements such as the raw count data from peptidase annotation and pathway completion estimates from KEGG-Decoder. 

**MetaSanity** also creates a database table for each genome. Each genome table contains all putative gene coding sequences and computed annotations.

In the [next blog](https://cjneely10.github.io/posts/2019/10/MetaSanity-Demo-3-BioMetaDB/), we will explore **BioMetaDB** and its core functionality, which may be the best reason to use **MetaSanity**.

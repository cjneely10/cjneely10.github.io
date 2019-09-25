---
title: 'MetaSanity Demo part 2'
date: 2019-09-06
permalink: /posts/2019/09/MetaSanity-demo-part-2/
tags:
  - MetaSanity
  - demos
---

In this blog, we will continue our work from the [first demo blog](https://cjneely10.github.io/posts/2019/09/MetaSanity-demo-part-1/). Previously, we had evaluated a set of metagenomic assembled genomes (MAGs) for completion and contamination. We then queried our dataset for high quality, non-redundant genomes, and stored these to a separate directory.

Now, we can begin structurally and functionally annotating each genome.

Project preparation
======
We are continuing our work in the directory `$HOME/test-run`. After the last blog, the contents of this directory should resemble the following:

<code>test-run/
├── genomes/
├── HighQuality/
├── eval.log
├── out/
├── Metagenomes/
└── PhyloSanity.ini</code>

Part 2 - FuncSanity
======

Config file preparation
------
Copy the default configuration file from your program package into your project directory.

`cd $HOME/test-run && cp /path/to/MetaSanity/Config/Docker/FuncSanity.ini ./`

The config file `FuncSanity.ini` can be used as-is; however, users may add additional flags or edit existing flags as needed. Edit the config file using `nano FuncSanity.ini`.

![](https://cjneely10.github.io/files/FuncSanity-pre-1.png)

![](https://cjneely10.github.io/files/FuncSanity-pre-2.png)

As before, I will now edit this configuration file to match the specifications of my system. For this analysis, I am not interested in predicting protein domains, and I only want to use SignalP in my extracellular peptidase annotation.

The `FuncSanity.ini` config file currently has SignalP commented out, so I will uncomment this section by removing the two `#` characters preceding the SignalP lines.

InterProScan is set an an optional program and thus its section is commented out by default.

I will raise the thread count on each applicable program to better match my server, but no other flags need to be changed.

![](https://cjneely10.github.io/files/FuncSanity-post-1.png)

![](https://cjneely10.github.io/files/FuncSanity-post-2.png)

Running FuncSanity
------
The project directory `test-run` should resemble the following structure:

<pre><code>test-run/
├── genomes/
├── FuncSanity.ini
├── HighQuality/
├── eval.log
├── out/
├── Metagenomes/
└── PhyloSanity.ini</code></pre>

At this point, we can choose to annotate the entire dataset, or just the high-quality, non-redundant dataset. For the purposes of this blog, we will annotate the entire dataset.

**FuncSanity** assumes that each genome represents a Gram negative bacteria. Users may also search for gram positive bacteria and archaea. This info for relevant genomes should be provided in a separate file and passed to `MetaSanity` from the command line using the `-t` flag. Pipeline searches can be run with any combination of gram +/- and bacteria/archaea. The format of this file should include the following info, separated by tabs, with one line per relevant fasta file passed to pipeline:

<pre><code>[fasta-file]\t[Bacteria/Archaea]\t[gram+/gram-]\n</code></pre> 

Example:

`example-fasta-file.fna\tArchaea\tgram+\n`

From within this directory, run the **FuncSanity** pipeline. You may redirect log messages to a separate file.

`MetaSanity -d genomes/ -c FuncSanity.ini FuncSanity 2>annot.log`

Depending on your system, this may run for several hours.

Once **FuncSanity** is complete, the project directory structure will resemble the following:

<pre><code>test-run/
├── annot.log	
├── genomes/
├── FuncSanity.ini
├── HighQuality/
├── eval.log
├── out/
├── Metagenomes/
└── PhyloSanity.ini</code></pre>

**FuncSanity** ran its pipeline using the existing `out/` directory and added its output to that directory. It also generated a new citations file. At this point, the **MetaSanity** pipeline is complete. If I decided to later incorporate PSORTb or InterProScan, I can simply update my `FuncSanity.ini` config file and rerun with the same command as above.

Within the `out/` directory, each input genome will have its own tab-delimited output file that ends in `.metagenome_annotation.tsv`. If the peptidase annotation was run, then summary data is in the `peptidase_results/combined_results` folder. Also, if the KEGG pathway annotation was run, then summary data is in `kegg_results/biodata_results`. However, all of this data was populated into out `Metagenomes` **BioMetaDB** project.

We can generate a quick summary of this data using a command-line query:

`dbdm SUMMARIZE -c Metagenomes`

Depending on the number of genomes, a good deal of output will display on your terminal. This is great, but it would be very helpful to be able to work with and manipulate this data. **BioMetaDB** steps in to provide a simple command-line interface for accessing your data quickly and easily.

Summary of **BioMetaDB** project output
------
**MetaSanity** generates a single summary database table based on the summary results of the entire pipeline. Within this table, raw count data from Peptidase pipelines are stored. Pathway completion estimates are provided from the KEGG-Decoder workflow. 

**MetaSanity** also creates a database table for each genome. Each table will contain all putative gene coding sequences and all available annotations.

In the [next blog](https://cjneely10.github.io/posts/2019/09/MetaSanity-demo-part-3/), we will explore **BioMetaDB** and its core functionality, which is (in my opinion) the biggest reason to use **MetaSanity**.

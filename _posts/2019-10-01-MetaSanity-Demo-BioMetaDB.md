---
title: 'MetaSanity Demo - BioMetaDB'
date: 2019-10-01
permalink: /posts/2019/10/MetaSanity-Demo-BioMetaDB/
tags:
  - MetaSanity
  - demos
---

In this blog, we will continue our work from the [previous demo blog](https://cjneely10.github.io/posts/2019/10/MetaSanity-Demo-Running-FuncSanity/). We had just completed both steps of the **MetaSanity** package, and we now have an output directory filled with output from each program and parsed tab-delimited data files (TSVs).

We also have a working **BioMetaDB** project that contains all of our results, but better.

Now, how do we actually use it?

Part 3 - BioMetaDB
======

Goals
------
In this blog, we will explore the completed results of **MetaSanity**. Specifically,

1. Link evaluation with annotation.
2. Link putative metabolic function with annotation.
3. Link evaluation and putative metabolic function.
4. Create comprehensive query statements.

About
------
**BioMetaDB**, accessible through the script `MetaSanity/BioMetaDB/dbdm.py`, part of the MetaSanity installation, is a database generator. This package manages FASTA files and data files that describe them, handling all CRUD operations and providing a simple command-line interface.


**MetaSanity** output
------
If you've been following this blog, both pipelines in **MetaSanity** are now complete at this point, and we have a **BioMetaDB** project names `Metagenomes` to use. Within this directory are two summary database tables as well as an extra table for each genome that was processed. **PhyloSanity** outputs the summary table named `evaluation`, and **FuncSanity** outputs the summary table named `functions` if either the KEGG Pathway annotation or Peptidase annotation were completed.

You can view these tables easily using:

`dbdm -c Metagenomes SUMMARIZE -t evaluation`

Note that we can omit the `-c Metagenomes` if we are working in a directory that has no other **BioMetaDB** projects.

`dbdm SUMMARIZE`

<pre><code>SUMMARIZE: View summary of all tables in database
 Project root directory:  Metagenomes
 Name of database:    Metagenomes.db

*******************************************************************************************
       Table Name:  evaluation  
Number of Records:  10/10        

          Database  Average               Std Dev     

        completion  92.882                4.817       
     contamination  3.365                 2.574       
-------------------------------------------------------------------------------------------

          Database  Most Frequent         Number      Total Count 

            _class  Phycisphaerales       4           10          
            _order  SM1A02                4           10          
            domain  Bacteria              10          10          
            family  Gimesia               2           10          
             genus  Gimesia               2           10          
       is_complete  True                  10          10          
   is_contaminated  False                 7           10          
  is_non_redundant  True                  10          10          
           kingdom  Planctomycetota       10          10          
            phylum  Phycisphaerae         4           10          
  redundant_copies  []                    10          10          
           species  sp002684655           1           10          
-------------------------------------------------------------------------------------------</code></pre>

`dbdm -c Metagenomes SUMMARIZE -t functions`

While this is definitely useful, **BioMetaDB** offers a far wider range of query options. You can find additional information [here](https://github.com/cjneely10/BioMetaDB) and additional examples [here](https://github.com/cjneely10/BioMetaDB/tree/master/Example).

Were we to run `dbdm -h`, we would see a large help statement that explains how to use all portions of **BioMetaDB**. **MetaSanity** does most of the database creation and updating that is needed, leaving for us a finalized database that is ready for analysis. We can always add additional tables, or remove tables as needed. 

For this blog, we will focus on the `SUMMARIZE` option of **BioMetaDB**, which provides us an interface for handling all project queries.

Before we start working through this blog's goals, let's explore a couple of examples.

Simple examples
------

View a summary of all genomes that were deemed high-quality and non-redundant by **PhyloSanity**.

`dbdm -c Metagenomes SUMMARIZE -t evaluation -q hqnr`

View a summary of gene calls for TOBG-CPC-51 that were given annotations by **FuncSanity**.

`dbdm -c Metagenomes SUMMARIZE -t tobg-cpc-51 -q annotated`

<pre><code>SUMMARIZE:   View summary of all tables in database
 Project root directory:    Metagenomes
 Name of database:      Metagenomes.db

**********************************************************************************************
       Table Name:      tobg-cpc-51 
Number of Records:      1749/4167      

               Database Average                 Std Dev     

    phage_contig_1      0.000                   0.000       
    phage_contig_2      0.002                   0.072       
    phage_contig_3      0.000                   0.000       
        prophage_1      0.000                   0.000       
        prophage_2      0.000                   0.000       
        prophage_3      0.000                   0.000       
----------------------------------------------------------------------------------------------

            Database    Most Frequent           Number      Total Count 

                cazy    GT41                    23          106         
    is_extracellular    False                   1715        1746        
                  ko    K08884                  25          1387        
         merops_pfam    PF00082                 9           132         
              prokka    atsA_12                 19          567         
-------------------------------------------------------------------------------------------</code></pre>

View a summary of all genomes that have at least a partial predicted glycolysis or chemotaxis pathway.

`dbdm -c Metagenomes SUMMARIZE -t functions -q "glycolysis > 0 OR chemotaxis > 0"`

View a summary of gene calls for TOBG-CPC-997 that have both KO and MEROPS-Pfam annotations.

`dbdm -c Metagenomes SUMMARIZE -t tobg-np-997 -q "ko_annot AND merops_pfam_annot"`

<pre><code>SUMMARIZE:   View summary of all tables in database
 Project root directory:    Metagenomes
 Name of database:      Metagenomes.db

**********************************************************************************************
         Table Name:    tobg-cpc-51 
  Number of Records:    63/4167      

               Database Average                 Std Dev     

----------------------------------------------------------------------------------------------

            Database    Most Frequent           Number      Total Count 

                cazy    CE10                    1           1           
    is_extracellular    False                   53          63          
                  ko    K07263                  3           63          
         merops_pfam    PF00117                 4           63          
              prokka    map_1                   1           17          
-------------------------------------------------------------------------------------------</code></pre>

View a list of all unique KO values assigned to TOBG-CPC-96 gene calls.

`dbdm -c Metagenomes SUMMARIZE -t tobg-cpc-96 -u ko`

View contigs in the TOBG-CPC-85 genome that were identified as likely being of phage origin.

`dbdm -c Metagenomes SUMMARIZE -t tobg-cpc-85 -q "phage_contig_1 > 0"`

View a list of all of the tables in the database

`dbdm -c Metagenomes SUMMARIZE -v t`

View a list of all of the columns in each table of the database.

`dbdm -c Metagenomes SUMMARIZE -v c`


Great! Even on a simple level, we can begin to see some potential utility. If we were to couple any of the above examples with `-w out_fna` or `-x prefix`, then we could store all matching FASTA records to an external directory, or generate a summary tab-delimited data file of all database table information, respectively.


Quick notes
------

- For genome tables (ex. tobg-cpc-85, etc.), we can do the following in our queries:
    - Add `_annot` to columns in the 2nd section of our database table - `ko_annot`, `prokka_annot`, etc.
    - Search for gene calls with at least one annotation using `annotated`.
- In the evaluation table, we can use the following:
    - Use `hqnr` to search for high quality, non-redundant genomes.
- In every other case, we treat data with the following considerations:
    - Quote string data within the query, ex: `-q "cazy == 'GT41'"`.
    - Treat columns that begin with `is_` as Boolean, ex: `-q is_complete`.
- Use simple comparison statements as needed
    - `==`, `<`, `>`, `<=`, `>=`, `!=`, `AND`, `OR`, `NOT`, `LIKE`
- Use regex expressions for strings
    - `-q "cazy LIKE 'GT%'"` would locate CAZy annotations that begin with `GT`.
- Ensure proper query formatting
    - Provide adequate spacing, as needed:
        - `"is_completeANDdomain == 'Bacteria'"` would fail, whereas `"is_complete AND domain == 'Bacteria'"` is valid.
        - `"is_complete~>ko_annot"` can lead to issues, but `"is_complete ~> ko_annot"` is perfect.
            - We will cover the `~>` **query operator** in a later section of this blog.


`dbdm SUMMARIZE`
------

Viewing a BioMetaDB project requires, at the very minimum, a valid **BioMetaDB** project. Aside from that, users are able to provide additional flags as needed. Let's peek at the `dbdm`'s help statement that relates to `SUMMARIZE`:

<pre><code>...
SUMMARIZE: Summarize project and query data. Write records or metadata to file
    (Req:  --config_file --view --query --table_name --alias --write --write_tsv --unique)
...
  -t TABLE_NAME, --table_name TABLE_NAME
                        Name of database table
  -c CONFIG_FILE, --config_file CONFIG_FILE
                        /path/to/BioMetaDB-project-directory
  -a ALIAS, --alias ALIAS
                        Provide alias for locating and creating table class
  -v VIEW, --view VIEW  View (c)olumns or (t)ables with SUMMARIZE                        
  -q QUERY, --query QUERY
                        evaluation ~> genome; function -> gen; eval >> fxn; eval ~> fxn -> gen;
  -u UNIQUE, --unique UNIQUE
                        View unique values of column using SUMMARIZE                        
  -w WRITE, --write WRITE
                        Write FASTA records from SUMMARIZE to outfile
  -x WRITE_TSV, --write_tsv WRITE_TSV
                        Write table record metadata from SUMMARIZE to outfile
  -p PATH, --path PATH  New path for moving project in MOVE command</code></pre>

Let's start working on some of these goals!


Link evaluation with annotation
------

Working with MAGs requires researchers to take added additional care to ensure genome quality and minimize genome redundancy. **BioMetaDB** makes it very easy to handle these queries directly from the command line. Consider the following command:

`dbdm -c Metagenomes SUMMARIZE -t evaluation -q is_complete`

<pre><code>SUMMARIZE:   View summary of all tables in database
 Project root directory:    Metagenomes
 Name of database:      Metagenomes.db

*******************************************************************************************
         Table Name:    evaluation  
  Number of Records:    10/10        

            Database    Average                 Std Dev     

          completion    92.882                  4.817       
       contamination    3.365                   2.574       
-------------------------------------------------------------------------------------------

            Database    Most Frequent           Number      Total Count 

              _class    Phycisphaerales         4           10          
              _order    SM1A02                  4           10          
              domain    Bacteria                10          10          
              family    Gimesia                 2           10          
               genus    Gimesia                 2           10          
         is_complete    True                    10          10          
     is_contaminated    False                   7           10          
    is_non_redundant    True                    10          10          
             kingdom    Planctomycetota         10          10          
              phylum    Phycisphaerae           4           10          
    redundant_copies    []                      10          10          
             species    sp002683825             1           10          
-------------------------------------------------------------------------------------------</code></pre>

As expected, this command will query the evaluation table for genomes whose Boolean 'is_complete' value is set to True - e.g., genomes that were determined to be complete as part of the **PhyloSanity** pipeline. Coupling this with `-w out_fna` and/or `-x complete_genomes.tsv` would output all genomes to a directory named `out_fna` and provide genome evaluation summaries in the file `complete_genomes.tsv`, respectively.

Handy. Let's introduce a **query operator** - a specific delimiter that allows us to link our database tables. Consider if we wanted to gather a summary of all extracellular peptidases that were identified during **FuncSanity**, but we only want this information for our genomes deemed to be complete. This is easily accomplished with the following command:

`dbdm -c Metagenomes SUMMARIZE -q "is_complete ~> is_extracellular"`

Using this command, all genomes determined to be complete are iteratively queried for extracellular peptidases. This data is summarily displayed to the screen by genome. As with above, gene calls can be stored to an external directory using `-w out_fna`, and summary data can be stored using `-x out`, with a TSV file generated for each genome that was queried.

In general, **we can link `evaluation` and `annotation`** using the format `evaluation query ~> annotation query`.

Here are a few more examples:

`dbdm -c Metagenomes SUMMARIZE -q "hqnr ~> annotated"`

<pre><code>SUMMARIZE:   View summary of all tables in database
 Project root directory:    Metagenomes
 Name of database:      Metagenomes.db

**********************************************************************************************
           Table Name:    tobg-cpc-51 
    Number of Records:    1749/4167      

              Database    Average                 Std Dev     

        phage_contig_1    0.000                   0.000       
        phage_contig_2    0.002                   0.072       
        phage_contig_3    0.000                   0.000       
            prophage_1    0.000                   0.000       
            prophage_2    0.000                   0.000       
            prophage_3    0.000                   0.000       
----------------------------------------------------------------------------------------------

              Database    Most Frequent           Number      Total Count 

                  cazy    GT41                    23          106         
      is_extracellular    False                   1715        1746        
                    ko    K08884                  25          1387        
           merops_pfam    PF00082                 9           132         
                prokka    atsA_12                 19          567         
-------------------------------------------------------------------------------------------

**********************************************************************************************
            Table Name: tobg-cpc-31 
    Number of Records:        2389/5988      

              Database    Average                 Std Dev     

        phage_contig_1    0.001                   0.029       
        phage_contig_2    0.006                   0.096       
        phage_contig_3    0.000                   0.000       
            prophage_1    0.000                   0.000       
            prophage_2    0.000                   0.000       
            prophage_3    0.000                   0.000       
----------------------------------------------------------------------------------------------

              Database    Most Frequent           Number      Total Count 

                  cazy    CE10                    13          126         
      is_extracellular    False                   2339        2359        
                    ko    K02456                  43          1690        
           merops_pfam    PF05569                 14          123         
                prokka    rcsC_19                 13          1593        
-------------------------------------------------------------------------------------------

**********************************************************************************************
            Table Name: tobg-cpc-96 
    Number of Records:        1170/2656      

          Database      Average                 Std Dev     

    phage_contig_1      0.000                   0.000       
    phage_contig_2      0.000                   0.000       
    phage_contig_3      0.000                   0.000       
        prophage_1      0.000                   0.000       
        prophage_2      0.000                   0.000       
        prophage_3      0.000                   0.000       
----------------------------------------------------------------------------------------------

            Database    Most Frequent           Number      Total Count 

                cazy    GT4                     5           51          
    is_extracellular    False                   1152        1166        
                  ko    K08884                  12          993         
         merops_pfam    PF00082                 7           77          
              prokka    pknD_10                 8           526         
-------------------------------------------------------------------------------------------

*******************************************************************************************
         Table Name:    tobg-cpc-3  
    Number of Records:        1455/2978      

            Database    Average                 Std Dev     

-------------------------------------------------------------------------------------------

            Database    Most Frequent           Number      Total Count 

                cazy    GT4                     11          78          
    is_extracellular    False                   1429        1455        
                  ko    K02004                  15          1082        
         merops_pfam    PF04389                 12          108         
              prokka    atsA_18                 15          914         
-------------------------------------------------------------------------------------------

**********************************************************************************************
        Table Name:      tobg-cpc-9  
 Number of Records:      1914/4282      

           Database      Average                 Std Dev     

     phage_contig_1      0.002                   0.091       
     phage_contig_2      0.000                   0.000       
     phage_contig_3      0.000                   0.000       
         prophage_1      0.000                   0.000       
         prophage_2      0.000                   0.000       
         prophage_3      0.000                   0.000       
----------------------------------------------------------------------------------------------

            Database     Most Frequent           Number      Total Count 

                cazy     GT4                     20          100         
    is_extracellular     False                   1858        1904        
                  ko     K08884                  18          1433        
         merops_pfam     PF00326                 12          139         
              prokka     atsA_22                 14          1082        
-------------------------------------------------------------------------------------------

**********************************************************************************************
           Table Name:      tobg-cpc-85 
    Number of Records:      1365/2980      

              Database      Average                 Std Dev     

        phage_contig_1      0.000                   0.000       
        phage_contig_2      0.000                   0.000       
        phage_contig_3      0.000                   0.000       
            prophage_1      0.000                   0.000       
            prophage_2      0.000                   0.000       
            prophage_3      0.000                   0.000       
----------------------------------------------------------------------------------------------

              Database      Most Frequent           Number      Total Count 

                  cazy      GT2                     7           69          
      is_extracellular      False                   1342        1364        
                    ko      K08884                  10          1045        
           merops_pfam      PF00326                 6           94          
                prokka     xpsE_2                  10          750         
-------------------------------------------------------------------------------------------

**********************************************************************************************
           Table Name:      tobg-np-99  
    Number of Records:      2509/6573      

              Database      Average                 Std Dev     

        phage_contig_1      0.000                   0.000       
        phage_contig_2      0.000                   0.000       
        phage_contig_3      0.000                   0.000       
            prophage_1      0.000                   0.000       
            prophage_2      0.000                   0.000       
            prophage_3      0.000                   0.000       
----------------------------------------------------------------------------------------------

              Database      Most Frequent           Number      Total Count 

                  cazy      CE10                    16          137         
      is_extracellular      False                   2478        2504        
                    ko      K02456                  49          1953        
           merops_pfam      PF05569                 15          133         
                prokka      zraR_6                  9           1251        
-------------------------------------------------------------------------------------------</code></pre>

`dbdm -c Metagenomes SUMMARIZE -q "NOT is_contaminated ~> prokka_annot"`

`dbdm -c Metagenomes SUMMARIZE -q "is_contaminated == False ~> prokka_annot"`

`dbdm -c Metagenomes SUMMARIZE -q "is_non_redundant ~> cazy_annot"`


Link putative metabolic function with annotation.
------

For researchers interested in metabolic function, gathering gene calls for genomes predicted to fulfill a particular pathway is useful for further downstream analysis.

Consider the following query:

`dbdm -c Metagenomes SUMMARIZE -t functions -q "chemotaxis > 0"`

As we expect, this query will check the `functions` table for genomes that have at least a partial pathway for chemotaxis. As before, the commands `-w out_fna` and `-x out.tsv` work as expected.

Let's introduce another **query operator**.

In general, **we can link `functions` and `annotation`** using the format `functions query -> annotation query`.

Here are a few more examples:

`dbdm -c Metagenomes SUMMARIZE -q "woodljungdahl > 0 -> prokka_annot"`

`dbdm -c Metagenomes SUMMARIZE -q "transporter_ammonia > 0.5 -> annotated"`

`dbdm -c Metagenomes SUMMARIZE -q "sulfur_assimilation == 1 -> annotated"`

For a specific example, we can look for genomes that are predicted to perform chemotaxis. Assessing the [KEGG annotations](https://github.com/bjtully/BioData/blob/master/KEGGDecoder/KOALA_definitions.txt) that belong in the chemotaxis pathway, it includes, among others, K03409. Let's go ahead and combine these searches.

`dbdm -c Metagenomes SUMMARIZE -q "chemotaxis > 0 -> ko == 'K03409'"`

Using this command, we query each genome with a partial chemotaxis pathway for this specific KO. If we had not determined a specific KO value to search for, we could also have gained insight into these genomes with the following command:

`dbdm -c Metagenomes SUMMARIZE -q "chemotaxis > 0 -> ko_annot"`

Recall that, for genome tables (ex. tobg-cpc-85, etc.), we can add `_annot` to column names of our database table - `ko_annot`, `prokka_annot`, etc. As with above, gene calls can be stored to an external directory using `-w out_fna`, and summary data can be stored using `-x out`, with a TSV file generated for each genome that was queried.


Link evaluation and putative metabolic function.
------

In fine-tuning one's data, it may be useful to investigate evaluation and metabolic functions simultaneously. To do this, we introduce one final **query operator** in the following command:

`dbdm -c Metagenomes SUMMARIZE -q "is_complete >> riboflavin_biosynthesis > 0"`

This command looks for genomes that are determined to be complete and outputs genomes (not gene calls) with at least a partial riboflavin biosynthesis pathway. As with above, output (in this case genomes) can be stored to an external directory using `-w out_fna`, and summary data can be stored using `-x out.tsv`.

In general, **we can link `evaluation` and `functions`** using the format `evaluation query >> functions query`.


Create comprehensive query statements.
------

Awesome! Our final goal in this blog is to link all three of our database tables together - genome evaluation, metabolic functions, and genome-specific annotations.

We will need two of our **query operators** - `evaluation ~> annotation` and `functions -> annotations`.

For this example dataset, we may want to determine if any high-quality, non-redundant genomes of the kingdom Planctomycetota had at least a partially complete dissimilatory nitrate reduction pathway, and to gather annotation information for all genes from those genomes that were assigned KO values. This is easily accomplished using the following:

`dbdm -c Metagenomes SUMMARIZE -q "hqnr AND kingdom == 'Planctomycetota' ~> dissim_nitrate_reduction > 0 -> ko_annot"`

Amazing! And, as before, we can store the gene calls or tsv data for this query using `-w out_fna` and `-x out`, respectively, outputting one tsv file per genome.

In general, **we can link `evaluation`, `functions`, and `annotations`** using the format `evaluation query ~> functions query -> annotation query`.


Wrapping up
======

In the past several blogs, we have walked through the use of **MetaSanity** to evaluate and annotate metagenomic-assembled genomes (MAGs). In this blog, we walked through **BioMetaDB**, an easy-to-use SQL package that lets users query their data and store results that are useful in their research. If you have any questions, feel free to send me a message!

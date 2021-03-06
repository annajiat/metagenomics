---
source: md
title: "Diversity analysis"
teaching: 10
exercises: 2
questions:
- "What is diversity intra metagenome?"
- "What is diversity inter metagenomes?"
objectives:
- "Understand α diversity"
- "Understand β diversity"
keypoints:
- "Edit the .Rmd files not the .md files"
---
## Using R studio
`http://ec2-3-235-238-92.compute-1.amazonaws.com:8787/`  

##  Manipulating lineage and rank tables in phyloseq  
Let's install phyloseq (This instruction might not work on certain version of R) 
and the rest of the required libraries for its execution:  

~~~
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("phyloseq")

install.packages(c("ggplot2", "readr", "patchwork"))
~~~
{: .language-r}  

Now, let's load the libraries (a process needed every time we began a new work 
in R and we are going to use phyloseq):

~~~
library("phyloseq")
library("ggplot2")
library("readr")
library("patchwork")
~~~
{: .language-r}

Now we have to load our metagenomes files on R 

~~~
OTUS <- read_delim("JC1A.kraken_ranked-wc","\t", escape_double = FALSE, trim_ws = TRUE)
TAXAS <- read_delim("JC1A.lineage_table-wc", "\t", escape_double = FALSE, 
                    col_types = cols(subspecies = col_character(),  
                    subspecies_2 = col_character()), trim_ws = TRUE)

~~~
{: .language-r}

Phyloseq objects are a collection of information regarding a single or a group pf metagenomes, 
these objects can be manually constructed using the basic data structures available in R or can
be created by importing the output of other programs, such as QUIIME.

Since we imported our data to basic R data types, we will make our phyloseq object manually.

~~~
names1 = OTUS$OTU
names2 = TAXAS$OTU
~~~
{: .language-r}

~~~
OTUS$OTU = NULL
TAXAS$OTU = NULL
~~~
{: .language-r}

~~~
abundances = as.matrix(OTUS)
lineages = as.matrix(TAXAS)
~~~
{: .language-r}

~~~
row.names(abundances) = names1
row.names(lineages) = names2
~~~
{: .language-r}

All that we've done so far is transforming the taxonomic lineage table 
and the OTU abundance table into a format that can be read by phyloseq.
Now we will make the phyloseq data types out of our tables.

~~~
OTU = otu_table(abundances, taxa_are_rows = TRUE)
TAX = tax_table(lineages)
~~~
{: .language-r}

We will now construct a phyloseq object using phyloseq data types: 

~~~
metagenome_JC1A = phyloseq(OTU, TAX)
~~~
{: .language-r}

> ## `.callout`
>
>If you look at our phyloseq object, you will see that there's more data types 
>that we can use to build our object. These are optional, so we will use our basic
>phyloseq object for now.  
{: .callout}


Then, we will prune all of the non-bacterial organisms in our metagenome. To do this 
we will make a subset of all bacterial groups and save them
~~~
Bacteria <- subset_taxa(metagenome_JC1A, superkingdom == "Bacteria")
~~~
{: .language-r}

We will then filter out the taxonomic groups that have less than 10 reads assigned to them.

~~~
metagenome_JC1A <- prune_taxa(taxa_sums(metagenome_JC1A)>10,metagenome_JC1A)
~~~
{: .language-r}


Now let's look at some statistics of our metagenomes, like the mean number 
of reads assigned to a taxonomic group

~~~
max(sample_sums(metagenome_JC1A))
min(sample_sums(metagenome_JC1A))
mean(sample_sums(metagenome_JC1A))
~~~
{: .language-r}

The max, min and mean can give us an idea of the eveness, but to have a more 
visual representation of the α diversity we can now look at a ggplot2
graph created using phyloseq 

~~~
p = plot_richness(metagenome_JC1A, measures = c("Observed", "Chao1", "Shannon")) 
p + geom_point(size=5, alpha=0.7)  
~~~
{: .language-r}


> ## `.discussion`
>
> How much did the α diversity change due to the filterings that we made?
{: .discussion}


> ## Exercise
> 
> How can you import the `JP4D metagenome` to phyloseq? 
> 
>> ## Solution
>> 
>> Repeat the previous instructions replacing JC1A for JP4D whenever it's appropiate
>> 
> {: .solution}
{: .challenge}  



Now that you have both phyloseq objects, one for each metagenome, you can merge them into one object:

~~~
merged_metagenomes = merge_phyloseq(metagenome_JC1A, metagenome_JP4D)
~~~
{: .language-r}


Let´s look at the phylum abundance of our metagenomes. 
Since our metagenomes have different sizes it might be a good idea to 
convert the number of assigned read into percentages (i.e. relative abundances). 

~~~
percentages  = transform_sample_counts(merged_metagenomes, function(x) x*100 / sum(x) )
~~~
{: .language-r}

Now, we can make a comparative graph between absolute reads and percentages.

~~~
absolute_count = plot_bar(merged_metagenomes, fill="phylum")
absolute_count = absolute_count + geom_bar(aes(color=phylum, fill=phylum), stat="identity", position="stack") + ggtitle("Absolute abundance")

percentages = plot_bar(percentages, fill="phylum")
percentages = percentages + geom_bar(aes(color=phylum, fill=phylum), stat="identity", position="stack") + ggtitle("Relative abundance")

absolute_count | percentages
~~~
{: .language-r}

> ## Exercise
> 
> Ejercicio `ERR2143795/JP4D_R1.fastq ` file? How confident
> 
>> ## Solution
>> ~~~
>> $ tail 
>> ~~~
>> {: .bash}
>> 
>> ~~~
>> texto
>> ~~~
>> {: .output}
>> 
>> soluion
>> 
> {: .solution}
{: .challenge}                             
                             


> ## `.callout`
>
> An aside or other comment.
{: .callout}

> ## `.discussion`
>
> Discussion questions.
{: .discussion}

                             
{% include links.md %}


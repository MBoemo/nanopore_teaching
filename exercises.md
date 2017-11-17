# Tutorial on genome assembly of repetitive genomes using long and short sequencing reads

**Software requirements**
- [ReadSim](http://sourceforge.net/p/readsim/wiki/manual/) (read simulator)
- [Velvet](https://www.ebi.ac.uk/~zerbino/velvet/) (short read *de-novo* assembler)
- [Canu](https://github.com/marbl/canu/releases) (long read *de-novo* assembler)
- [QUAST](http://bioinf.spbau.ru/quast) (assessment of assemblies)
- [samtools](http://www.htslib.org/download/) (for manipulating alignments)

These have been installed for you in the /usr directory and appropriate aliases have been added to /etc/profile.d.  You won't need to type the full path to the software.


### Introduction
The aim of this exercise is to demonstrate the advantages of long reads in the assembly of repetitive genome sequences. We will try to assemble a small part of Y chromosome known to be highly repetitive (many satellites). For a tutorial on de-novo assembly and read alignment see [here](https://github.com/demharters/assemblyTutorial).


### Generate test reads
For the purposes of this demonstration we will use simulated data. Download the reference sequence from [here](https://github.com/MBoemo/nanopore_teaching/blob/master/ref.fasta).  Right click on "Raw" and click "Copy link address".  Then use the command wget to download the reference file.

To generate simulated reads, we'll use readsim.py.  Enter the following to see the information about how to run readsim.py.  Understand what each option does.

```
readsim.py -h
```

Note that for most software, you can enter the command followed by -h or --help to read information about how to run it.

Generate short reads with the following command (scroll horizontally to see the full command):

```
readsim.py sim fa --ref ref.fasta --pre shortReadsCov30 --rev_strd on --tech nanopore --read_mu 30 --read_dist normal --cov_mu 30 --err_sub_mu 0.001 --err_in_mu 0.001 --err_del_mu 0.001
```
*Note the use of double dashes "--" which indicate options.*

This simulation will generate a set of short fasta reads (30 bases on average) with a 30x coverage using our reference sequence as a template. We set the substitution, insertion and deletion error rates to 0.1% to replicate the typical characteristics of short reads (even though the technology is set to "nanopore").

Generate long reads with the following command:

```
readsim.py sim fa --ref ref.fasta --pre longReadsCov30 --rev_strd on --tech nanopore --read_mu 15000 --read_dist normal --cov_mu 30 --err_sub_mu 0.03 --err_in_mu 0.03 --err_del_mu 0.03
```

This simulation will generate a set of long fasta reads (15kb on average) also with a 30x coverage and using our reference sequence as a template. The substitution, insertion and deletion error rates are set to 3% (i.e. we are simulating corrected reads). The error rates selected here are high and are constantly improving as Nanopore technology matures. For more information on correction see [Loman 2015, Nature Methods](http://www.nature.com/nmeth/journal/v12/n8/full/nmeth.3444.html).


### *De-novo* assembly
Perform *de-novo* assembly with short reads using Velvet.  Velveth takes a number of sequence files as input, generates a hashtable and spits out two files sequences and roadmaps, which are required by velvetg.

``` 
velveth shortReadsCov30_assembly 21 shortReadsCov30.fasta
velvetg shortReadsCov30_assembly
```
This will create a folder called “shortReadsCov30_assembly” that contains your short-read assembly.

In the velveth command '21' is the kmer length used for the hash table. If you would like to know more read section 5.2 in the [manual](http://www.ebi.ac.uk/~zerbino/velvet/Manual.pdf). If you are interested try playing around with this value and see how it affects the assembly.

For more information on how Velvet works, see [here](http://microbialinformaticsj.biomedcentral.com/articles/10.1186/2042-5783-3-2).

Perform *de-novo* assembly with long reads using Canu.

```
canu -p longReadsCov30 -d longReadsCov30_assembly java=/usr/jre-9.0.1/bin/java genomesize=50000 -nanopore-raw longReadsCov30.fasta
```
*Note: Canu requires a file "specfile.dat" in the working directory. This file is used to pass more options to canu. For our purposes this file can be empty.*

This will create a folder called “longReadsCov30_assembly” that contains your long-read assembly.

Canu is based on the Celera assembler. For terminology see [here](http://wgs-assembler.sourceforge.net/wiki/index.php/Celera_Assembler_Terminology).


### Assessment of assemblies
Assess both of your assemblies with QUAST:

```
quast.py assemblyFolder/contigFile -R ref.fasta
```
The contig file will be named something like "longReadsCov30.contigs.fasta" or "contigs.fa".

Open the quast results and look through them. You should see a large difference in '% genome fraction', which is the fraction of your genome that has been covered by reads.


### Questions
- Q1. What is the main difference between the two assemblies?
- Q2. Let's assess our reference sequence with RepeatMasker. This tool will allow us to identify any repetitive elements.
Go to the [RepeatMasker webserver](http://www.repeatmasker.org/cgi-bin/WEBRepeatMasker) and upload the reference sequence.
*Note, the sequence has to be shorter than 100kb.* Press 'submit sequence'.
*You may have to refresh the page after a while.*
As you can see, our sequence is full of satellites. How will this have affected the short read assembly?  The long read assembly?
- Q3. How high can you go with the error rates in the simulated long reads before the assembly starts to fail? Assume 15kb read length and 30x coverage.
- Q4. How does alignment of short reads to a reference sequence compare to short read de-novo assembly in terms of '% genome covered'?


### Further reading
- A tutorial on samtools is [here](http://biobits.org/samtools_primer.html).
- Beginner’s guide to comparative bacterial genome analysis using next-generation sequence data
[DOI: 10.1186/2042-5783-3-2](http://microbialinformaticsj.biomedcentral.com/articles/10.1186/2042-5783-3-2)
- Treangen et al. "[Repetitive DNA and next-generation sequencing: computational challenges and solutions.](http://www.nature.com/nrg/journal/v13/n1/full/nrg3117.html)" Nature Reviews Genetics 13 (2012): 36-46.
- Loman et al. "[A complete bacterial genome assembled de novo using only nanopore sequencing data.](https://www.nature.com/articles/nmeth.3444)" Nature Methods 12 (2015): 733–735.
- Simpson et al. "[Detecting DNA cytosine methylation using nanopore sequencing.](https://www.nature.com/articles/nmeth.4184)" Nature Methods 14 (2017): 407–410.
- [Celera (Canu) Assembler](http://wgs-assembler.sourceforge.net/wiki/index.php/Celera_Assembler_Terminology)
 [Canu Terminology](http://wgs-assembler.sourceforge.net/wiki/index.php/Celera_Assembler_Terminology) (some of it is irrelevant to nanopore reads e.g. mate-pairs)
[Slides on genome analysis](http://schatzlab.cshl.edu/teaching/) by the Schatz lab

# Tutorial on genome assembly of repetitive genomes using long and short sequencing reads

**Software requirements**
- [ReadSim](http://sourceforge.net/p/readsim/wiki/manual/) (read simulator)
- [Velvet](https://www.ebi.ac.uk/~zerbino/velvet/) (short read *de-novo* assembler)
- [Canu](https://github.com/marbl/canu/releases) (long read *de-novo* assembler)
- [QUAST](http://bioinf.spbau.ru/quast) (assessment of assemblies)
- [Bowtie 2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml) (short read aligner)
<!--- - [LAST](http://last.cbrc.jp/) (long read aligner) --->
- [samtools](http://www.htslib.org/download/) (for manipulating alignments)
- [Tablet](https://ics.hutton.ac.uk/tablet/) (assembly viewer)

### Introduction
The aim of this exercise is to demonstrate the advantages of long reads in the assembly of repetitive genome sequences. We will try to assemble a small part of Y chromosome known to be highly repetitive (many satellites). For a tutorial on de-novo assembly and read alignment see [here](https://github.com/demharters/assemblyTutorial).

### Generate test reads
For the purposes of this demonstration we will use simulated data. Download the reference sequence from [here](https://figshare.com/s/97580ff5981bed6e921a).
*If you would like to skip this step you can find simulated datasets for long and short reads under the same link.*

Generate short reads with the following command:

```
readsim.py sim fa --ref ref.fasta --pre shortReadsCov30 --rev_strd on\
 --tech nanopore --read_mu 30 --read_dist normal --cov_mu 30\
 --err_sub_mu 0.001 --err_in_mu 0.001 --err_del_mu 0.001
```
*Note the use of double dashes "--". Backslashes indicate line breaks.*

This simulation will generate a set of short fasta reads (30 bases on average) with a 30x coverage using our reference sequence as a template. We set the substitution, insertion and deletion error rates to 0.1% to replicate the typical characteristics of short reads (even though the technology is set to "nanopore").

Generate long reads with the following command:

```
readsim.py sim fa --ref ref.fasta --pre longReadsCov30\
 --rev_strd on --tech nanopore --read_mu 15000 --read_dist normal --cov_mu 30\
 --err_sub_mu 0.03 --err_in_mu 0.03 --err_del_mu 0.03
```

This simulation will generate a set of long fasta reads (15kb on average) also with a 30x coverage and using our reference sequence as a template. The substitution, insertion and deletion error rates are set to 3% (i.e. we are simulating corrected reads). The error rates selected here are high and are constantly improving as Nanopore technology matures. For more information on correction see [Loman 2015, Nature Methods](http://www.nature.com/nmeth/journal/v12/n8/full/nmeth.3444.html).


### *De-novo* assembly

Perform *de-novo* assembly with short reads using Velvet.

``` 
velveth shortReadsCov30_assembly 21 shortReadsCov30.fasta
velvetg shorReadsCov30_assembly
```
*This will create a folder called “shortReadsCov30_assembly” that contains your short-read assembly.*

*Velveth takes a number of sequence files as input, generates a hashtable and spits out two files sequences and roadmaps, which are required by velvetg.*

In the velveth command '21' is the kmer length used for the hash table. If you would like to know more read section 5.2 in the [manual](http://www.ebi.ac.uk/~zerbino/velvet/Manual.pdf). If you are interested try playing around with this value and see how it affects the assembly.

For more information on how velvet works, see [here](http://microbialinformaticsj.biomedcentral.com/articles/10.1186/2042-5783-3-2).

Perform *de-novo* assembly with long reads using Canu.

```
canu -p longReadsCov30 -d longReadsCov30_assembly genomesize=50000\
 -nanopore-raw longReadsCov30.fasta
```
*Canu requires a file "specfile.dat" in the working directory. This file is used to pass more options to canu. For our purposes this file can be empty.*

This will create a folder called “longReadsCov30_assembly” that contains your long-read assembly.

Canu is based on the Celera assembler. For terminology see [here](http://wgs-assembler.sourceforge.net/wiki/index.php/Celera_Assembler_Terminology).

### Assessment of assemblies
Assess both of your assemblies with QUAST:

```
quast.py assemblyFolder/contigFile -R ref.fasta
```
*The contig file will be named something like "longReadsCov30.contigs.fasta" or "contigs.fa".*

Open quast_results/results/report.pdf to see the results. What do you see?

*Spoiler: You should see a large difference in '% genome fraction', which is the fraction of your genome that has been covered by reads.*


### Questions
- Q1. What is the main difference between the two assemblies?
- Q2. How do the RepeatMasker results explain the difference in quality between the two assemblies?
Let's assess our reference sequence with RepeatMasker. This tool will allow us to identify any repetitive elements.
Go to the [RepeatMasker webserver](http://www.repeatmasker.org/cgi-bin/WEBRepeatMasker) and upload the reference sequence.
*Note, the sequence has to be shorter than 100kb.* Press 'submit sequence'.
*You may have to refresh the page after a while.*
As you can see our sequence is full of satellites. This is bad news for short reads and it is most likely the reason for our poor *de-novo* assembly. Our long read assembly did a lot better it seems. As expected, this suggests that assembly of repetitive genome regions requires long reads that span entire repeats.
- Q3. How high can you go with the error rates in the simulated long reads before the assembly starts to fail? Assume 15kb read length and 30x coverage.
- Q4. How does alignment of short reads to a reference sequence compare to short read de-novo assembly in terms of '% genome covered'?

To assemble the short reads by alignment we need a different alignment tool. Here, we will use Bowtie 2.

```
bowtie2-build ref.fasta ref.fasta
bowtie2 -f -x ref.fasta -U shortReadsCov30.fasta -S shortReadsCov30_aligned.sam
samtools view -bS shortReadsCov30_aligned.sam > shortReadsCov30_aligned.bam
samtools sort shortReadsCov30_aligned.bam shortReadsCov30_aligned.sorted.bam
samtools index shortReadsCov30_aligned.sorted.bam
```

To visualise the alignment open the [Tablet](https://ics.hutton.ac.uk/tablet/) assembly viewer, and load your sorted.bam file as the assembly and your ref.fasta file as reference. A new entry should appear in the left column. Select it and have a look at the assembly. If you don’t see the coverage, go to ‘Advanced’ and select ‘coverage’.

How do you explain the difference between the *de-novo* and the alignment assembly?

### Further reading
- Tutorial on [samtools](http://biobits.org/samtools_primer.html).
- Beginner’s guide to comparative bacterial genome analysis using next-generation sequence data
[DOI: 10.1186/2042-5783-3-2](http://microbialinformaticsj.biomedcentral.com/articles/10.1186/2042-5783-3-2)
- Treangen et al. "[Repetitive DNA and next-generation sequencing: computational challenges and solutions.](http://www.nature.com/nrg/journal/v13/n1/full/nrg3117.html)" Nature Reviews Genetics 13.1 (2012): 36-46.
- [Celera (Canu) Assembler](http://wgs-assembler.sourceforge.net/wiki/index.php/Celera_Assembler_Terminology)
 [Canu Terminology](http://wgs-assembler.sourceforge.net/wiki/index.php/Celera_Assembler_Terminology) (some of it is irrelevant to nanopore reads e.g. mate-pairs)
[Slides on genome analysis](http://schatzlab.cshl.edu/teaching/) by the Schatz lab

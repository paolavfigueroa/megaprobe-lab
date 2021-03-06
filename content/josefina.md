Title: Josefina Correa Menendez
Date: 2016-08-26 
Category: People 
Tags: de-novo

#Biography

My name is Josefina Correa. I am an undergraduate Biology major at the University of Puerto Rico, Rio Piedras campus; 
I am also taking computer science courses with the goal of completing a dual concentration. I am a member of the IDI-BD2K program
(Increasing Diversity in Big Data to Knowledge), an initiative that seeks to train students in the adequate use of large quantities of data for producing coherent quantiative analysis within the context of Biomedical Informatics. This semester I will be working with Dr. Ortiz and Dr. García-Arrarás on a project pertaining to *de-novo* sequencing.

#Contact

* e-mail: josefina.correa@upr.edu
* Github: https://github.com/josefinacmenendez

#Project Description

This project aims to assemble and annotate transcripts from H. glaberrima.

#Weekly Reports
##Second semester
##Week 3: 1/30/17 - 2/5/17
I spent some time this week working on the manuscript.
I also worked on mapping the contigs from two different assemblies from the same transcriptome data to aid in comparing differential expression based on the Datatube GUI.

The two assemblies are:     
(1) combined_transcripts_cleaned_fasta_cap_contigs.fasta (assembled using The Eel-pond mRNA-seq Protocol)     
(2) Hglaberrima_RNC_transcriptome_Contigs.fasta.zip (assembled by Vladimir Mashanov, DOI: 10.1186/1471-2164-15-357)     

Mapping contig ID's from two different assemblies from the same data:

```
import argparse
from Bio import SeqIO

parser = argparse.ArgumentParser(description = "reads two .fasta files")
parser.add_argument("--fasta_1", help = ".fasta file 1", required = True)
parser.add_argument("--fasta_2", help = ".fasta file 2", required = True)

args = parser.parse_args()

fasta_1 = args.fasta_1
fasta_2 = args.fasta_2

def get_contig_dict(fasta_file):
        contig_dict = {}
        with open(fasta_file, 'rU') as handle:
                for record in SeqIO.parse(handle, 'fasta'):
                        contig_dict[str(record.seq)] = record.id
        return contig_dict

def map_ids(dict1, dict2):
        mapped_ids = []
        differ_ids = 0
        for key in dict1.keys():
                if key in dict2.keys():
                        mapped_ids.append( [dict1[key], dict2[key] ] )
                        print mapped_ids
                        if dict1[key] != dict2[key]:
                                differ_ids += 1
                else:
                        mapped_ids.append( [dict1[key] , "NA" ] )

        if differ_ids  == 0:
                mapped_ids.insert( 0 , ['Different_Ids' , differ_ids] )
        return mapped_ids

contig_dict_1 = get_contig_dict(fasta_1)
contig_dict_2 = get_contig_dict(fasta_2)
mapped_contig_ids = map_ids(contig_dict_1, contig_dict_2)

for item in mapped_contig_ids:
        print item

```



I downloaded the assembled contigs from dx.doi.org/10.6070/H4PN93J1

##Week 2: 1/23/17 - 1/29/17
I have been working on revising the initial submission of a manuscript that I helped write during last summer's internship at U. Pitt's DBMI.

The original submission can be found here; I am listed in the acknowledgements section:

```
https://f1000research.com/articles/5-1919/v1
```

##Week 1: 1/16/17 - 1/22/17
I re-ran The Eel-Pond mRNAseq Protocol on Hulk after the server was re-established. I made some adjustments to some scripts to annotate the contigs using the S. purpuratus ref-seq.
The following commands were used:

Make a virtual environment and installing the necessary tools to make transcript families and annotate the assembly
```
module load anaconda
conda create -n eel-pond python=2.7
source activate eel-pond
```
Install screed
```
cd /home/josefina/.conda/envs/eel-pond/share
git clone https://github.com/ged-lab/screed.git
cd screed
git checkout protocols-v0.8.3
python setup.py install
```
Install khmer
```
cd /home/josefina/.conda/envs/eel-pond/share
git clone https://github.com/ged-lab/khmer.git
cd khmer
git checkout protocols-v0.8.3
make
echo 'export PYTHONPATH=$PYTHONPATH:/home/josefina/.conda/envs/eel-pond/share/khmer' >> ~/.bashrc
source ~/.bashrc
```
Install Blast Legacy v 2.2.24
```
cd /home/josefina/.conda/envs/eel-pond/bin
curl -O ftp://ftp.ncbi.nlm.nih.gov/blast/executables/legacy/2.2.24/blast-2.2.24-x64-linux.tar.gz
tar xzf blast-2.2.24-x64-linux.tar.gz
cd bin
mv blast-2.2.24 ../BLAST/
mv blast-2.2.24-x64-linux.tar.gz ../BLAST/
cd ../BLAST/
cp blast-2.2.24/bin/* /home/josefina/.conda/envs/eel-pond/bin
cp -r blast-2.2.24/data /home/josefina/.conda/envs/eel-pond/blast-data
```

Modify the path of some scripts:
```
cd share/eel-pond
nano make-uni-best-hits.py
nano make-reciprocal-best-hits.py
echo 'added sys.path.inser(0, '/home/josefina/.conda/envs/eel-pond/blastkit/lib') to make-reciprocal-best-hits.py and make-uni-best-hits.py'
```



Build transcript families
```
echo 'Starting from step 5: Building Transcript Families using Blast2'
python /home/josefina/.conda/envs/eel-pond/share/khmer/scripts/do-partition.py -x 1e9 -N 4 --threads 4 pepino combined_transcripts_cleaned_fasta_cap_contigs.fasta
python /home/josefina/.conda/envs/eel-pond/share/eel-pond/rename-with-partitions.py pepino combined_transcripts_cleaned_fasta_cap_contigs.fasta.part
```

Blast the contigs
```
mv combined_transcripts_cleaned_fasta_cap_contigs.fasta.part.renamed.fasta.gz pepino_combined_transcripts_renamed.fasta.gz
gunzip pepino_combined_transcripts_renamed.fasta.gz
formatdb -i pepino_combined_transcripts_renamed.fasta -o T -p F
for file in mouse.1.protein.faa.gz mouse.2.protein.faa.gz mouse.3.protein.faa.gz; do curl -O ftp://ftp.ncbi.nih.gov/refseq/M_musculus/mRNA_Prot/${file}; done
gunzip mouse.[123].protein.faa.gz
cat mouse.[123].protein.faa > mouse.protein.faa
formatdb -i mouse.protein.faa -o T -p T
```

```
screen -S BLASTALL_1
Ctrl A D #to unattach screen
blastall -i mouse.protein.faa -d pepino_combined_transcripts_renamed.fasta -e 1e-3 -p tblastn -o mouse.x.pepino -a 8 -v 4 -b 4
```
```
screen -S BLASTALL_2
blastall -i pepino_combined_transcripts_renamed.fasta -d mouse.protein.faa -e 1e-3 -p blastx -o pepino.x.mouse -a 8 -v 4 -b 4
Ctrl A D #to unattach screen
```
```
python /home/josefina/.conda/envs/eel-pond/share/eel-pond/make-namedb.py mouse.protein.faa mouse.namedb
python -m screed.fadbm mouse.protein.faa
python /home/josefina/.conda/envs/eel-pond/share/eel-pond/make-uni-best-hits.py pepino.x.mouse pepino.x.mouse.homol
```
```
python /home/josefina/.conda/envs/eel-pond/share/eel-pond/make-reciprocal-best-hits.py pepino.x.mouse mouse.x.pepino pepino.x.mouse.ortho
```

```
python /home/josefina/.conda/envs/eel-pond/share/eel-pond/annotate-seqs.py pepino_combined_transcripts_renamed.fasta pepino.x.mouse.ortho pepino.x.mouse.homol
```

Blast the contigs against S. purpuratus ref-seq
```
cd ../pepino.on.purpuratus/
wget ftp://ftp.ncbi.nlm.nih.gov/genomes/refseq/invertebrate/Strongylocentrotus_purpuratus/representative/GCF_000002235.4_Spur_4.2/GCF_000002235.4_Spur_4.2_protein.faa.gz
gunzip GCF_000002235.4_Spur_4.2_protein.faa.gz
formatdb -i GCF_000002235.4_Spur_4.2_protein.faa -o T -p T
formatdb -i combined_transcripts_cleaned_fasta_cap_contigs.renamed.fasta -o T -p F

screen -S BLASTX
blastall -i combined_transcripts_cleaned_fasta_cap_contigs.renamed.fasta -d GCF_000002235.4_Spur_4.2_protein.faa -e 1e-3 -p blastx -o pepino.x.purpuratus -a 8 -v 4 -b 4
Ctrl A D

screen -S TBLASTN
blastall -i GCF_000002235.4_Spur_4.2_protein.faa -d pepino_db -e 1e-3 -p tblastn -o purpuratus.x.pepino -a 8 -v 4 -b 4
Ctrl A D

python /home/josefina/.conda/envs/eel-pond/share/eel-pond/make-uni-best-hits.py pepino.x.purpuratus pepino.x.purpuratus.homol

python /home/josefina/.conda/envs/eel-pond/share/eel-pond/make-reciprocal-best-hits.py pepino.x.purpuratus purpuratus.x.pepino pepino.x.purpuratus.ortho

python /home/josefina/.conda/envs/eel-pond/share/eel-pond/make-namedb.py GCF_000002235.4_Spur_4.2_protein.faa purpuratus.namedb

python -m screed.fadbm GCF_000002235.4_Spur_4.2_protein.faa
```

Obtain homologies and orthologies using the S. purpuratus ref-seq

```
python /home/josefina/.conda/envs/eel-pond/share/eel-pond/make-uni-best-hits.py pepino.x.purpuratus pepino.x.purpuratus.homol

python /home/josefina/.conda/envs/eel-pond/share/eel-pond/make-reciprocal-best-hits.py pepino.x.purpuratus purpuratus.x.pepino pepino.x.purpuratus.ortho
  
python /home/josefina/.conda/envs/eel-pond/share/eel-pond/make-namedb.py 
```

Edit the namedb.py script to allow for parsing purpuratus.namedb
```
nano /home/josefina/.conda/envs/eel-pond/share/eel-pond/namedb.py
```

Format GCF_000002235.4_Spur_4.2_protein.faa as purpuratus.namedb
```
GCF_000002235.4_Spur_4.2_protein.faa purpuratus.namedb
```
```
python -m screed.fadbm GCF_000002235.4_Spur_4.2_protein.faa
```
Annotate the sequences using the S. purpuratus ref-seq
```
python /home/josefina/.conda/envs/eel-pond/share/eel-pond/annotate-seqs.py pepino_db pepino.x.purpuratus.ortho pepino.x.purpuratus.homol
```


##First semester
##Week 16: 12/04/16 - 12/05/16

annotate-seqs.py was fixed; the changes are documented on the forks humberto-ortiz/eel-pond and josefinacmenendez/eel-pond

However, when attempted with the pepino data, no hits are found for the following reason: no orthologies or homologies are found. I found that on the previous step (building transcript families) rename-with-partitions.py was building a name with a space between > NAME instead of building it >NAME; I fixed this, but this did not correct the problem. It's as if blastparser is not parsing the data into make-uni-best-hits.py or make-reciprocal-best-hits.py.

##Week 15: 11/28/16 - 12/4/16

I attempted the eel-pond procedure locally, but obtained no hits when running annotate-seqs.py

I also familiarized myself with using git hub via the command line.

The following steps describe how to get started with git hub using the command line:

1. Configure the repository
2. Create a copy of the repository on a local directory
3. Add a file (also works for editing a file)
4. Commit the file
5. Push the file
6. Check the status

```
git config --global user.email "EMAIL"
git config --global user.name "USERNAME"
git clone http://github.com/USERNAME/REPOSITORY.git
cd REPOSITORY
git add FILENAME
git commit -m "MESSAGE"
git push
```

To remove the file, input:

```
git pull FILENAME
```

To add all files in a directory:

```
git add . 
```
OR
```
git add -A
```

##Week 14: 11/21/16 - 11/27/16

I attempted the procedure using data from S. purpuratus, taken from ftp://ftp.ncbi.nih.gov/genomes/Strongylocentrotus_purpuratus/protein/ and obtained no hits. 

I also attempted to run the tutorial using the canned blasts and the Nemastotella transcriptome, and yet obtained no hits.
I downloaded the BLAST version 2.2.26 and used formatdb instead of makeblastdb to format the mouse and the nemastotella data.
This BLAST version was downloaded from: https://ftp.ncbi.nlm.nih.gov/blast/executables/legacy/2.2.26/ and installed according to directions detailed on https://github.com/ctb/blastkit. Rather than using virtualenv, a conda environment (named blastkit) was used. However, no hits were obtained when annotating the transcripts. 

##Week 13: 11/14/16 - 11/20/16

Annotating transcript families

The following protocol is being used: https://khmer-protocols.readthedocs.io/en/latest/mrnaseq/6-annotating-transcript-families.html


Downloading from NCBI RefSeq Mouse database to /data/josefina/DATATUBE/mRNAseq/data/refSeqMouse/:
```
(blastkit)[josefina@hulk annotations_2]$ for file in mouse.1.protein.faa.gz mouse.2.protein.faa.gz mouse.3.protein.faa.gz
(blastkit)[josefina@hulk annotations_2]$ 	do 
(blastkit)[josefina@hulk annotations_2]$ 	wget "ftp://ftp.ncbi.nlm.nih.gov/refseq/M_musculus/mRNA_Prot/${file}"
(blastkit)[josefina@hulk annotations_2]$ 	done
```
decompressing files
```
(blastkit)[josefina@hulk annotations_2]$ gunzip mouse.[123].protein.faa.gz
(blastkit)[josefina@hulk annotations_2]$ cat mouse.[123].protein.faa > mouse.protein.faa
```
formating files using makeblastdb instead of formatdb:
files are in /data/josefina/DATATUBE/mRNAseq/annotations/
```
(blastkit)[josefina@hulk annotations_2]$ makeblastdb -in mouse.protein.faa -dbtype prot

Building a new DB, current time: 11/15/2016 12:29:19
New DB name:   mouse.protein.faa
New DB title:  mouse.protein.faa
Sequence type: Protein
Keep Linkouts: T
Keep MBits: T
Maximum file size: 1000000000B
Adding sequences from FASTA; added 76419 sequences in 6.85848 seconds.

(blastkit)[josefina@hulk annotations_2]$ makeblastdb -in combined_transcripts_cleaned_renamed_pepino.fa -dbtype nucl

Building a new DB, current time: 11/15/2016 13:51:19
New DB name:   combined_transcripts_cleaned_renamed_pepino.fa
New DB title:  combined_transcripts_cleaned_renamed_pepino.fa
Sequence type: Nucleotide
Keep Linkouts: T
Keep MBits: T
Maximum file size: 1000000000B
Adding sequences from FASTA; added 63218 sequences in 8.16066 seconds.
```
running blastx
```
(blastkit)[josefina@hulk annotations_2]$ blastx -db mouse.protein.faa -query combined_transcripts_cleaned_renamed_pepino.fa \ -evalue 1e-3 -num_threads 8 -num_descriptions 4 -num_alignments 4 -out pepino.x.mouse

```
running tblastn
```
(blastkit)[josefina@hulk annotations_2]$ tblastn -db combined_transcripts_cleaned_renamed_pepino.fa -query \
mouse.protein.faa -evalue 1e-3 -num_threads 8 -num_descriptions 4 -num_alignments 4 -out mouse.x.pepino
```
Several warnings were observed wen running tblastn due to short sequences in the mouse RNA-seq data:

```
Warning: lcl|Query_2936 gi|9256636|ref|NP_061348.1| 60S ribosomal protein L41 [Mus musculus]: Warning: Could not calculate ungapped Karlin-Altschul parameters due to an invalid query sequence or its translation. Please verify the query sequence(s) and/or filtering options

(blastkit)[josefina@hulk annotations_2]$  grep -n "9256636" mouse.protein.faa
18946:>gi|9256636|ref|NP_061348.1| 60S ribosomal protein L41 [Mus musculus]

(blastkit)[josefina@hulk annotations_2]$  grep -A 1 "9256636" mouse.protein.faa
>gi|9256636|ref|NP_061348.1| 60S ribosomal protein L41 [Mus musculus]
MRAKWRKKRMRRLKRKRRKMRQRSK
```
The following commands use scripts from the eel-pond tutorial by Dr. Titus Brown.
These scripts are available at: https://github.com/ctb/eel-pond.
Screed was downloaded into the virutal environment using 'pip install screed'.

Putative homology (best BLAST hit) and orthology (reciprocal best hits) were calculated:

```
(blastkit)[josefina@hulk annotations_2]$ python ../eelpond_scripts/make-uni-best-hits.py pepino.x.mouse pepino.x.mouse.homol
(blastkit)[josefina@hulk annotations_2]$ python ../eelpond_scripts/make-reciprocal-best-hits.py pepino.x.mouse mouse.x.pepino pepino.x.mouse.ortho
```

The mouse RNA-seq data was re-formatted:
```
(blastkit)[josefina@hulk annotations_2]$ python ../eelpond_scripts/make-namedb.py mouse.protein.faa mouse.namedb
(blastkit)[josefina@hulk annotations_2]$ python -m screed.fadbm mouse.protein.faa
```
Annotation was attempted using the following commands:
```
(blastkit)[josefina@hulk annotations_2]$ python ../eelpond_scripts/annotate-seqs.py \ combined_transcripts_cleaned_renamed_pepino.fa pepino.x.mouse.ortho pepino.x.mouse.homol 
Scanning sequences -- first pass to gather info
... 0
... 25000
... 50000
second pass: annotating
... x2 0
... x2 25000
... x2 50000
----
63217 sequences total
0 annotated / ortho
0 annotated / homol
0 annotated / tr
0 total annotated

annotated sequences in FASTA format: combined_transcripts_cleaned_renamed_pepino.fa.annot
annotation spreadsheet in: combined_transcripts_cleaned_renamed_pepino.fa.annot.csv
annotation spreadsheet with sequences (warning: LARGE): combined_transcripts_cleaned_renamed_pepino.fa.annot.large.csv
```

##Week 12: 11/07/16 - 11/13/16

Building transcript families:

Ran the fifth step of The Eel Pond mRNAseq Protocol from https://khmer-protocols.readthedocs.io/en/mrnaseq/index.html locally.
Failed attempts to reproduce this protocol on Hulk were made and the following was learned:

* gcc version 5.4.0 or greater is required
* installing khmer using pip install khmer and running do-partition.py results in the following error:
```
terminate called after throwing an instance of 'std::ios_base::failure'

what(): basic_ios::clear
```
The following commands worked:
```
module load gcc/5.4.0
module load anaconda
conda create --name khmer_tools python
source activate khmer_tools
cd /home/josefina/.conda/envs/khmer_tools

git clone https://github.com/dib-lab/khmer.git
make install-dependencies
make
./setup.py install --user

cd /data/josefina/DATATUBE/partitions_on_hulk

python do-partition.py -x 8e9 -N 4 --threads ${THREADS:-1} pepino 

combined_transcripts_cleaned_fasta_cap_contigs.fa.gz 
rename-with-partitions.py pepino combined_transcripts_cleaned_fasta_cap_contigs.fa.gz.part

mv combined_transcripts_cleaned_fasta_cap_contigs.renamed.fasta.gz \
	combined_transcripts_renamed.fa.gz

```
##Week 11: 10/31/16 - 11/06/16

Building transcript families:

Ran the fifth step of The Eel Pond mRNAseq Protocol from https://khmer-protocols.readthedocs.io/en/mrnaseq/index.html
I attempted to run this on Hulk by using the following commands:
```
[josefina@hulk khmer_venv]$ module load gcc
[josefina@hulk khmer_venv]$ module load anaconda
[josefina@hulk khmer_venv]$ conda create -n khmer python
[josefina@hulk khmer_venv]$ . activate khmer
(khmer)[josefina@hulk khmer_venv]$ pip install khmer
```
However, when running the script do-partition.py, the following error came up:

```
(khmer)[josefina@hulk khmer_venv]$ do-partition.py

Traceback (most recent call last):
  File "/home/josefina/.conda/envs/khmer/bin/do-partition.py", line 46, in <module>
    import khmer
  File "/home/josefina/.conda/envs/khmer/lib/python2.7/site-packages/khmer/__init__.py", line 42, in <module>
    from khmer._khmer import Countgraph as _Countgraph
ImportError: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by /home/josefina/.conda/envs/khmer/lib/python2.7/site-packages/khmer/_khmer.so)
```
It seems there is some sort of incompatibility with the gcc version recquired to run this, and the gcc version installed on Hulk.

```
(khmer)[josefina@hulk khmer_venv]$ gcc --version
gcc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-17)
Copyright (C) 2010 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
However, it appears that the loaded gcc version is up to date:
```
(khmer)[josefina@hulk khmer_venv]$ module list
Currently Loaded Modulefiles:
  1) gcc/6.2.0   2) anaconda

```
I created a virtual environment locally and installed khmer as follows, and did not obtain the previous error when inputting 'do-partition.py':
```
josefina@josefina-Q552UB:~/virtual_environments$ sudo apt-get update
josefina@josefina-Q552UB:~/virtual_environments$ sudo apt-get -y install screen git curl gcc make g++ python-dev \ > unzip default-jre pkg-config libncurses5-dev r-base-core r-cran-gplots python-matplotlib python-pip \   
> python-virtualenv sysstat fastqc trimmomatic fastx-toolkit bowtie samtools blast2

josefina@josefina-Q552UB:~/virtual_environments$ gcc --version
gcc (Ubuntu 5.4.0-6ubuntu1~16.04.2) 5.4.0 20160609
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.


josefina@josefina-Q552UB:~/virtual_environments$ python2.7 -m virtualenv khmer_venv
josefina@josefina-Q552UB:~/virtual_environments$ source khmer_venv/bin/activate
(khmer_venv) josefina@josefina-Q552UB:~/virtual_environments$ pip install khmer
(khmer_venv) josefina@josefina-Q552UB:~/virtual_environments$ do-partition.py
|| This is the script do-partition.py in khmer.
|| You are running khmer version 2.0
|| You are also using screed version 0.9
||
|| If you use this script in a publication, please cite EACH of the following:
||
||   * MR Crusoe et al., 2015. http://dx.doi.org/10.12688/f1000research.6924.1
||   * J Pell et al., http://dx.doi.org/10.1073/pnas.1121464109
||
|| Please see http://khmer.readthedocs.org/en/latest/citations.html for details.

usage: do-partition.py [-h] [--version] [--ksize KSIZE] [--n_tables N_TABLES]
                       [-U UNIQUE_KMERS] [--fp-rate FP_RATE]
                       [--max-tablesize MAX_TABLESIZE | -M MAX_MEMORY_USAGE]
                       [--threads THREADS] [--subset-size SUBSET_SIZE]
                       [--no-big-traverse] [--keep-subsets] [-f]
                       graphbase input_sequence_filename
                       [input_sequence_filename ...]
do-partition.py: error: too few arguments

```

##Week 10: 10/24/16 - 10/28/16

Made the following updates to Datatube_Heatmap at <https://github.com/josefinacmenendez/Datatube_Heatmap>:

* The full list of contigs can be accessed on <https://josefinacmenendez.shinyapps.io/Datatube_Heatmap/> . To do so, type the contig name, or any letter that might be in that name (i.e. G for the Gypsy contigs).   
* The generated heatmap can be exported as a pdf  
* The differential expression data is now processed on global.R

##Week 9: 10/17/16- 10/21/16

Made the following updates to Datatube_Heatmap at <https://github.com/josefinacmenendez/Datatube_Heatmap>:

* corrected error messages
* improved image scaling
* the full list of contigs can be accessed through R studio, but not through shinyapps.io; this will be fixed soon
  
##Week 8: 10/10/16- 10/14/16

Set up a repository for the shiny app built for showing the heatmaps:

* <https://github.com/josefinacmenendez/Datatube_Heatmap>

The shiny app is functional. However, the following issues will be addressed this week:

*  Parsing the full set of contigs into the selectizeInput list crashes the app
*  There are several error messages that should be substituted with prompt messages

##Week 7: 10/3/16 - 10/7/16

Worked on writing an R script for plotting heatmaps using differential gene expression for H. glaberrima data:
The data is available on Hulk. I also spent some time becoming familiar with Shiny. I will be developing an app to view the heatmaps in a more user-friendly and interactive way.
```{R}
#set the working directory to wherever the data is and read the three files accordingly.
setwd("C:/Users/Josefina/Google Drive/Coursework/Megaprobe/Datatube")
library(gplots)

nvs2 <- read.csv("N_vs_day2_whole_list.csv")
nvs12<- read.csv("N_vs_day12_whole_list.csv")
nvs20<- read.csv("N_vs_day20_whole_list.csv")

contig.number <- nvs2$id

nvs2.log2Foldchange <- nvs2$log2FoldChange
nvs12.log2Foldchange<- nvs12$log2FoldChange
nvs20.log2Foldchange<- nvs20$log2FoldChange

dge_pepino <- cbind(nvs2.log2Foldchange,nvs12.log2Foldchange,nvs20.log2Foldchange)
dge_pepino <- as.matrix(toHeatmap)
colnames(dge_pepino) <- c("Day2","Day12","Day20")
rownames(dge_pepino) <- contig.number
library(functional)
#Remove "Inf", "-Inf", and "NA"
dge_pepino <- dge_pepino[apply(dge_pepino, 1, Compose(is.finite, all)), , drop=FALSE]

####GET ALL HEATMAPS####
maxSize = dim(dge_pepino)[1]

startPoint = 1
endPoint = 20
while(startPoint < maxSize)
{
  heatmap.2(dge_pepino[c(startPoint:endPoint),], trace = "none", col = redblue(20), main = startPoint);
  startPoint = startPoint + 20;
  endPoint = endPoint + 20;
  if(endPoint > maxSize)
  {
    endPoint = maxSize;
  };
}

write.csv(dge_pepino, "pepino_deg.csv", row.names = TRUE)
```

##Week 6: 9/26/16 - 9/30/16
##Week 5: 9/19/16 - 9/23/16

##Week 4: 9/12/16 - 9/16/16
I set up the virtual machine with the necessary tools to run the assembly:

* Downloaded git from <https://git-scm.com/download/win>

* Created virtual machine with Ubuntu 16.04 OS following the instructions detailed on   
<https://aws.amazon.com/getting-started/tutorials/launch-a-virtual-machine/>

* Used Git Bash to install the base image containing trimommatic and khmer tools using the following commands as detailed on <http://2016-aug-nonmodel-rnaseq.readthedocs.io/en/latest/install.html>

    - Boot a recent Ubuntu on Amazon (wily 15.10 image or later)
    - Log in as user ‘ubuntu’.
    - Run the following commands:
```
      sudo apt-get -y update && sudo apt-get -y install r-base python3-matplotlib libzmq3-dev python3.5-dev
      texlive-latex-extra texlive-latex-recommended python3-virtualenv trimmomatic fastqc python-pip python-dev bowtie samtools zlib1g-dev ncurses-dev
      
      sudo pip install -U setuptools khmer==2.0 jupyter jupyter_client ipython pandas
```

##Week 3: 9/05/16 - 9/09/16
I have been familiarizing myself with the project, reading what has been published on the matter by Mashanov *et. al.*:
<http://www.biomedcentral.com/1471-2164/15/357>

##Week 2: 8/29/16 - 9/02/16
Dr. Ortiz, Dr. García-Arrarás and I met to discuss the biological intrinciacies of the *de-novo* sequencing project.
We discussed the biological aspects of the project and detailed the following goals for this semester:

* Use Vladimir's Databank as a database to search for homologies between other species 
* Search for differential gene expression among reads
* Produce a new assembly to expand the databank and include the transcriptome of reads for the regenerated and normal intestine of *H. glaberrima*

##Week 1: 8/22/16 - 8/26/16

First lab meeting today. I am excited to learn more about my project.

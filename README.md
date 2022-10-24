# MCHelper (Beta, non-fully tested version)
MCHelper: An automatic tool to curate transposable element libraries

## Table of Contents  
* [Introduction](#introduction)  
* [Installation](#installation)  
* [Testing](#testing)  
* [Usage](#usage) 
* [Inputs](#inputs) 
* [Outputs](#outputs)
* [Important notes](#notes)

## Introduction
<a name="introduction"/>

Tools for de novo transposable element (TE) annotation still require substantial manual curation by a TE expert. However, manual curation of TE libraries is time-demanding and as such is unfeasible for large scale genomic studies. Moreover, it is an unreproducible process because curators make decisions depending on the expertise and the knowledge they have about TEs structure and biology. We present the Manual Curator Helper tool (MCHelper) that standardizes and automatizes the curation process. MCHelper filters low quality consensus, automatically checks their structural integrity, and extends the consensus sequences, if needed. Additionally, it can help to identify TE consensus sequences that were initially considered as unclassified elements in the raw library. In O. sativa and D. rerio, the MCHelper processed library was, on average, 12% more accurate in the number of copies annotated (in the whole genome) than the raw library generated by RepeatModeler 2 and ~40% more accurate in the euchromatic region of the D. melanogaster genome. Those annotations were also more accurately classified, and contained less short (<100 bp), and ambiguous (belonging to two or more families) copies. Also, the MCHelper processed libraries have less consensus sequences and up to 1 kb longer than the raw libraries in the three tested genomes, thus reducing the annotation times.

## Installation:
<a name="installation"/>

It is recommended to install the dependencies in an Anaconda environment. 

```
git clone https://github.com/gonzalezlab/mchelper.git
```

Then, locate the MCHelper folder and find the file named "curation.yml". Then, install the environment: 
```
conda env create -f MCHelper/curation.yml
```

Now, download the pfam databases, put it into db folder, and unzip all the databases needed by MCHelper:
```
cd MCHelper/db
wget [path_to_pfam](https://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.hmm.dat.gz) 
unzip '*.zip'
```
And that's it. You have now installed MCHelper.

## Testing:
<a name="testing"/>
To test MCHelper, we provide some example inputs and also the expected results (located at Test_dir/) to allow you to compare with your own outputs. To check MCHelper is running properly, you can do:

First, activate the anaconda enviroment:
```
conda activate curation
```

Then, be sure you are in the main folder (this one where MCHelper.py is located) and unzip the D. melanogaster genome:
```
unzip Test_dir/repet_input/Dmel_genome.zip -d Test_dir/repet_input/
```

Now, run the MCHelper script:
```
mkdir Test_dir/repet_output_own

python3 MCHelper.py -r 123 -t 8 -i Test_dir/repet_input/ -o Test_dir/repet_output_own -g Test_dir/repet_input/Dmel_genome.fasta --input_type repet -b Test_dir/repet_input/diptera_odb10.fa -a F -n Dmel
```

This test will take the REPET's output and will do the curation automatically, using most of the parameters by default.
If you want to run the test for the fasta input, you can execute:
```
unzip Test_dir/fasta_input/Dmel_genome.zip -d Test_dir/fasta_input/

mkdir Test_dir/fasta_output_own

python3 MCHelper.py -r 123 -t 8 -l Test_dir/fasta_input/Dmel-families.fa -o Test_dir/fasta_output_own -g Test_dir/fasta_input/Dmel_genome.fna --input_type fasta -b Test_dir/repet_input/diptera_odb10.fa -a F
```
## Usage:
<a name="usage"/>

Be sure you have activated the anaconda environment:
```
conda activate curation
```

Then, execute MCHelper with default parameters. For REPET input (see [Testing](#testing) for a practical example):
```
python3 MCHelper.py -i path/to/repet_output -o path/to/MCHelper_output -n repet_name_project --input_type repet -b path/to/reference_genes -a F
```

For fasta input:
```
python3 MCHelper.py -l path/to/TE_library_in_fasta -o path/to/MCHelper_output --input_type fasta -b path/to/reference_genes -a F
```

To see the full help documentation run:
```
python3 MCHelper.py --help
```

Full list of parameters include:
* -h, --help            show this help message and exit
* -r MODULE, --module MODULE: module of curation [1-3], being 123 the whole pipeline. Required*
* -i INPUT_DIR, --input INPUT_DIR:  Directory with the files required to do the curation (REPET output directory). Required*
* -g GENOME, --genome GENOME: Genome used to detect the TEs. Required*
* -o OUTPUTDIR, --output OUTPUTDIR: Path to the output directory. Required*
* --te_aid TE_AID:       Do you want to use TE-aid? [Y or N]. Default=Y
* -a AUTOMATIC:          Level of automation: F: fully automated, S: semi-automated, M: fully manual?. Default=S
* -n PROJ_NAME:          REPET project name. Required*
* -t CORES:              cores to execute some steps in parallel. Default=all available cores
* -j EXTENSION_MODULE_SEQS_FILE:  Path to the sequences to be used in the extension module.
* -k UNCLASSIFIED_MODULE_SEQS_FILE  Path to the sequences to be used in the unclassified module.
* -m REF_LIBRARY_UNCLASSIFIED_MODULE: Path to the sequences to be used as references in the unclassified module.
* -v VERBOSE            Verbose? [Y or N]. Default=N
* --input_type INPUT_TYPE:  Input type: fasta or REPET.
* -l USER_LIBRARY:       User defined library to be used with input type fasta.
* -b BUSCO_LIBRARY:      Reference/BUSCO genes to filter out TEs.
* -c MINFULLLENCOPIES:   Minimum number of full-length copies to process an element. Default=1
* -s PERC_SSR:           Maximum length covered by single repetitions (in percentage between 0-100) allowed for a TE not to be removed. Default=60
* --version             show program's version number and exit

MCHelper can be run in three different modes: Fully automatic (F), semi-automatic (S) and manual (M). The way you can control this is with the parameter **-a [F,S or M]**. Notice that the fully automatic mode will make all the decision by you and, at the end, will generate different outputs curated and non-curated sequences. In contrast, the semi-automatic mode runs the structural check and allows the user to inspect the consensus sequences that do not fit the structural requirements. The manual mode does not run the structural check and sends all the consensus sequences to manual inspection. 

MCHelper is a modular pipeline (see figure below), which can be run in a integrated way or module by module. You can control this with the -r or --module parameter, indicating which of the three modules you want to run. **If you want to run the whole pipeline, select -r 123**. Otherwise, if you want just run one of them, select the number corresponding to the module (classified module=1, unclassified module=3, and extension module=2).

<p align="center">
  <img src="https://github.com/GonzalezLab/MCHelper/blob/main/MCHelper_process.png">
</p>

## Inputs
<a name="inputs"/>
The input files required by MCHelper will depend of the tool you used to create the TE library. **If you used REPET**, the you will need the following files:

* the genome assembly 
* the library created by the TEdenovo pipeline. This library is named as projName_refTEs.fa, where projName is the name of your own REPET project. 
* the table with features created by PASTEC and is normally named projName_denovoLibTEs_PC.classif. Again, projName is the name of your own REPET project.
* a folder containing coverage plots created with the REPET tool "plotCoverage.py". This folder must be named "plotCoverage" and must be placed in the input folder specified in the -i parameter.
* a folder containing the gff files generated by the REPET tool "CreateGFF3sForClassifFeatures.py". This folder must be named "gff_reversed" and must be placed in the input folder specified in the -i parameter. 
* the full length fragment file generated by the TEannot pipeline. This files is usually named as "projName_chr_allTEs_nr_noSSR_join_path.annotStatsPerTE_FullLengthFrag.txt" where projName is the name of your own REPET project. This file must be inside of a folder named TEannot that must be inside the input folder specified in the -i parameter. 

*If you used any other tool that generates TE libraries in fasta format*, then you will need the following:
* the genome assembly 
* the library created by the tool. 

In the lastest case, MCHelper will find the information required to do the curation process. This information include: 
* to find how many full length copies and fragment have each consensus
* Structural features such as terminal repeats, and coding domains
* BLASTn, BLASTx, and tBLASTx with TE databases
 
## Outputs
<a name="outputs"/>

## Important notes
<a name="notes"/>
MCHelper is currently under development and has not yet been thoroughly tested. This version may change to improve both technical and methodological aspects. Please, if you wish to use this software, do so with moderation and always check that the results you get are more or less as expected. If you wish to report any issues, please do so in the appropriate section of this repository. **Thank you very much for your interest in MCHelper.**

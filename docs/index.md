# TB Drug resistance tutorial

## Learing objectives

* Asses the quality of raw sequence data
* Determine the genetic resistance profiles of TB samples using TB-Profiler

## Introduction

In this tutorial, we will learn how to assess the quality of raw sequence data and determine the genetic resistance profiles of TB samples using TB-Profiler. We will be analysing data that was generated for a study that aimed to investigate the genetic diversity of TB strains the Phillipines: [Whole genome sequencing analysis of Mycobacterium tuberculosis reveals circulating strain types and drug-resistance mutations in the Philippines](https://pubmed.ncbi.nlm.nih.gov/39179783/).

## Setting up the environment

These tools are written in different programming languages and have different dependencies and require a unix operating system to run. There are several options to set up the environment:
    * Install the tools on your local machine
    * Use a cloud-based platform such as AWS or Google Cloud to create a virtual machine
    * Use a containerisation platform such as Docker or Singularity


### Installing the tools on your local machine

!!! danger "Windows"    
    This tutorial assumes that you are using a unix operating system. If you are on windows, you will first need to set up wsl on your machine. You can follow the instructions [here](https://docs.microsoft.com/en-us/windows/wsl/install-win10). Following this,
    you will have access to a unix terminal and can follow the instructions below.

We will be using a conda environment to manage these dependencies. If you don't have conda installed, you can follow the instructions [here](https://conda-forge.org/download/).

### Using GitHub Codespaces

If you don't want to install the tools on your local machine, you can use GitHub Codespaces to create a cloud-based virtual machine. This will allow you to run the tools in a unix environment without having to install anything on your local machine. You'll need to have a GitHub account to use this service.

####  GitHub Codespaces

GitHub Codespaces is a cloud-based development environment that allows users to run and edit code directly in a virtual machine (VM) hosted by GitHub. It provides a fully configured workspace, pre-installed with necessary tools like Git, Python, and Docker, eliminating the need for users to set up a local environment. Every GitHub user has a certain number of free Codespace hours per month (120 for a free account), making it an ideal solution for doing some occasional bioinformatics. 

#### Visual Studio Code (VS Code)

VS Code is a lightweight yet powerful code editor that supports multiple programming languages. It also allows users to connect to GitHub Codespaces, allowing users to work on cloud-based VMs as if they were local files.
Users will launch GitHub Codespaces via VS Code, enabling them to interact with their virtual machine through a familiar coding interface.


#### How These Work Together:

1. Users launch GitHub Codespaces to create a cloud-based virtual machine.
2. They connect to the VM using VS Code, accessing the terminal and code files.

This setup allows users to work on cloud-hosted projects with both command-line and GUI access, making it ideal for bioinformatics and data analysis workflows.


### Getting set up

#### 1. Set up a GitHub account

Head over to https://github.com/ and sign up for an account. You can skip this step if you already have an account.

#### 2. Download Visual Studio Code

Get the latest version of vscode from https://code.visualstudio.com/ and follow install instructions from the website.

#### 4. Get connected

Once you have installed VScode, you can connect to your GitHub Codespace using the following steps:

1. Open Visual Studio Code.
2. Install the GitHub Codespaces extension by clicking on the following link and following instructions: [GitHub Codespaces](https://marketplace.visualstudio.com/items?itemName=GitHub.codespaces).
3. Open the command palette by pressing `Ctrl+Shift+P` (Windows/Linux) or `Cmd+Shift+P` (Mac).
4. Type `Create new Codespace` and select the option.
5. Enter `jodyphelan/tb-dr-tutorial` in the repository field.
6. You will then be proppted to select a branch. You should select the `main` branch.
7. You will be asked what type of instance to create. Select `2 cores, 8GB RAM, 32 GB storage` option.

It should take a few minutes to create the Codespace. Once it's ready, you will be able to access the terminal and files directly from VS Code.


## Set up bioinformatics software

Before we start, we need to set up the environment. We will be using the following tools:

* FastQC
* MultiQC
* TB-Profiler
* fastq-dl

We can install these software using a tool called `conda`. Conda is an open-source package management system and environment management system that runs on Windows, macOS, and Linux. Conda quickly installs, runs, and updates packages and their dependencies. Conda easily creates, saves, loads, and switches between environments on your local computer. It was created for Python programs, but it can package and distribute software for any language.

Conda should already be installed in your Codespace. You can check this by running the following command:

```bash
conda --version
```

If you see a version number, then conda is installed. If not, you can install it by following the instructions [here](https://conda.io/projects/conda/en/latest/user-guide/install/index.html). Before we start installing tools, you'll need to initialise conda by running the following command:

### Conda initialisation 

Before we start installing tools, you'll need to initialise conda by running the following command:

```bash
conda init
source ~/.bashrc
```

This will initialise conda to be loaded every time you open a new terminal. Next we need to add software channels so that conda can find the tools we want to install. Channels are the locations where packages are stored. Conda packages are downloaded from remote channels, which are URLs to directories containing conda packages. The [bioconda](https://bioconda.github.io/) channel contains bioinformatics software and the conda-forge channel contains a wide range of software packages. Use the following command to add the bioconda and conda-forge channels:

```bash
conda config --add channels bioconda
conda config --add channels conda-forge
conda config --set channel_priority strict
``` 

The actions above only need to be done once, so you won't need to run these commands again. 

### Create a new conda environment

Conda environments allow you to create isolated environments that have their own set of software installed. This is useful when you have different projects that require different versions of software. We will create a new conda environment called `tb` and install the required software in this environment. You can create a new conda environment by running the following command:

```bash
conda create -n tb fastqc multiqc tb-profiler fastq-dl
```

This will create a new conda environment called `tb` and install the required software. You can activate the environment by running the following command:

```bash
conda activate tb
```

You should see `(tb)` at the beginning of your terminal prompt, indicating that the `tb` environment is active.

### Download the data

We will be using data from the study [Whole genome sequencing analysis of Mycobacterium tuberculosis reveals circulating strain types and drug-resistance mutations in the Philippines](https://pubmed.ncbi.nlm.nih.gov/39179783/). The data is available on the European Nucleotide Archive (ENA) under the accession number [PRJEB37886](https://www.ebi.ac.uk/ena/browser/view/PRJEB37886). We will be downloading the data using the `fastq-dl` tool. You can download the data by running the following command:

```bash
fastq-dl -o data -a ERR6635398
fastq-dl -o data -a ERR6635159
fastq-dl -o data -a ERR6635124
fastq-dl -o data -a ERR6635248
fastq-dl -o data -a ERR6635174
fastq-dl -o data -a ERR6634973
```

This will download the raw sequence data for the samples with the accession numbers `ERR6635398`, `ERR6635159`, `ERR6635124`, `ERR6635248`, `ERR6635174`, and `ERR6634973` to a directory called `data`.

### Assess the quality of the raw sequence data

Before we start analysing the data, it's a good idea to assess the quality of the raw sequence data. We can do this using the `fastqc` tool. You can run `fastqc` on the raw sequence data by running the following command:

```bash
fastqc data/*.fastq.gz
```

This will generate a report for each of the raw sequence data files. We will then combine the reports into a single report using the `multiqc` tool. You can run `multiqc` on the `fastqc` reports by running the following command:

```bash
multiqc .
```

This will generate a report called `multiqc_report.html` that summarises the quality of the raw sequence data. You can open this on a web browser to view the report. Remember, if you are running this on a cloud-based platform, you will need to download the report to view it.

!!! danger "Important"
    There are a few things that are important to look out for in the `fastqc` reports:
    
    * Per base sequence quality: This shows the quality of the bases at each position in the read. You want to see a high quality score (green) across most positions. It may be normal to see a drop in quality towards the end of the read.
    


### Determine the genetic resistance profiles of TB samples

Now that we have assessed the quality of the raw sequence data, we can determine the genetic resistance profiles of the TB samples using TB-Profiler. TB-Profiler is a tool that identifies genetic mutations associated with drug resistance in TB samples. You can run TB-Profiler on the raw sequence data by running the following commands:

```bash
tb-profiler profile -1 data/ERR6635398_1.fastq.gz -2 data/ERR6635398_2.fastq.gz  -p ERR6635398 --snp_dist 100  -t 2
tb-profiler profile -1 data/ERR6635159_1.fastq.gz -2 data/ERR6635159_2.fastq.gz  -p ERR6635159 --snp_dist 100  -t 2
tb-profiler profile -1 data/ERR6635124_1.fastq.gz -2 data/ERR6635124_2.fastq.gz  -p ERR6635124 --snp_dist 100  -t 2
tb-profiler profile -1 data/ERR6635248_1.fastq.gz -2 data/ERR6635248_2.fastq.gz  -p ERR6635248 --snp_dist 100  -t 2
tb-profiler profile -1 data/ERR6635174_1.fastq.gz -2 data/ERR6635174_2.fastq.gz  -p ERR6635174 --snp_dist 100  -t 2
tb-profiler profile -1 data/ERR6634973_1.fastq.gz -2 data/ERR6634973_2.fastq.gz  -p ERR6634973 --snp_dist 100  -t 2
```

This will run TB-Profiler on each of the raw sequence data files and generate a report for each sample. The reports will contain information about the genetic resistance profiles of the samples. You can open the reports in a text editor to view the results. Specific parameters are used to specify the location of the raw sequence data files (`-1` and `-2`), the output prefix (`-p`), the SNP distance to use when performing clustering (`--snp_dist`), and the number of threads (`-t`).

We can then combine the reports into a single report using the `collate` command:

```bash
tb-profiler collate --itol
```

This will generate a few different files including:
    * tbprofiler.txt: A tab-delimited file containing the results of the TB-Profiler analysis
    * tbprofiler.variants.csv: A CSV file containing the variants identified by TB-Profiler
    * tbprofiler.transmission_graph.json: A JSON file containing the transmission graph of the samples    

Download these files to view the results. You can open the `tbprofiler.txt` file in a spreadsheet program such as Excel or Google Sheets to view the results.

## Conclusion

In this tutorial, we have learned how to assess the quality of raw sequence data and determine the genetic resistance profiles of TB samples using TB-Profiler. We have also learned how to set up a conda environment and install the required software. You can now use these tools to analyse your own TB sequence data.
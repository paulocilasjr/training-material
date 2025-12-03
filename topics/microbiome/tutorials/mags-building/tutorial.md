---
layout: tutorial_hands_on

title: "Building and Annotating Metagenome-Assembled Genomes (MAGs) from Short Metagenomics Paired Reads"
zenodo_link: ""
answer_histories:
  - label: "UseGalaxy.eu"
    history: https://usegalaxy.eu/u/berenice/h/building-and-annotating-metagenome-assembled-genomes-mags-from-short-metagenomics-paired-reads
    date: 2025-12-04
  - label: "UseGalaxy.fr"
    history: https://usegalaxy.fr/u/bebatut/h/building-and-annotating-metagenome-assembled-genomes-mags-from-short-metagenomics-paired-reads
    date: 2025-12-04
  - label: "UseGalaxy.org"
    history: https://usegalaxy.org/u/bebatut/h/building-and-annotating-metagenome-assembled-genomes-mags-from-short-metagenomics-paired-reads
    date: 2025-12-04
  - label: "UseGalaxy.org.au"
    history: https://usegalaxy.org.au/u/bebatut/h/building-and-annotating-metagenome-assembled-genomes-mags-from-short-metagenomics-paired-reads
    date: 2025-12-04
level: Intermediate
questions:
  - Why is it important to perform quality control and remove contamination from raw metagenomic reads before assembly?
  - What is the purpose of binning in metagenomic analysis? How does binning help in reconstructing metagenome-assembled genomes (MAGs) from complex microbial communities?
  - What metrics are commonly used to assess the quality of MAGs? How do completeness and contamination levels affect the reliability of downstream analyses?
  - How does taxonomic assignment contribute to the analysis of MAGs? What databases or tools can be used for assigning taxonomy, and why is it important to use up-to-date references?
  - What is the significance of functional annotation in metagenomic studies? How can tools like Bakta help uncover the biological roles of microbial communities?
objectives:
  - List the key steps involved in MAGs building from raw data.
  - Define essential terms such as MAGs (Metagenome-Assembled Genomes), binning, and functional annotation.
  - Explain the importance of preprocessing metagenomic reads, including quality control and contamination removal.
  - Describe the purpose and process of assembling, binning, and refining MAGs.
  - Compare the quality of MAGs based on completeness, contamination, and other metrics.
  - Assess the quality of MAGs and determine whether they meet standards for downstream analysis.
  - Summarize how taxonomic assignment and functional annotation contribute to understanding microbial communities.
  - Evaluate the reliability of taxonomic assignments and functional annotations based on reference databases.
  - Analyze the relative abundance of microbial taxa in the samples and infer ecological dynamics.
  - Identify the types of genomic features annotated by Bakta (e.g., CDS, rRNA, tRNA).
  - Interpret the functional annotation results to identify metabolic pathways, virulence factors, and other biological roles.
time_estimation: "6H"
key_points:
  - Preprocessing the metagenomic reads, including quality control and contamination removal, is essential for ensuring the accuracy and reliability of downstream analyses. Skipping this step can introduce errors, artifacts, and biases that compromise the integrity of the results.
  - Assembly and binning are the processes that transform raw reads into microbial genomes. Assembly reconstructs genomic sequences from metagenomic reads, while binning groups these sequences into putative microbial genomes, or MAGs. These steps are critical for resolving the taxonomic and functional diversity of complex microbial communities.
  - High-quality MAGs are essential for accurate analysis. Completeness and contamination metrics are key indicators of MAG quality. High-quality MAGs ensure reliable taxonomic assignments and functional annotations, which are necessary for meaningful biological interpretations.
  - Taxonomic and functional annotation reveal the biological roles of microbes. Taxonomic assignment connects your MAGs to known microbial lineages, facilitating comparisons with existing databases. Functional annotation uncovers the metabolic, ecological, and pathological potential of microbial communities, providing insights into their biological roles and interactions.
  - Metagenomic analysis is an iterative process. It involves multiple interconnected steps, from preprocessing to functional annotation. Each step builds on the previous one, and revisiting and refining your analyses can lead to deeper insights and more accurate conclusions. As tools and databases evolve, staying updated ensures that your analyses remain cutting-edge and reliable.
edam_ontology:
  - topic_3174 # Metagenomics
  - topic_0196 # Sequence assembly
contributions:
  authorship:
  - bebatut
  funding:
  - elixir-europe
  - ifb
subtopic: metagenomics
tags:
  - assembly
  - metagenomics
  - microgalaxy
---

Metagenomics has revolutionized our understanding of microbial communities by enabling the study of genetic material directly from environmental samples. One of the most powerful applications of metagenomics is the reconstruction of **Metagenome-Assembled Genomes (MAGs)**, i.e. near-complete or complete genomes of individual microorganisms recovered from complex microbial communities. MAGs provide invaluable insights into microbial diversity, function, and ecology, without the need for laboratory cultivation.

This tutorial is designed for anyone interested in reconstructing MAGs from paired short metagenomic reads. You will learn the essential steps, from quality control and read preprocessing to assembly, binning, refinement, and annotations of MAGs. By the end of this tutorial, you will be equipped with the knowledge, tools, and workflows to confidently generate high-quality MAGs from your own metagenomic datasets.

To guide through this process, we will use **Galaxy workflows** developed within the [FAIRyMAGs project](https://elixir-europe.org/how-we-work/scientific-programme/commissioned-services/science/bfsp/fairymags) and maintained by the [Intergalactic Workflow Commission](https://iwc.galaxyproject.org/). These workflows are available via the [IWC Workflow Library](https://iwc.galaxyproject.org/), as well as two workflow registries: [WorkflowHub](https://workflowhub.eu/) and [Dockstore](https://dockstore.org/).

### Case Study: Honey Bee Gut Microbiome

To illustrate the workflows, we will use public data from a study investigating the response of the **honey bee gut microbiota** to *Nosema ceranae* (a highly prevalent microsporidian parasite) under the influence of a probiotic and/or a neonicotinoid insecticides ({% cite sbaghdi2024response %}). The honey bee gut microbiome comprises a relatively simple core community dominated by **five bacterial lineages**, including *Snodgrassella alvi* and *Gilliamella apicola*, which play essential roles in digestion, immunity, and pathogen defense ({% cite motta2024honeybee %}). However, this microbiome is highly sensitive to abiotic stressors, such as pesticides, pollutants, and climate change ({% cite motta2024honeybee %}, {% cite ramsey2019varroa %}, {% cite alberoni2021neonicotinoids %}). For example, exposure to neonicotinoids has been linked to shifts in microbial diversity and increased susceptibility to opportunistic pathogens {% cite alberoni2021neonicotinoids %}.

While research often examines stressors in isolation, their synergistic effects remain understudied ({% cite paris2020honeybee %}, {% cite meixner2010historical %}). In the {% cite sbaghdi2024response %} study, western honey bees were experimentally infected with *N. ceranae* spores, exposed to the neonicotinoid thiamethoxam, and/or treated with the probiotic bacterium *Pediococcus acidilactici*, which is thought to enhance tolerance to *N. ceranae*. The study used deep shotgun metagenomic Illumina sequencing to analyze 21 samples, with taxonomic composition explored using MetaPhlAn4 ({% cite blanco2023extending %}).

> <comment-title></comment-title>
> Learn more about taxonomic profiling in our dedicated tutorial: [**Taxonomic Profiling and Visualization of Metagenomic Data**]({% link topics/microbiome/tutorials/taxonomic-profiling/tutorial.md %})
{: .comment}

In this tutorial, we will focus on **two representative samples** from this dataset to demonstrate the MAG reconstruction process.

Sample Alias | Treatment | Replicate | BioSample | Run
--- | --- | --- | --- | ---
IP2 | `NP`: infected with *N. ceranae* spores and treated with the probiotic | 2 | [SAMN35523843](https://www.ebi.ac.uk/ena/browser/view/SAMN35523843) | [SRR24759598](https://www.ebi.ac.uk/ena/browser/view/SRR24759598)
I1 | `N`: infected with *N. ceranae* spores | 1 | [SAMN35523839](https://www.ebi.ac.uk/ena/browser/view/SAMN35523839) | [SRR24759616](https://www.ebi.ac.uk/ena/browser/view/SRR24759616)

> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}


# Prepare Galaxy and Data

A well-organized workspace is essential for efficient and reproducible metagenomic analysis. In Galaxy, each analysis should be conducted within its own **dedicated history** to ensure clarity, avoid data mixing, and streamline collaboration or future reference.

> <hands-on-title>Prepare history</hands-on-title>
>
> 1. Create a new history for this analysis
>
>    {% snippet faqs/galaxy/histories_create_new.md %}
>
> 2. Rename the history
>
>    {% snippet faqs/galaxy/histories_rename.md %}
>
{: .hands_on}

To proceed with our analysis, we need to **import the raw metagenomic sequencing data** from the NCBI Sequence Read Archive (SRA). For this tutorial, we use two representative datasets, identified by their SRA run accession numbers: `SRR24759598` and `SRR24759616`. These datasets will serve as the starting point for our quality control, preprocessing, and MAG reconstruction workflows. Let’s retrieve and prepare these reads for downstream analysis.

> <hands-on-title>Get data from NCBI SRA</hands-on-title>
>
> 1. Create a new file with one SRA per line
>
>    ```text
>    SRR24759598
>    SRR24759616
>    ```
>
>    {% snippet faqs/galaxy/datasets_create_new_file.md %}
>
> 2. {% tool [Faster Download and Extract Reads in FASTQ](toolshed.g2.bx.psu.edu/repos/iuc/sra_tools/fasterq_dump/3.1.1+galaxy1) %}
>   - *"select input type"*: `List of SRA accession, one per line`
>      - {% icon param-file %}*"sra accession list"*: created dataset
>
> 3. Rename `Pair-end data (fasterq-dump)` collection to `Raw reads`
>
> > <comment-title></comment-title>
> > 
> > Download from NCBI SRA can take time. If it takes too long, you can import the data from Zenodo.
> > 
> > > <hands-on-title>Import raw data</hands-on-title>
> > >
> > > 1. Import the two files from [Zenodo]({{ page.zenodo_link }}) or the Shared Data library:
> > >
> > >    ```text
> > >    {{ page.zenodo_link }}/files/SRR24759598_forward.fasta
> > >    {{ page.zenodo_link }}/files/SRR24759598_reverse.fasta
> > >    {{ page.zenodo_link }}/files/SRR24759616_forward.fasta
> > >    {{ page.zenodo_link }}/files/SRR24759616_reverse.fasta
> > >    ```
> > >
> > > 2. Create a collection named `Raw reads`, rename the pairs with the sample name (`SRR24759598` and `SRR24759616`)
> > >
> > {: .hands_on}
> {: .comment}
{: .hands_on}

# Preprocess the Reads

The quality and composition of our metagenomic dataset directly influence the success of Metagenome-Assembled Genome (MAG) reconstruction. To ensure accurate, high-resolution results, it is essential to preprocess the reads through a series of critical steps:

- **Quality Control**: Assess and refine sequencing data to remove low-quality bases, adapters, and short reads that could compromise downstream analyses.
- **Contamination Removal**: Filter out host-derived sequences (e.g., honey bee DNA) and potential human contaminants, which can obscure microbial signals and introduce bias.

By meticulously preparing the reads, we create a clean, microbial-enriched dataset—the optimal foundation for robust MAG reconstruction. Let's walk through these essential preprocessing steps.

## Control Raw Data Quality

**Poor-quality reads**, such as those with low base-calling accuracy, adapter contamination, or insufficient length, can introduce errors, bias assemblies, and compromise the integrity of your results. Before proceeding with any analysis, it is essential to **assess, trim, and filter** our raw sequencing data to ensure only high-quality reads are retained.

We will use a [versatile quality control workflow](https://iwc.galaxyproject.org/workflow/short-read-qc-trimming-main/) to process our FASTQ files. This workflow is designed to:
1. **Trim and filter reads** using **fastp** ({% cite Chen_2018 %}), removing low-quality bases (default: minimum quality score of 15) and short reads (default: minimum length of 15 bases). These parameters can be adjusted based on your specific dataset and requirements.
2. Aggregate quality reports for all samples into a single, comprehensive overview using **MultiQC** ({% cite ewels2016multiqc %}), allowing to quickly visualize and compare the quality metrics across the datasets.

> <comment-title></comment-title>
> Learn more about sequencing data quality control in our dedicated tutorial: [**Quality Control**]({% link topics/sequence-analysis/tutorials/quality-control/tutorial.md %})
{: .comment}

Let's import the workflow from WorkflowHub:

{% snippet faqs/galaxy/workflows_run_wfh.md title="short-read-qc-trimming/main" wfhub_id="2025" version="1"%}

We can now launch it:

> <hands-on-title>Control raw data quality</hands-on-title>
>
> 1. Run the workflow using the following parameters
>    - {% icon param-collection %} *"Raw reads"*: `Raw reads` collection
>    - *"Qualified quality score"*: `15`
>    - *"Minimal read length"*: `15`
> 2. Inspect the MultiQC report
{: .hands_on}

> <question-title></question-title>
>
> 1. How many reads are in each dataset?
> 2. What are the length of the reads before trimming?
> 3. What are the miminum values for average bp quality score for forward and reverse reads, before and and after trimming and filtering?
>
> > <solution-title></solution-title>
> >
> > 1. 43.9 millions reads for SRR24759598 and 42.8 millions for SRR24759616
> > 2. 150 bp
> > 3. Minimum average bp quality scores given the "Sequence Quality" graph:
> > 
> >     | SRR24759598 | SRR24759616
> >    --- | --- | ---
> >    Forward (Read 1) - Before | 34.5 | 34.6
> >    Forward (Read 1) - After | 34.5 | 34.6
> >    Reverse (Read 2) - Before | 33.4 | 32.7
> >    Reverse (Read 2) - After | 33.4 | 32.7
> >
> {: .solution}
>
{: .question}

## Remove Contamination and Host Reads

Metagenomic datasets often contain **non-microbial sequences** that can compromise the accuracy and quality of downstream analyses, including MAG reconstruction. These sequences may originate from **host DNA** (in this case, the honey bee) or **external contamination**, such as human DNA introduced during sample handling or sequencing. Failing to remove these sequences can lead to misassembly, incorrect taxonomic assignments, and skewed functional interpretations.

We will now focus on filtering out both host (bee) and potential human contamination from our metagenomic reads. This critical step ensures that our dataset is enriched for microbial sequences, improving the reliability of our MAGs and enabling more precise biological insights. We will use a dedicated [workflow](https://iwc.galaxyproject.org/workflow/host-contamination-removal-short-reads-main/) to:
1. **Map reads against reference genomes** for the host (bee) and common contaminants using  **Bowtie2** ({% cite Langmead_2009 %}, {% cite Langmead_2012 %}), enabling precise identification and removal of non-microbial sequences.
2. Aggregate and visualize mapping results with **MultiQC** ({% cite ewels2016multiqc %}), providing a comprehensive overview of the filtering process and ensuring transparency in our data cleaning efforts.

Let's import the workflow from WorkflowHub:

{% snippet faqs/galaxy/workflows_run_wfh.md title="host-contamination-removal-short-reads/main" wfhub_id="2026" version="1" %}

We will run the workflow twice:
1. **First**, we focus on **filtering out host (honey bee) reads**, i.e. sequences originating from the bee itself, which can dominate the dataset and obscure microbial signals
2. **Second**, we will target **potential human contamination**, which may have been introduced during sample handling or sequencing. 

Let's begin by launching the workflow to remove honey bee-derived sequences. 

> <hands-on-title>Remove host reads</hands-on-title>
>
> 1. Run the workflow using the following parameters
>    - {% icon param-collection %} *"Short-reads"*: `Trimmed reads` collection
>    - *"Host/Contaminant Reference Genome"*: `A. mellifera genome (apiMel3, Baylor HGSC Amel_3.0)`
>
>    {% snippet faqs/galaxy/workflows_run.md %}
>
> 2. Rename `Reads without host or contaminant reads` collection to `Reads without bee reads`
> 3. Inspect the MultiQC report
>
{: .hands_on}

> <question-title></question-title>
>
> 1. Which percentage of reads mapped to the honey bee reference genome?
> 2. How many reads have been removed?
>
> > <solution-title></solution-title>
> >
> > 1. 2.8% for SRR24759598 and 1.1% for SRR24759616
> > 2. 2.8% for SRR24759598 and 1.1% for SRR24759616
> >
> {: .solution}
>
{: .question}

With the host (honey bee) sequences successfully filtered out, we now turn our attention to removing potential human contamination from the dataset. We run the workflow a second time, this time aligning the reads against a human reference genome:
 
> <hands-on-title>Remove human reads</hands-on-title>
>
> 3. Run the workflow using the following parameters
>    - {% icon param-collection %} *"Short-reads"*: `Reads without bee reads` collection
>    - *"Host/Contaminant Reference Genome"*: `Human (Homo sapiens): hg38 Full`
> 4. Rename `Reads without host or contaminant reads` collection to `Reads without bee and human reads`
> 3. Inspect the newly generated MultiQC report
>
{: .hands_on}

> <question-title></question-title>
>
> 1. Which percentage of reads mapped to the honey bee reference genome?
> 2. How many reads have been removed?
>
> > <solution-title></solution-title>
> >
> > 0.1% in both samples
> >
> {: .solution}
>
{: .question}

We have now completed the essential preprocessing steps to ensure our metagenomic dataset is **clean, high-quality, and enriched for microbial sequences**. By performing **rigorous quality control** and systematically **removing host and contaminant reads**, we have minimized potential biases and maximized the reliability of your data. With these preparations complete, our dataset is now optimized for the next phase: **assembling and reconstructing Metagenome-Assembled Genomes (MAGs)**. 

# Build, Refine, and Annotate Metagenome-Assembled Genomes (MAGs)

Now that our metagenomic reads are **clean, high-quality, and free of contamination**, we are ready to embark on the core phase of our analysis: **reconstructing and annotating Metagenome-Assembled Genomes (MAGs)**. This process transforms our processed sequencing data into **near-complete or complete microbial genomes**, enabling deeper insights into the functional potential, taxonomy, and ecological roles of the microorganisms in our samples.

We will use a comprehensive workflow to go through each step of MAG reconstruction and annotation:

![This flowchart illustrates the comprehensive workflow for building and annotating Metagenome-Assembled Genomes (MAGs) from metagenomic reads. The process begins with sample preparation, including quality control and host removal sub-workflows, followed by grouping samples. Reads are assembled into contigs using MEGAHIT and metaSPADES, and then aligned with Bowtie2. Contigs are binned using four tools: MaxBin2, SemiBin, MetaBAT2, and CONCOCT. The resulting bins are refined using Binette and dereplicated with dRep based on CheckM2 quality metrics. The best quality bins are assessed for completeness and contamination using QUAST, CheckM, and CheckM2, while abundance is estimated with CoverM. Taxonomic classification is performed using GTDB-Tk, and functional annotation is carried out with Bakta. Finally, all results are consolidated into a MultiQC report for easy visualization and analysis. The diagram uses color-coded boxes and arrows to represent each step and data flow, providing a clear and structured overview of the entire MAG reconstruction and annotation process.](images/fairymags_workflow.png)

1. **Assembly**: Reconstruct genomic sequences (contigs) from our metagenomic reads using **MEGAHIT** ({% cite Li_2015 %}) or **metaSPADES** ({% cite Nurk_2017 %}).
2. **Binning**: Group contigs into discrete genomic bins using four complementary tools: **MetaBAT2** ({% cite Kang_2019 %}), **MaxBin2** ({% cite Wu_2015 %}), **SemiBin** ({% cite Pan_2022 %}), and **CONCOCT** ({% cite Alneberg_2014 %}), each offering unique strengths for capturing microbial diversity.
3. **Refinement**: Improve the quality and completeness of the MAGs by refining bins with **Binette** ({% cite Mainguy2024 %}), the successor to metaWRAP, which removes contamination and merges related bins.
4. **Dereplication**: Consolidate the MAGs across all input samples using **dRep** ({% cite olm2017drep %}), which clusters genomes based on CheckM2 quality metrics to eliminate redundancy.
5. **Quality Assessment**: Evaluate the integrity of the MAGs with **CheckM** ({% cite Parks2015 %}) and **CheckM2** ({% cite Chklovski2023CheckM2 %}) for completeness and contamination estimates.
6. **Abundance Estimation**: Quantify the relative abundance of each MAG in our samples using **CoverM** ({% cite aroney2025coverm %}), providing insights into microbial population dynamics.
7. **Taxonomic Assignment**: Classify the MAGs using **GTDB-Tk** ({% cite Chaumeil2019 %}), aligning them with the Genome Taxonomy Database for accurate taxonomic placement.
8. **Functional Annotation**: Identify and annotate genes within our MAGs using **Bakta** ({% cite Schwengers_2021 %}), revealing their functional potential and biological roles.

All results are consolidated into a single **MultiQC** ({% cite ewels2016multiqc %}) report, allowing us to easily visualize and interpret the quality and characteristics of our MAGs.

Let's import the workflow from WorkflowHub:

{% snippet faqs/galaxy/workflows_run_wfh.md title="mags-building/main" wfhub_id="1352" version="4"%}

> <hands-on-title>Build, refine, and annotate MAGs</hands-on-title>
>
> 1. Run the workflow using the following parameters
>    - *"Trimmed reads from grouped samples"*: `Reads without bee and human reads`
>    - *"Trimmed reads"*: `Reads without bee and human reads`
>    - *"Choose Assembler"*: `MEGAHIT`
>
>      > <comment-title></comment-title>
>      > metaSPAdes is an alternative assembler.
>      >
>      > MEGAHIT is less computationally intensive and generate higher quality single and shorter contigs but shorter. metaSPAdes is very computationally intensive, but generates longer/more complete assemblies.
>      {: .comment}
>
>    - *"Minimum length of contigs to output"*: `200`
>
>    - *"Environment for the built-in model (SemiBin)"*: `global`
>
>      > <comment-title></comment-title>
>      > Environment for the built-in model (SemiBin), options are: 
>      > - `human_gut`, 
>      > - `dog_gut`,
>      > - `ocean`,
>      > - `soil`,
>      > - `cat_gut`,
>      > - `human_oral`,
>      > - `mouse_gut`,
>      > - `pig_gut`,
>      > - `built_environment`,
>      > - `wastewater`,
>      > - `chicken_caecum`,
>      > - `global`
>      {: .comment}
>
>    - *"Minimum MAG completeness percentage"*: `75`
>
>      {% snippet topics/microbiome/faqs/minimum_mag_completeness_percentage.md %}
>
>    - *"Contamination weight (Binette)"*: `2`
>
>      > <comment-title></comment-title>
>      > This weight is used for the scoring the bins. A low weight favor complete bins over low contaminated bins.
>      {: .comment}
>
>    - *"Read length (CONCOCT)"*: `100`
>
>      > <comment-title></comment-title>
>      > CONCOCT requires the read length for coverage. Best use fastQC to estimate the mean value
>      {: .comment}
>
>    - *"CheckM2 Database"*: most recent one
>
>      > <comment-title></comment-title>
>      > This database is used for quality assessment for Binette, dRep, and quality assessment of the final bins.
>      {: .comment}
>
>    - *"Minimum MAG length"*: `50000`
>
>      {% snippet topics/microbiome/faqs/minimum_mag_length.md %}
>
>    - *"ANI threshold for dereplication"*: `0.95`
>
>      {% snippet topics/microbiome/faqs/ani.md %}
>
>      {% snippet topics/microbiome/faqs/ani_dereplication_threshold.md %}
>
>    - *"Maximum MAG contamination percentage"*: `25`
>
>      {% snippet topics/microbiome/faqs/maximum_mag_contamination_percentage.md %}
>
>    - *"Run GTDB-Tk on MAGs"*: `Yes`
>    - *"GTDB-tk Database"*: most recent one
>    - *"Bakta Database"*: most recent one
>    - *"AMRFinderPlus Database for Bakta"*: most recent one
{: .hands_on}

To ensure clarity and efficiency, we will **walk through each step of the workflow** to carefully inspect the generated results. Given that executing the entire workflow can be time-consuming, we will also **provide the option to import an history with pre-computed results for each step**. This approach allows to focus on understanding the outputs and interpreting the findings without waiting for lengthy computations.

> <details-title>Import history with pre-computed results</details-title>
>
> > <hands-on-title>Import history with pre-computed results</hands-on-title>
> >
> > 1. Import the history in one of the following URLs:
> >    - [{{ page.answer_histories[0].label }}]({{ page.answer_histories[0].history }})
> >    - [{{ page.answer_histories[1].label }}]({{ page.answer_histories[1].history }})
> >    - [{{ page.answer_histories[2].label }}]({{ page.answer_histories[2].history }})
> >    - [{{ page.answer_histories[3].label }}]({{ page.answer_histories[3].history }})
> >
> >    {% snippet faqs/galaxy/histories_import.md %}
> >
> {: .hands_on}
{: .details}

## Assembly: Reconstructing Genomic Sequences from Metagenomic Reads

The first critical step in the MAG reconstruction workflow is **assembly**: the computational process of reconstructing genomic sequences, known as **contigs**, from fragmented metagenomic reads. Assembly can be likened to solving a complex jigsaw puzzle: the goal is to identify reads that "fit together" by detecting overlapping sequences. However, unlike a traditional puzzle, this task is complicated by several challenges inherent to genomics and metagenomics:

- **Genomic Complexity:** Repeated sequences, missing fragments, and errors introduced during sequencing make accurate reconstruction difficult.
- **Data Volume:** Metagenomic datasets are often massive, requiring significant computational resources.
- **Community Diversity:** Unequal representation of microbial community members—ranging from dominant to rare species—can skew assembly results.
- **Genomic Similarity:** The presence of closely related microorganisms or multiple strains of the same species adds layers of complexity.
- **Data Limitations:** Minor community members may be underrepresented, leading to incomplete or fragmented assemblies.

To address these challenges, a variety of **metagenomic assemblers** have been developed, including **metaSPAdes** ({% cite Nurk_2017 %}) and **MEGAHIT** ({% cite Li_2015 %}). Each assembler has unique computational characteristics, and their performance can vary depending on the microbiome being studied. As demonstrated by the **Critical Assessment of Metagenome Interpretation (CAMI) initiative** ({% cite sczyrba2017 %}, {% cite meyer2021 %}, {% cite meyer2022 %}), the choice of assembler often depends on the specific goals of the analysis, such as maximizing contiguity, accuracy, or computational efficiency. Selecting the right tool is essential for achieving high-quality assemblies tailored to your research objectives.

> <comment-title></comment-title>
> Learn more about metagenomic assembly, algorithms and the tools in our dedicated tutorial: [**Assembly of metagenomic sequencing data**]({% link topics/microbiome/tutorials/metagenomics-assembly/tutorial.md %})
{: .comment}

In this workflow, we used **MEGAHIT**, but **metaSPAdes** is also proposed as an alternative assembler.

> <question-title></question-title>
>
> 1. How many contigs were assembled for the SRR24759598 sample?
> 2. And for the SRR24759616 sample?
> 3. What is the minimum length of the contigs?
>
> > <solution-title></solution-title>
> >
> > 1. The MEGAHIT assembly for the SRR24759598 sample produced 534,026 contigs.
> > 2. 571,844 contigs for the SRR24759616 sample.
> > 3. When we executed the workflow, we set the *"Minimum length of contigs to output"* parameter to **200 base pairs**. Upon inspecting the resulting FASTA file, we confirmed that all contigs have a length exceeding this threshold, as indicated by the `len` attribute in the sequence headers.
> >
> {: .solution}
>
{: .question}

Once the assembly process is complete, **assessing the quality of the assembled contigs** is a crucial step to ensure the reliability and accuracy of downstream analyses. Assembly quality can be systematically evaluated using **metaQUAST** ({% cite mikheenko2016 %}), the specialized metagenomics mode in the widely used **QUAST** tool ({% cite Gurevich_2013 %}).

QUAST provides a comprehensive suite of metrics to gauge assembly performance, including:
- **Contiguity statistics**, such as the number of contigs, total assembly length, and N50/L50 values, which reflect the length and distribution of assembled sequences.
- **Genomic feature analysis**, including gene and operon prediction, to assess the biological relevance of the assembly.
- **Comparison to reference genomes** (if available), enabling the identification of structural variations, misassemblies, and potential errors.

By leveraging QUAST, we can gain valuable insights into the completeness, accuracy, and overall quality of our metagenomic assemblies, ensuring robust and meaningful results for further analysis. 

QUAST generates multiple outputs but the main output is a HTML report which aggregate different metrics.

Let's inspect the HTML reports:

> <hands-on-title>Inspect QUAST HTML reports</hands-on-title>
>
> 1. Enable Window Manager
> 
>    {% snippet faqs/galaxy/features_scratchbook.md %}
> 
> 2. Open both QUAST HTML reports
> 3. Click on `Extended report`
{: .hands_on}

At the top of each report, we find a **summary table** displaying key statistics for contigs. The rows represent various metrics, while the columns correspond to the different sample assemblies.

> <comment-title></comment-title>
> Learn more about assembly statistics generated by QUAST in the [**Assembly of metagenomic sequencing data**]({% link topics/microbiome/tutorials/metagenomics-assembly/tutorial.md %})
{: .comment}

Let's now examine the table, from top to bottom, to interpret the assembly results for each sample:

- **Statistics without reference**

   > <question-title></question-title>
   >
   > 1. How many contigs are reported for SRR24759598? And for SRR24759616?
   > 2. Why are these numbers different from the number of sequences in the output of MEGAHIT? Which statistics in the QUAST report corresponds to number of sequences in the output of MEGAHIT?
   > 3. What is the length of the longest contig in SRR24759598? And in SRR24759616?
   > 4. What is the total length in SRR24759598? And in SRR24759616?
   > 5. How similar are the microbial communities between the two samples, based on the contig statistics?
   >
   > > <solution-title></solution-title>
   > >
   > > 1. Number of contigs:
   > >    - 89,313 contigs for SRR24759598
   > >    - 85,018 contigs for SRR24759616.
   > > 2. The numbers in the QUAST report are lower because QUAST **only includes contigs longer than 500 bp** by default. The statistic # contigs (≥ 0 bp) in the QUAST report corresponds to the total number of sequences in the MEGAHIT output.
   > > 3. Length of the Longest Contig:
   > >   - 63,871 bp in SRR24759598
   > >   - 65,608 bp for SRR24759616.
   > > 4. Total Assembly Length:
   > >   - 137,737,947 bp in SRR24759598
   > >   - 117,282,673 bp for SRR24759616.
   > > 5. The two samples, SRR24759598 and SRR24759616, exhibit comparable contig statistics, suggesting a degree of similarity in their microbial communities. Both samples have a similar number of contigs (89,313 vs. 85,018) and the lengths of their longest contigs are close (63,871 bp vs. 65,608 bp). However, the total assembly length differs by about 20 million base pairs, which may indicate variations in microbial diversity, abundance, or sequencing depth between the two samples. Further analysis, such as taxonomic or functional annotation of the contigs, would be needed to determine the extent of their biological similarity.
   > {: .solution}
   >
   {: .question}

   Beyond simply counting contigs and measuring their lengths, two key statistics (**N50** and **L50**) are widely used to assess assembly quality:

   - **N50:** This value represents the length of the shortest contig in a set where the combined lengths of all contigs **equal or exceed half of the total assembly size**. A higher N50 indicates better contiguity, meaning fewer gaps and longer contiguous sequences in the assembly.
   - **L50:** This is the **minimum number of contigs** needed to cover at least 50% of the total assembly length. A lower L50 suggests that the assembly is more contiguous, as fewer, longer contigs are required to reach half of the genome's total length.

   > <details-title>Understanding N50 and L50: Key Metrics for Assembly Quality</details-title>
   >
   > The **N50 statistic** is a widely used metric to evaluate the **contiguity** of a genome assembly. When all contigs in an assembly are sorted by length, the N50 represents the **length of the shortest contig** in the set that collectively contains at least 50% of the total assembled bases. For example, if an assembly has an N50 of 10,000 base pairs (bp), it means that 50% of the total assembled sequence is contained in contigs that are **10,000 bp or longer**.
   > 
   > **Example Calculation**:
   > Consider an assembly with 9 contigs of the following lengths: **2, 3, 4, 5, 6, 7, 8, 9, and 10 bp**.
   > - The **total length** of the assembly is **54 bp**.
   > - Half of this total is **27 bp**.
   > - The sum of the three longest contigs (**10 + 9 + 8 = 27 bp**) reaches this halfway point.
   > - Therefore, the **N50 is 8 bp**, meaning that the smallest contig in the set covering half the assembly is 8 bp long.
   > 
   > **Important Considerations for N50**:
   > - **Assembly Size Matters:** When comparing N50 values across different assemblies, it is essential that the **total assembly sizes are similar**. N50 alone is not meaningful if the assemblies vary significantly in size.
   > - **Limitations of N50:** N50 does not always reflect the true quality of an assembly. For instance, consider two assemblies with the following contig lengths:
   >   - Assembly 1: **3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 25, 25, 150, 1500**
   >   - Assembly 2: **50, 500, 530, 650**
   > 
   > Both assemblies may have the **same N50**, but Assembly 2 is clearly more contiguous, with fewer and longer contigs.
   > 
   > 
   > The **L50 statistic** complements N50 by indicating the **minimum number of contigs** required to cover at least 50% of the total assembly length. In the previous example, the L50 is **3**, as only three contigs (10, 9, and 8 bp) are needed to reach half of the total assembly length.
   > 
   > **Why N50 and L50 Matter**:
   > - **N50** provides insight into the **length distribution** of contigs, with higher values indicating better contiguity.
   > - **L50** reflects the **compactness** of the assembly, with lower values suggesting fewer gaps and longer contiguous sequences.
   > 
   > Together, N50 and L50 offer a more comprehensive view of assembly quality, helping researchers assess the continuity and reliability of their genomic reconstructions.
   {: .details}

   > <question-title></question-title>
   >
   > 1. What is the N50 for SRR24759598? And for SRR24759616?
   > 2. What is the L50 for SRR24759598? And for SRR24759616?
   > 3. How do the N50 and L50 values compare between the two samples, and what does this indicate about their assembly quality?
   >
   > > <solution-title></solution-title>
   > >
   > > 1. N50 values:
   > >   - 2,214 for SRR24759598
   > >   - 1,755 for SRR24759616.
   > > 2. L50 valueS:
   > >   - 11,639 for SRR24759598
   > >   - 12,926 for SRR24759616.
   > > 3. The N50 value is higher for SRR24759598 (2,214 bp) compared to SRR24759616 (1,755 bp), indicating that the assembly for SRR24759598 is slightly more contiguous, with longer contigs covering half of the total assembly length. However, the L50 value is lower for SRR24759598 (11,639) than for SRR24759616 (12,926), meaning fewer contigs are needed to reach 50% of the total assembly length in SRR24759598. This suggests that while both assemblies are fragmented, SRR24759598 has a modestly better contiguity overall. The differences in these metrics may reflect variations in microbial diversity, sequencing depth, or assembly performance between the two samples.
   > {: .solution}
   >
   {: .question}

- **Read mapping**: results of the mapping of the raw reads on the contigs

    > <question-title></question-title>
    >
    > 1. What is the % of SRR24759598 reads mapped to SRR24759598 contigs? And for SRR24759616?
    > 2. What is the percentage of reads used to build the assemblies for SRR24759598? and SRR24759616?
    >
    > > <solution-title></solution-title>
    > >
    > > 1. % of mapped reads:
    > >   - 97.1% for SRR24759598
    > >   - 98.53% for SRR24759616
    > > 2. 97.1% of reads were used to build the contigs for SRR24759598 and 98.53% for SRR24759616.
    > {: .solution}
    >
    {: .question}

## Binning: Grouping Sequences into Microbial Genomes

**Metagenomic binning** is a computational process that classifies DNA sequences from metagenomic data into discrete groups, or **bins**, based on their similarity. The primary goal is to **assign sequences to their original organisms or taxonomic groups**, enabling researchers to reconstruct individual microbial genomes and explore the diversity and functional potential of complex microbial communities.

Binning relies on a combination of **sequence composition, coverage, and similarity** to group contigs into bins. These bins ideally represent the genomes of distinct microorganisms present in the sample. However, the process is inherently challenging due to:
- **Genomic complexity**, such as repeated sequences and strain-level variation.
- **Uneven sequencing coverage** across different microbial species.
- **Contamination and chimeras**, which can obscure the boundaries between genomes.

Numerous algorithms have been developed for metagenomic binning, each with unique strengths and limitations. The choice of tool often depends on the dataset's characteristics and the research objectives. Benchmark studies (e.g., {% cite sczyrba2017 %}, {% cite meyer2021 %}, {% cite meyer2022 %}) provide practical guidance on selecting the most suitable binner for specific environments and datasets.

> <comment-title></comment-title>
> Learn more about metagenomic binning and available tools in our dedicated tutorial: [**Binning of metagenomic sequencing data**]({% link topics/microbiome/tutorials/metagenomics-binning/tutorial.md %})
{: .comment}

A widely adopted strategy is to **use multiple binning tools** to maximize accuracy, followed by **bin refinement** to consolidate and improve the results. This approach leverages the strengths of different algorithms, producing a more robust and reliable set of bins.

In this analysis, we employed **four complementary binning tools**, each offering distinct advantages for capturing microbial diversity:
- **MetaBAT2** ({% cite Kang_2019 %}): Known for its efficiency and accuracy in binning metagenomic contigs.
- **MaxBin2** ({% cite Wu_2015 %}): Utilizes probabilistic models to improve binning accuracy.
- **SemiBin** ({% cite Pan_2022 %}): Incorporates deep learning to enhance binning performance, especially for complex datasets.
- **CONCOCT** ({% cite Alneberg_2014 %}): Uses Gaussian mixture models to cluster contigs based on coverage and composition.

Let's know look at the results of each binner for each sample.

> <hands-on-title>Inspect bins for each binner</hands-on-title>
>
> 1. Get number of bins generated by **MetaBAT2**
>    - Open the `MetaBAT2 on collection X and collection Y: Bin sequences` collection
>    - Extract the number of bin, i.e. the number of element for each sample list
> 2. Get number of bins generated by **MaxBin2**
>    - Open the `MaxBin2 on collection X and collection Y: Bins` collection
>    - Extract the number of bin, i.e. the number of element for each sample list
> 3. Get number of bins generated by **SemiBin**
>    - Open the `SemiBin on collection X and collection Y: Reconstructed bins` collection
>    - Extract the number of bin, i.e. the number of element for each sample list
> 4. Get number of bins generated by **CONCOCT**
>    - Open the `CONCOCT: Extract a fasta file on collection X and collection Y : Bins` collection
>    - Extract the number of bin, i.e. the number of element for each sample list
>
{: .hands_on}

> <question-title></question-title>
>
> 1. How many bins have been found for each sample and binner?
> 2. Which binning tools generated the most and least bins for the samples?
> 3. Is the trend in the number of bins generated consistent across both samples?
>
> > <solution-title></solution-title>
> >
> > 1. Number of bins per sample and binner:
> >
> >    Binner | SRR24759598 | SRR24759616
> >    --- | --- | ---
> >    MetaBAT2 | 35 | 27
> >    MaxBin2 | 42 | 37
> >    SemiBin | 45 | 36
> >    CONCOCT | 81 | 62
> > 
> > 2. For both samples:
> >    - MetaBAT2 generated the fewest bins, indicating a more conservative approach to binning.
> >    - CONCOCT produced the most bins, suggesting a tendency to create finer, potentially more fragmented groupings.
> >    - MaxBin2 and SemiBin yielded a similar number of bins, reflecting comparable performance in terms of bin granularity.
> >
> > 3. Yes, regardless of the binning tool used, SRR24759598 consistently produced more bins than SRR24759616. This suggests that the microbial community in SRR24759598 may be more complex, diverse, or fragmented compared to SRR24759616.
> {: .solution}
>
{: .question}

Beyond simply comparing the total number of bins, we can also examine the **contigs per bin** for each binning tool, which provides deeper insight into the **quality and granularity** of the reconstructed microbial genomes. 

For that, we will use the `collection X, collection Y, and others (as list)` collection of collection. This structure contains two sub-collections—one for each sample. Within each sub-collection, there are four tables, each corresponding to a different binning tool (MetaBAT2, MaxBin2, SemiBin, and CONCOCT). Each table consists of two columns: the contig identifier and its assigned bin ID. 
We will group the contigs by their bin IDs and count the number of contigs in each bin. This will allow us to compute statistics and evaluate how contigs are distributed across bins for each binning tool, providing insights into the quality and granularity of the bins.

> <hands-on-title>Get statistics for the number of contigs per bin for each binner</hands-on-title>
>
> 1. {% tool [Group data by a column and perform aggregate operation on other columns](Grouping1) %} with following parameters:
>    - {% icon param-collection %} *"Select data"*: `collection X, collection Y, and others (as list)`
>    - *"Group by column"*: `Column: 2`
>    - {% icon param-repeat %} *Insert Operation*
>       - *"Type"*: `Count`
>       - *"On column"*: `Column: 1`
> 
> 2. {% tool [Summary Statistics](Summary_Statistics1) %} with following parameters:
>    - {% icon param-collection %} *"Summary statistics on"*: output of **Group data**
>    - *"Column or expression"*: `c2`
> 
> 3. {% tool [Collapse Collection](toolshed.g2.bx.psu.edu/repos/nml/collapse_collections/collapse_dataset/5.1.0) %} with following parameters:
>    - {% icon param-collection %} *"Collection of files to collapse into single dataset"*: output of **Summary Statistics**
{: .hands_on}

Now, we have **two tables**—one for each sample—containing **statistics on the number of contigs per bin**. Each table is organized with **one row per binning tool**, listed in the following order:
- **CONCOCT**
- **MetaBAT2**
- **MaxBin2**
- **SemiBin**

> <question-title></question-title>
>
> 1. What are the statistics of number of contigs per bin for each sample and binner?
> 2. How do you interpret the binning tool statistics for the samples?
> 3. Is the trend in the number of contigs per bin consistent across both samples?
> 4. What are the practical implications?
>
> > <solution-title></solution-title>
> >
> > 1. Statistics for SRR24759598
> >
> >    Binner | Sum | mean | stdev | 0% | 25% | 50% | 75% | 100%
> >    --- | --- | --- | --- | --- | --- | --- | --- | ---
> >    CONCOCT | 33700 | 416.049 | 1047.46 | 1 | 2 | 17 | 132 | 5640
> >    MetaBAT2 | 8871 | 253.457 | 231.777 | 2 | 97.5 | 158 | 417.5 | 809
> >    MaxBin2 | 25686 | 611.571 | 676.384 | 7 | 59.25 | 477.5 | 931.75 | 2607
> >    SemiBin | 7727 | 171.711 | 136.596 | 4 | 71 | 131 | 252 | 602 
> > 
> >    Statistics for SRR24759616
> >
> >    Binner | sum | mean | stdev | 0% | 25% | 50% | 75% | 100%
> >    --- | --- | --- | --- | --- | --- | --- | --- | ---
> >    CONCOCT | 29492 | 475.677 | 1227.33 | 1 | 1.25 | 22 | 339 | 6792
> >    MetaBAT2 | 7703 | 285.296 | 307.37 | 3 | 72 | 185 | 306 | 1204
> >    MaxBin2 | 25660 | 693.514 | 613.431 | 2 | 248 | 644 | 972 | 3056
> >    SemiBin | 6315 | 175.417 | 143.068 | 8 | 64.75 | 161 | 237.5 | 649
> > 2. General Trends Across Binners
> >    - CONCOCT consistently used less contigs to generate the bins (sum) for both samples, but with a wide range of contigs per bin (high standard deviation). This indicates that CONCOCT tends to create many small bins, some of which may be fragmented or incomplete.
> >    - MetaBAT2 uses fewer contigs and shows lower variability in the number of contigs per bin. This suggests a more conservative approach, likely producing more consolidated and potentially higher-quality bins.
> >    - MaxBin2 uses a moderate number of bins but with a higher mean number of contigs per bin compared to MetaBAT2 and SemiBin. Its standard deviation is also relatively high, indicating variability in bin sizes.
> >    - SemiBin uses the fewest bins (after MetaBAT2) and shows low variability in contig counts per bin, suggesting a balanced and consistent binning approach.
> >
> >    Distribution of Contigs per Bin
> >    - CONCOCT:
> >      - SRR24759598: The median number of contigs per bin is 17, but the distribution is highly skewed, with some bins containing up to 5,640 contigs. This suggests potential over-splitting of contigs into many small bins.
> >      - SRR24759616: The median is 22, with a maximum of 6,792 contigs per bin, indicating a similar trend of over-splitting.
> >    - MetaBAT2:
> >      - SRR24759598: The median number of contigs per bin is 158, with a relatively narrow interquartile range (97.5 to 417.5). This indicates a more even distribution of contigs across bins.
> >      - SRR24759616: The median is 185, with a similar distribution pattern, suggesting consistent binning performance.
> >    - MaxBin2:
> >      - SRR24759598: The median number of contigs per bin is 477.5, with a broad distribution (59.25 to 931.75). This indicates variability in bin sizes, with some bins containing many contigs.
> >      - SRR24759616: The median is 644, with a similar broad distribution, reflecting a tendency to create larger bins.
> >    - SemiBin:
> >      - SRR24759598: The median number of contigs per bin is 131, with a tight interquartile range (71 to 252). This suggests a uniform binning process.
> >      - SRR24759616: The median is 161, with a similar tight distribution, indicating consistent binning.
> >
> > 3. For both samples, CONCOCT and MaxBin2 produce more bins than MetaBAT2 and SemiBin, but the distribution of contigs per bin is more variable for CONCOCT. SRR24759598 generally has slightly more bins across all tools compared to SRR24759616, which may reflect differences in microbial diversity or sequencing depth between the samples. MetaBAT2 and SemiBin show similar trends in both samples, with relatively consistent bin sizes and fewer extreme outliers.
> >
> > 4. CONCOCT may be useful for capturing fine-scale diversity but may require additional refinement to merge small or fragmented bins. MetaBAT2 and SemiBin are likely to produce more reliable and consolidated bins, making them suitable for downstream analyses like genome reconstruction and functional annotation. MaxBin2 offers a balance between the number of bins and the number of contigs per bin but may require further validation due to its broader distribution.
> >
> {: .solution}
>
{: .question}

We can also evaluate the binning with **CheckM2** ({% cite Chklovski2023CheckM2 %}). CheckM2 is a software tool used in metagenomics binning to assess the completeness and contamination of genome bins. It compares the genome bins to a set of universal single-copy marker genes that are present in nearly all bacterial and archaeal genomes. By identifying the presence or absence of these marker genes in the bins, CheckM2 can estimate the completeness of each genome bin (i.e., the percentage of the total set of universal single-copy marker genes that are present in the bin) and the degree of contamination (i.e., the percentage of marker genes that are found in more than one bin).

> <hands-on-title>Assessing the completeness and contamination of bins for each binner</hands-on-title>
>
> 1. Find and run the **Quality control of bins generated by MetaBAT2, MaxBin2, SemiBin, and CONCOCT** public workflow with parameters:
>    
>    {% snippet faqs/galaxy/workflows_run.md tab="Public workflows" name="Quality control of bins generated by MetaBAT2, MaxBin2, SemiBin, and CONCOCT" %}
>     
>    - {% icon param-collection %} *"MetaBAT2 Bins"*: `MetaBAT2 on collection X and collection Y: Bin sequences`
>    - {% icon param-collection %} *"MaxBin2 Bins"*: `MaxBin2 on collection X and collection Y: Bins`
>    - {% icon param-collection %} *"SemiBin Bins"*: `SemiBin on collection X and collection Y: Reconstructed bins`
>    - {% icon param-collection %} *"CONCOCT Bins"*: `CONCOCT: Extract a fasta file on collection X and collection Y : Bins`
> 2. Inspect the `MultiQC for CheckM on bins right after binners` file
>
{: .hands_on}

> <question-title></question-title>
>
> 1. How many bins have a predicted completeness of 100% and greater than 90% for each sample? Are these high-completeness bins distributed across all binning tools?
> 2. How many of the bins with a predicted completeness greater than 90% have a predicted contamination above 50%? What does this imply about the quality of these bins?
>
> > <solution-title></solution-title>
> >
> > 1. SRR24759598:
> >
> >    Binner | Predicted completness = 100% | Predicted completness > 90%
> >    --- | --- | --
> >    MetaBAT2 | 1 | 4
> >    MaxBin2 | 0 | 1
> >    SemiBin | 2 | 4
> >    CONCOCT | 4 | 10
> >    **Total** | 7 | 19
> >
> >    SRR24759616:
> >
> >    Binner | Predicted completness = 100% | Predicted completness > 90%
> >    --- | --- | --
> >    MetaBAT2 | 1 | 3
> >    MaxBin2 | 0 | 1
> >    SemiBin | 1 | 2
> >    CONCOCT | 3 | 8
> >    **Total** | 5 | 14
> > 
> >    CONCOCT produces the highest number of high-completeness bins (both 100% and >90%) for both samples, suggesting it may be more effective at reconstructing near-complete genomes in these datasets. However, all binning tools contribute to the high-completeness bins, with MetaBAT2 and SemiBin also generating notable numbers of high-quality bins. This indicates that combining multiple binning tools can enhance the recovery of complete and near-complete microbial genomes.
> > 
> > 2. Number of bins with completeness > 90% that also have contamination > 50%
> > 
> >    Binner | SRR24759598 | SRR24759616
> >    --- | --- | --
> >    MetaBAT2 | 0 / 4 | 1 / 3
> >    MaxBin2 | 0 / 1 | 0 / 1
> >    SemiBin | 0 / 1 | 0 / 2
> >    CONCOCT | 5 / 10 | 5 / 8
> >    **Total** | 5 / 19 | 6 / 14
> >
> >    Implications:
> >    - A significant proportion of **high-completeness bins from CONCOCT** (50% for SRR24759598 and 62.5% for SRR24759616) exhibit **contamination levels above 50%**. This suggests that while CONCOCT effectively captures near-complete genomes, it may also include **substantial contamination from other organisms**, potentially compromising the accuracy of downstream analyses.
> >    - MetaBAT2, MaxBin2, and SemiBin produce **fewer contaminated high-completeness bins**, indicating that their bins are generally more reliable and purer for further genomic studies.
> >    - These results highlight the importance of **post-binning refinement** to reduce contamination and improve the quality of high-completeness bins, especially when using tools like CONCOCT.
> >
> {: .solution}
>
{: .question}

## Bin Refinement: Enhancing the Quality and Completeness of Bins

Metagenomic binning is a powerful approach for reconstructing microbial genomes from complex environmental samples. However, as we have seen, the bins produced by the individual binning tools vary in quality. While these tools excel at capturing microbial diversity, their outputs may include **fragmented, redundant, or contaminated bins**, which can compromise downstream analyses.

To address these challenges, **bin refinement** is a critical next step. This process aims to **improve the quality and completeness of MAGs** by:
- **Removing contamination** from bins, ensuring that each bin represents a single, coherent genome.
- **Merging related bins** that may have been incorrectly split by individual binning tools, thereby increasing completeness.

In this workflow, we refine our bins using **[Binette](https://github.com/genotoul-bioinfo/Binette)** ({% cite Mainguy2024 %}), the successor to **metaWRAP**. Binette leverages advanced algorithms to **automatically detect and remove contamination** while **merging bins that likely originate from the same microbial genome**. By integrating results from multiple binning tools, Binette enhances the accuracy and reliability of MAGs, making them more suitable for downstream analyses such as taxonomic classification, functional annotation, and comparative genomics.

Binette generates several collections: one with conserved bins and two collections of quality reports. Let's inspect the final quality reports:

> <hands-on-title>Inspect QUAST HTML reports</hands-on-title>
>
> 1. Enable Window Manager
> 
>    {% snippet faqs/galaxy/features_scratchbook.md %}
> 
> 2. Open both Binette Final Quality Reports
{: .hands_on}

> <question-title></question-title>
>
> 1. How many bins are left after refinement?
> 2. What are the different columns in the reports?
> 3. What are the different operations in the **origin** column?
> 4. What is the minimal value for completeness in the refined bins?
> 5. How many bins have a completeness greater than 99%, and what does this indicate?
> 6. What are the maximum contamination values and what does this indicate?
>
> > <solution-title></solution-title>
> >
> > 1. 13 for SRR24759598 and 14 for SRR24759616
> > 2. Column in the reports
> >
> >    Column Name | Description
> >    --- | ---
> >    **bin_id** | This column displays the unique ID of the bin.
> >    **origin**  | Indicates the source or origin of the bin, specifying from which bin set it originates or the intermediate set operation that created it.
> >    **name** | The name of the bin.
> >    **completeness** | The completeness of the bin, determined by CheckM2.
> >    **contamination** | The contamination of the bin, determined by CheckM2.
> >    **score** | Computed score: `completeness - contamination * weight`. The contamination weight can be customized using the `--contamination_weight` option.
> >    **size** | Total size of the bin in nucleotides.
> >    **N50** | The N50 of the bin, representing the length for which 50% of the total nucleotides are in contigs of that length or longer.
> >    **contig_count** | Number of contigs contained within the bin.     
> > 
> > 3. When at least two bins overlap (i.e., share at least one contig), Binette applies fundamental set operations to generate new bins:
> >   - Intersection: the generated bin contains contigs that are shared by overlapping bins.
> >   - Difference: the generated bin includes contigs that are unique to one bin and not found in others.
> >   - Union: the generated bin encompasses all contigs from the overlapping bins.
> >
> > 4. The **minimal completeness value** observed in the refined bins is 76.92% for SRR24759598 and 75.19% for SRR24759616. These values align with the **75% completeness threshold** set in the refinement tool, ensuring that only high-quality bins are retained.
> > 5. 4 bins for SRR24759598 and 2 bins for SRR24759616 exhibit a completeness of greater than 99%. This indicates that these bins are **near-complete representations of their respective microbial genomes**, suggesting high-quality genome reconstructions suitable for in-depth functional and taxonomic analyses.
> > 6. The maximum contamination values are 47.01% for SRR24759598 and 77.2% for SRR24759616. These values indicate that some bins still contain a significant proportion of sequences from multiple organisms, which can affect the accuracy of downstream analyses. High contamination levels suggest that further refinement or manual curation may be necessary to improve the purity of these bins and ensure reliable genomic interpretations.
> >
> {: .solution}
>
{: .question}


## De-Replication: Ensuring Non-Redundancy in MAGs

**De-replication of Metagenome-Assembled Genomes (MAGs)** is a crucial step in MAGs building that reduces redundancy by identifying and retaining only the **highest-quality representative MAG** from each set of highly similar genomes. This process involves addressing key questions, such as:
- **What threshold of similarity defines redundant MAGs?** (e.g., \\( >\\) 99%  average nucleotide identity)
- **How do we determine which MAG is the "best" representative?** (e.g., based on completeness, contamination, and assembly quality)
- **How can we ensure the final MAG set is both representative and non-redundant?**

When metagenomic samples are assembled individually, **redundant MAGs** (often representing the same microbial population) can distort downstream analyses ({% cite shaiber2019composite %}). 

![Image shows the process of individual assembly on two strains and five samples, after individual assembly of samples two samples are chosen for de-replication process. In parallel, co-assembly on all five samples is performed](./images/individual-assembly.png "Individual assembly followed by de-replication vs co-assembly"){:width="80%"}

For example, maintaining redundancy in a MAG dataset leads to challenges during the step of **mapping reads on MAGs**, where sequencing reads may align to multiple highly similar MAGs. This can result in:
- **Artificially low abundance estimates** for each taxon, as reads are distributed across redundant MAGs rather than accurately reflecting true population levels ({% cite evans2020dereplicate %}).
- **Misinterpretation of population dynamics**, where multiple redundant MAGs may falsely suggest the presence of distinct but ecologically equivalent populations, rather than a single abundant population spanning multiple samples.
- **Complications in manual curation**, particularly when using tools like **Anvi'o**. The presence of multiple similar MAGs—some of which may be incomplete—can lead to **unreliable differential coverage patterns**, potentially causing the incorrect removal of genomic regions that belong to the same population.

To mitigate these issues, studies have applied various similarity thresholds for de-replication, including:
- **\\( > \\) 95% average nucleotide identity** ({% cite almeida2019new %})
- **\\( > \\) 98% average nucleotide identity** ({% cite anantharaman2016thousands %})
- **\\( > \\) 99.5% amino acid identity** ({% cite parks2017recovery %})

{% snippet topics/microbiome/faqs/ani.md %}

**[dRep](https://drep.readthedocs.io/)** ({% cite olm2017drep %}) is a widely used tool designed to **cluster MAGs based on sequence similarity** and retain only the highest-quality representative from each cluster. By incorporating **CheckM2 quality metrics**, dRep ensures that the final MAG set is **non-redundant and optimized for downstream analyses**, such as taxonomic profiling and functional annotation.

In this workflow, we use **dRep** to **consolidate MAGs across all input samples**. For that, the bins generated by **Binette** across all samples (here 2) are pooled together and the **CheckM2** is run to evaluate their quality. The pooled bins, the CheckM2 report are given as input to **dRep** together with several values we defined when we launched the workflow:

- *"Minimum MAG completeness percentage"*: `75`
  
  {% snippet topics/microbiome/faqs/minimum_mag_completeness_percentage.md %}

- *"Maximum MAG contamination percentage"*: `25`

  {% snippet topics/microbiome/faqs/maximum_mag_contamination_percentage.md %}

- *"Minimum MAG length"*: `50000`
   
  {% snippet topics/microbiome/faqs/minimum_mag_length.md %}

- *"ANI threshold for dereplication"*: `0.95`

  {% snippet topics/microbiome/faqs/ani_dereplication_threshold.md %}

> <question-title></question-title>
>
> 1. How many bins were given as inputs to dRep? How many have been kept after dRep?
> 2. What does MASH Clustering Dendrogram (in `Primary_clustering_dendrogam.pdf`) show? How were the bins combined?
> 
>   ![This dendrogram visualizes the MASH clustering of Metagenome-Assembled Genomes (MAGs) based on their Average Nucleotide Identity (ANI), with the x-axis representing the MASH ANI values ranging from 0.0 to 100.0. Each leaf in the tree corresponds to a MAG, labeled with its sample identifier, bin number, and a pair of values in parentheses indicating the number of single-copy marker genes identified and the number of contaminant markers detected (e.g., (14_0) means 14 single-copy genes and 0 contaminants). The vertical dotted line marks a clustering threshold, grouping MAGs with high ANI values (close to 100) on the left, indicating high similarity, while MAGs on the right are more divergent. Colored brackets highlight clusters of closely related MAGs, illustrating how MASH clustering groups similar genomes together for downstream de-replication or analysis. This visualization helps identify redundant MAGs and ensures that only representative, high-quality genomes are retained.](./images/drep_primary_clustering_dendrogam.png)
>
> > <solution-title></solution-title>
> >
> > 1. 27 bins were given as inputs to dRep (13 for SRR24759598 and 14 for SRR24759616). dRep kept only 16 of them given the `Widb.csv` file and the number of fasta in `dereplicated_genomes` collection
> > 2. This dendrogram illustrates the **MASH clustering** of Metagenome-Assembled Genomes (MAGs) based on their **Average Nucleotide Identity (ANI)**. The x-axis represents the **MASH ANI values**, ranging from 0.0 to 100.0, indicating the similarity between genomes. Each leaf in the tree corresponds to a MAG, labeled with its **sample identifier, bin number, and a pair of values in parentheses** (e.g., `(14_0)`), which represent the **number of single-copy marker genes identified** and the **number of contaminant markers detected**, respectively.
> > 
> >    **Key Observations from the Dendrogram**
> >    1. **Clustering by ANI:** MAGs with **high ANI values** (close to 100) are clustered on the **left side** of the dendrogram, indicating high similarity. MAGs with **lower ANI values** are positioned towards the **right side**, indicating greater divergence.
> >    2. **Colored Brackets:** The colored brackets highlight clusters of closely related MAGs, showing how MASH clustering groups similar genomes together. These clusters are useful for identifying **redundant MAGs** that can be combined or de-replicated.
> >    3. **Vertical Dotted Line:** The vertical dotted line marks a **clustering threshold**, separating highly similar MAGs (left) from more divergent ones (right).
> >
> >    **How Were the Bins Combined?**
> >
> >    The bins were combined based on their **ANI values** and **clustering results** from the MASH algorithm. Here’s how the process works:
> >    1. **MASH Clustering:** The MASH algorithm calculates the **ANI** between all pairs of MAGs, creating a similarity matrix. MAGs with ANI values above a certain threshold (often **95% or higher**) are considered **sufficiently similar** to be grouped into the same cluster.
> >    2. **Cluster Formation:** MAGs within each cluster are considered **redundant or highly similar**. From each cluster, the **highest-quality MAG** (based on completeness, contamination, and other quality metrics) is selected as the **representative genome**.
> >    3. **De-Replication:** Redundant MAGs within each cluster are **removed or combined** with the representative genome. This process ensures that the final set of MAGs is **non-redundant** and of the highest possible quality.   
> >
> {: .solution}
>
{: .question}

By clustering MAGs based on their quality and similarity, dRep eliminates redundancy, producing a **high-confidence, non-redundant MAG set** that is ideal for further metagenomic studies. This step is essential for improving the accuracy and reliability of abundance estimates, population dynamics, and functional interpretations.

## Quality Assessment of Metagenome-Assembled Genomes (MAGs)

To ensure the **reliability and accuracy** of our Metagenome-Assembled Genomes (MAGs), we assess their **completeness** and **contamination** using two state-of-the-art tools:
- {% tool [CheckM](toolshed.g2.bx.psu.edu/repos/iuc/checkm_lineage_wf/checkm_lineage_wf/1.2.4+galaxy2) %} ({% cite Parks2015 %})
- {% tool [CheckM2](toolshed.g2.bx.psu.edu/repos/iuc/drep_dereplicate/drep_dereplicate/3.6.2+galaxy1) %} ({% cite Chklovski2023CheckM2 %})

These tools provide **quantitative estimates** of genome quality, helping us identify high-confidence MAGs for downstream analyses. **CheckM** leverages a database of lineage-specific marker genes, while **CheckM2** uses a machine learning approach to improve accuracy and speed, especially for novel or poorly characterized genomes.

{% snippet topics/microbiome/faqs/checkm_checkm2.md %}

By evaluating both completeness and contamination, we can ensure that our MAGs are **sufficiently complete** to represent near-full genomes and **sufficiently pure** to avoid misinterpretations in functional and taxonomic analyses.

> <question-title></question-title>
>
> 1. How many bins have a predicted completeness of 100% and greater than 90% by **CheckM2**?
> 2. What the different marker lineages and their taxonomic levels found for the bins by **CheckM**?
>
> > <solution-title></solution-title>
> >
> > 1. 2 bins have a predicted completeness of 100% and 4 bins a predicted completeness greater than 90%.
> > 
> > 2. Marker lineages:
> > 
> >    Lowest Lineage | Level | Number of bins
> >    --- | --- | ---
> >    k__Bacteria (UID203) | Kingdom | 1
> >    c__Gammaproteobacteria (UID4387) | Class | 3
> >    c__Betaproteobacteria (UID3888) | Class | 1
> >    o__Lactobacillales (UID463) | Order | 4
> >    o__Clostridiales (UID1226) | Order | 2
> >    f__Enterobacteriaceae (UID5121) | Family | 2
> >    f__Bifidobacteriaceae (UID1458) | Family | 1
> >    g__Lactobacillus (UID377) | Genus | 2
> >
> {: .solution}
>
{: .question}

## Microbial Abundance Estimation from Metagenome-Assembled Genomes (MAGs)

Understanding the **relative abundance** of microbial populations is a key step in metagenomic analysis. Once high-quality Metagenome-Assembled Genomes (MAGs) are recovered, their abundance can be estimated by **aligning sequencing reads** to their contigs and calculating **genome coverage statistics**.

Genome coverage is determined by **summing the total aligning bases** across all contigs in a MAG and normalizing by the **total genome length**. This metric provides a quantitative measure of how abundant each microbial population is within the sample. To ensure accuracy, **coverage filters** are often applied to exclude false positives—genomes that appear present due to sporadic or low-coverage read alignments.

**CoverM** ({% cite aroney2025coverm %}) is a powerful and scalable tool designed for computing **per-genome coverage statistics**. By using CoverM, we efficiently calculate coverage across large datasets, enabling robust insights into **microbial population dynamics** and community composition.

> <hands-on-title>Inspect CoverM results</hands-on-title>
>
> 1. Inspect CoverM results in the `MultiQC on data X, data Y, and others: Webpage` file
>
{: .hands_on}

> <question-title></question-title>
>
> 1. What does the relative abundance (%) represent in this table?
> 2. What are the percentages of unmapped reads for both samples? 
> 3. What do these unmapped reads represent?
> 4. What is the relative abundance of `SRR24759598_bin_61470` in each sample, and what does this indicate about its presence in the microbial community?
> 5. What is the relative abundance of `SRR24759616_bin_59648` in each sample, and what does this indicate about its presence in the microbial community?
> 6. Why do some MAGs show a significant difference in relative abundance between the two samples, and what could this indicate?
>
> > <solution-title></solution-title>
> >
> > 1. The relative abundance (%) in this table represents the proportion of sequencing reads that map to each MAG relative to the total mapped reads in each sample. It provides a measure of how prevalent each microbial population is within a given sample.
> > 2. For SRR24759598, 71.4% of the sequencing reads were not mapped to the Metagenome-Assembled Genomes (MAGs). For SRR24759616, 74.7% of the reads remained unmapped.
> > 3. Unmapped reads typically indicate sequences that could not be aligned to the assembled MAGs. This can occur due to several reasons:
> >    - **Uncharacterized Microbes**: The reads may originate from microbial species or strains not represented in the MAGs, either because they are low-abundance or difficult to assemble.
> >    - **Non-Microbial DNA**: Some reads may come from host DNA, viruses, phages, or environmental contaminants that are not captured in the MAGs.
> >    - **Technical Limitations**: Short or low-quality reads, as well as repetitive genomic regions, may fail to align reliably to the MAGs.
> >    - **Incomplete Assemblies**: If the MAGs are fragmented or incomplete, some reads from those genomes may not map successfully.
> >
> >    Understanding the proportion and potential sources of unmapped reads is crucial for interpreting the completeness of our metagenomic analysis and identifying areas for further investigation.
> >
> > 4. `SRR24759598_bin_61470` has a relative abundance of 11.1% in SRR24759598 and 10.5% in SRR24759616, indicating that this microbial population is **similarly abundant** in both samples.
> > 5. According to the table, `SRR24759616_bin_59648` has a relative abundance of 0.7% in SRR24759598 and 2.1% in SRR24759616.
> >
> >    The microbial population represented by `SRR24759616_bin_59648` is approximately three times more abundant in the SRR24759616 sample compared to SRR24759598. This suggests that the environmental conditions, biological interactions, or sample-specific factors in SRR24759616 may favor the growth or persistence of this microbial population.
> > 
> >    Such differences in abundance can indicate **shifts in microbial community structure** between the two samples. This could be due to variations in nutrient availability, environmental stressors, or interactions with other microbes or hosts.
> >
> >    If this bin represents a microbe of interest (e.g., a pathogen, a key metabolic contributor, or a biomarker), its higher abundance in SRR24759616 may warrant further investigation to understand its functional role or ecological impact in that sample.
> >   
> > 6. Significant differences in relative abundance between the two samples can indicate several biological or technical factors:
> >   - **Environmental or Biological Variations**: Differences in microbial abundance may reflect changes in environmental conditions, nutrient availability, or host interactions between the two samples. For example, a microbial population that thrives under specific conditions may be more abundant in one sample than the other.
> >   - **Sample-Specific Microbial Populations**: Some MAGs may represent unique or specialized microbial populations that are present in only one sample or at significantly different levels. For instance, SRR24759598_bin_52557 has a relative abundance of 1.9% in SRR24759598 but only 0.2% in SRR24759616, suggesting it is much more prevalent in the first sample.
> >   - **Technical Factors**: Differences in sequencing depth, library preparation, or assembly quality can also affect relative abundance estimates. For example, if one sample was sequenced more deeply, low-abundance microbial populations might be detected more reliably.
> >   - **Biological Competition or Interaction**: Microbial populations may compete for resources or interact in ways that affect their abundance. A decrease in one population might correspond to an increase in another due to competitive exclusion or symbiosis.
> >
> {: .solution}
>
{: .question}

## Taxonomic Assignment of the MAGs

Taxonomic assignment is a critical step in metagenomic analysis, enabling us to **identify and classify** microbial genomes based on their genetic content. By assigning taxonomic labels to Metagenome-Assembled Genomes (MAGs), we can **interpret microbial diversity**, explore ecological roles, and compare communities across different environments.

The **[Genome Taxonomy Database (GTDB)](https://gtdb.ecogenomic.org/)** ({% cite Parks2021GTDBAO %}) is a **comprehensive, standardized resource** for bacterial and archaeal taxonomy. Unlike traditional taxonomy, which relies on phenotypic and genetic traits, GTDB uses **genome sequences** to define taxonomic relationships. It provides a **consistent and up-to-date** framework for classifying microbial genomes, ensuring that taxonomic assignments are **accurate and reproducible**.

GTDB includes
- **Genome-based taxonomy** using **120 ubiquitous single-copy marker genes** to infer phylogenetic relationships.
- **Standardized nomenclature** following a **consistent naming convention** that reflects evolutionary relationships.
- **Regular updates** to incorporate new genomic data to refine taxonomic classifications.

To classify microbial genomes using the GTDB, we will use **GTDB-Tk** (Genome Taxonomy Database Toolkit, {% cite Chaumeil2019 %}, {% cite chaumeil2022gtdb %}) is a **bioinformatics tool** designed to classify microbial genomes using the GTDB. It aligns input genomes to the GTDB reference database and assigns taxonomic labels based on **genomic similarity and phylogenetic placement**.

> <details-title>How GTDB-Tk works?</details-title>
> 
> 1. **Identify Marker Genes:** GTDB-Tk scans input genomes for the **120 single-copy marker genes** used by GTDB. These genes are conserved across bacterial and archaeal lineages, providing a robust foundation for taxonomic classification.
> 
> 2. **Align and Compare:** The identified marker genes are aligned to the **GTDB reference database**, and their sequences are compared to determine the closest taxonomic relatives.
> 
> 3. **Taxonomic Assignment:** GTDB-Tk assigns a **taxonomic classification** to each input genome, from domain to species level, based on the **phylogenetic placement** of the marker genes.
> 
> 4. **Quality Control:** The tool also provides **quality metrics**, such as the **percentage of marker genes recovered** and the **confidence of taxonomic assignments**, helping researchers assess the reliability of the classifications.
{: .details}

> <hands-on-title>Format GTDB-Tk results</hands-on-title>
>
> 1. {% tool [Extract dataset](__EXTRACT_DATASET__) %} with following parameters:
>    - {% icon param-collection %} *"Input List"*: `GTDB-Tk Classify genomes on data X, data Y, and others (summary)`
> 2. {% tool [Cut](Cut1) %} with following parameters:
>    - *"Cut columns"*: `c1,c2`
>    - {% icon param-file %} *"From"*: output of **Extract dataset**
> 3. {% tool [Replace Text](toolshed.g2.bx.psu.edu/repos/bgruening/text_processing/tp_replace_in_column/9.5+galaxy2) %} with following parameters:
>    - {% icon param-file %} *"File to process"*: output of **Cut**
>    - In *Replacement*
>      - *"in column"*: `Column:2`
>      - *"Find pattern"*: `;`
>      - *"Replace with"*: `\t`
>
{: .hands_on}

> <question-title></question-title>
>
> 1. What is the lowest taxonomic level identified in the GTDB-Tk classification
> 2. Have all bins been assigned a taxonomic label at this level?
> 3. What are the different taxons found at the different levels?
> 4. The honeybee gut microbiota is relatively simple, dominated by five core bacterial lineages corresponding to clusters within the genera *Bifidobacterium*, *Bombilactobacillus* (previously called *Lactobacillus*), *Gilliamella*, *Lactobacillus* and *Snodgrassella* ({% cite kwong2016gut %}). Other non-core bacteria are commonly present, and include *Bartonella*, *Commensalibacter* and *Frischella* ({% cite kwong2016gut %}). Are the identified genera expected in the samples? How are the 
> 5. What is the taxonomic assignment for `SRR24759616_bin_59648`? Could that explain the difference of relative abundance between the samples?
>
> > <solution-title></solution-title>
> >
> > 1. The **lowest taxonomic level identified** in the GTDB-Tk classification is the **species level**. 
> > 2. **Most bins** have been successfully assigned a taxonomic label at the species level. **One bin (`SRR24759616_bin_101784`)** lacks a species-level assignment, which may occur due to:
> >    - **Insufficient genomic information**: lowest completeness and highest contamination given **CheckM2** results
> >    - **Lack of closely related reference genomes** in the GTDB database.
> >    - **Novel or poorly characterized microbial lineages** that do not align well with existing taxonomic classifications.
> >    
> >    This highlights the importance of **evaluating the completeness and quality of MAGs** and considering **higher taxonomic ranks** (e.g., genus or family) when species-level assignments are unavailable.
> > 3. The table presents the taxonomic classification of Metagenome-Assembled Genomes (MAGs) at various taxonomic levels, from domain to species:
> >   
> >    | MAGs                          | Domain     | Phylum          | Class              | Order               | Family              | Genus               | Species                     |
> >    |-------------------------------|------------|-----------------|--------------------|---------------------|---------------------|---------------------|------------------------------|
> >    | SRR24759598_bin_100002 | Bacteria   | Bacillota_A      | Clostridia         | Lachnospirales      | *Lachnospiraceae*   | *Novisyntrophococcus* | *Novisyntrophococcus liquoris* |
> >    | SRR24759616_bin_101784 | Bacteria   | Bacillota_A      | Clostridia         | Lachnospirales      | *Lachnospiraceae*   | *Mobilisporobacter*  |                              |
> >    | SRR24759598_bin_111386 | Bacteria   | Bacillota        | Bacilli            | Lactobacillales     | *Lactobacillaceae*  | *Apilactobacillus*   | *Apilactobacillus kunkeei_C*  |
> >    | SRR24759598_bin_1966    | Bacteria   | Bacillota        | Bacilli            | Lactobacillales     | *Lactobacillaceae*  | *Apilactobacillus*   | *Apilactobacillus apinorum*   |
> >    | SRR24759598_bin_135536  | Bacteria   | Bacillota        | Bacilli            | Lactobacillales     | *Lactobacillaceae*  | *Fructobacillus*     | *Fructobacillus fructosus*    |
> >    | SRR24759616_bin_1222    | Bacteria   | Bacillota        | Bacilli            | Lactobacillales     | *Lactobacillaceae*  | *Lactobacillus*      | *Lactobacillus helsingborgensis* |
> >    | SRR24759616_bin_72251   | Bacteria   | Bacillota        | Bacilli            | Lactobacillales     | *Lactobacillaceae*  | *Lactobacillus*      | *Lactobacillus melliventris*   |
> >    | SRR24759598_bin_130125  | Bacteria   | Bacillota        | Bacilli            | Lactobacillales     | *Enterococcaceae*   | *Enterococcus*       | *Enterococcus faecalis*       |
> >    | SRR24759616_bin_100610  | Bacteria   | Actinomycetota   | Actinomycetes      | Actinomycetales    | *Bifidobacteriaceae*| *Bifidobacterium*    | *Bifidobacterium indicum*       |
> >    | SRR24759598_bin_123826  | Bacteria   | Pseudomonadota   | Gammaproteobacteria| Enterobacterales   | *Enterobacteriaceae*| *Proteus*            | *Proteus mirabilis*            |
> >    | SRR24759598_bin_131158 | Bacteria   | Pseudomonadota   | Gammaproteobacteria| Enterobacterales   | *Enterobacteriaceae*| *Serratia*           | *Serratia ureilytica*           |
> >    | SRR24759598_bin_52557   | Bacteria   | Pseudomonadota   | Gammaproteobacteria| Enterobacterales   | *Enterobacteriaceae*| *Klebsiella*         | *Klebsiella oxytoca*            |
> >    | SRR24759616_bin_59648   | Bacteria   | Pseudomonadota   | Gammaproteobacteria| Enterobacterales   | *Enterobacteriaceae*| *Frischella*         | *Frischella perrara*             |
> >    | SRR24759598_bin_91015   | Bacteria   | Pseudomonadota   | Gammaproteobacteria| Enterobacterales   | *Enterobacteriaceae*| *Gilliamella*        | *Gilliamella sp945271295*       |
> >    | SRR24759616_bin_100282  | Bacteria   | Pseudomonadota   | Gammaproteobacteria| Burkholderiales    | *Neisseriaceae*     | *Snodgrassella*      | *Snodgrassella alvi*             |
> >    | SRR24759598_bin_61470   | Bacteria   | Pseudomonadota   | Alphaproteobacteria| Rhizobiales         | *Rhizobiaceae*      | *Bartonella_A*       | *Bartonella_A choladocola*      |
> >
> >    Comments:    
> >     - **Domain** level: All bins belong to the **Bacteria** domain.
> >     - **Phylum** level: The majority of bins belong to **Bacillota** and **Pseudomonadota**.**Actinomycetota** is represented by only one bin (`SRR24759616_bin_100610`).
> >     - **Class** level: **Clostridia** and **Bacilli** are the dominant classes within the **Bacillota** phylum.**Gammaproteobacteria** is the most common class within **Pseudomonadota**.
> >     - **Order** Level: **Lactobacillales** and **Enterobacterales** are the most frequently observed orders. **Rhizobiales** and **Burkholderiales** are represented by fewer bins.
> >     - **Family** Level: ***Lactobacillaceae*** and ***Enterobacteriaceae*** are the most common families.***Neisseriaceae*** and ***Bifidobacteriaceae*** are represented by only one bin each.
> >      - **Genus** Level: ***Apilactobacillus***, ***Lactobacillus***, and ***Proteus*** are among the most frequently observed genera. ***Bartonella_A***, ***Snodgrassella***, and ***Bifidobacterium*** are represented by only one bin each.
> >      - **Species** Level:  Most bins have been assigned to the species level. **`SRR24759616_bin_1222`** has a species assignment marked with `?`, indicating uncertainty in the classification.
> >
> >    The table showcases a **diverse microbial community** spanning multiple phyla, classes, orders, families, genera, and species. The **dominant phyla** are **Bacillota** and **Pseudomonadota**, with notable representation from **Actinomycetota**. The **most common genera** include **Apilactobacillus**, **Lactobacillus**, and **Proteus**, reflecting a mix of environmentally and clinically relevant microbes. The presence of **unassigned or uncertain species** highlights the potential for novel microbial discovery within these samples.
> >
> > 4. The five core bacterial lineage dominating the honeybee gut microbiota are found as two of the three non-core bacteria that are commonly present:
> > 
> >    Observed Genus | Expected in Honeybee Gut Microbiota? | Notes 
> >    --- | --- | --
> >    *Novisyntrophococcus* | No | Not well-documented in bees; potentially novel or environmental.
> >    *Mobilisporobacter* | No | Not typically reported in bees; potentially novel or contaminant.
> >    *Apilactobacillus* | Yes | Formerly *Lactobacillus*; commonly associated with bees and bee products.
> >    *Fructobacillus* | Occasionally | Environmental bacteria.
> >    *Lactobacillus* | Yes | Core member; contributes to carbohydrate metabolism and pathogen resistance.
> >    *Enterococcus* | Occasionally | Pathogenic bacteria.
> >    *Bifidobacterium* | Yes | Core member; supports digestive health and immune function.
> >    *Proteus* | No | Opportunistic pathogen; likely contamination.
> >    *Serratia* | Occasionally | Pathogenic bacteria.
> >    *Klebsiella* | No | Clinical associate; likely contamination.
> >    *Frischella* | Yes | Commonly present; involved host immune system activation.
> >    *Gilliamella* | Yes | Core member; involved in pectin degradation and nutrient metabolism.
> >    *Snodgrassella* | Yes | Core member; essential for biofilm formation and immune modulation.
> >    *Bartonella_A* |  Yes | Commonly present.
> >    
> >    **Note**: The genus *Lactobacillus* has undergone taxonomic revisions, and some species previously classified as *Lactobacillus* are now placed in *Bombilactobacillus* and *Apilactobacillus*.
> > 
> > 5. The taxonomic assignment for **`SRR24759616_bin_59648`** is ***Frischella perrara***, a well-known member of the **honeybee gut microbiome**. According to the **CoverM results**, ***Frischella perrara*** shows a **relative abundance of 0.7% in SRR24759598** (infected with *Nosema ceranae* spores and treated with a probiotic) and **2.1% in SRR24759616** (infected with *N. ceranae* spores but **not treated** with the probiotic).
> >
> >    *Frischella perrara* is known to **activate the honeybee immune system** ({% cite motta2024honeybee %}), including:
> >    - **Humoral immunity:** Stimulates the **Toll and Imd pathways**, leading to the production of **antimicrobial peptides** that help combat pathogens.
> >    - **Cellular immunity:** Triggers the **melanization cascade**, which contributes to the **scab-like phenotype** observed in infected tissues.
> >
> >    The **higher relative abundance of *Frischella perrara* in SRR24759616** (2.1%) compared to SRR24759598 (0.7%) could be attributed to:
> >    1. **Probiotic Treatment in SRR24759598:** The probiotic may have **suppressed or outcompeted** *Frischella perrara*, leading to its lower abundance. Probiotics can **modulate the gut microbiome**, potentially reducing the dominance of certain immune-activating bacteria.
> >    2. **Immune Response in SRR24759616:** Without probiotic treatment, *Frischella perrara* may **proliferate in response to *N. ceranae* infection**, as it plays a role in **immune activation**. The **increased abundance** could reflect a **heightened immune response** to combat the pathogen.
> >    3. **Microbiome Dysbiosis:** *Nosema ceranae* infection can **disrupt the gut microbiome**, potentially allowing *Frischella perrara* to thrive in the absence of competing microbes.
> >    
> >    The difference in relative abundance of ***Frischella perrara*** between the two samples likely reflects the **impact of probiotic treatment** and the **immune response dynamics** in *Nosema*-infected bees. But we would need more sample to really draw conclusions.
> {: .solution}
>
{: .question}

Let's add the relative abundance values.

> <hands-on-title>Add abundance values at the genus level</hands-on-title>
>
> 1. {% tool [Replace Text](toolshed.g2.bx.psu.edu/repos/bgruening/text_processing/tp_replace_in_column/9.5+galaxy2) %} with following parameters:
>    - {% icon param-file %} *"File to process"*: output of **Replace Text**
>    - In *Replacement*
>      - *"in column"*: `Column:1`
>      - *"Find pattern"*: `.fasta`
>      - *"Replace with"*: `\t`
> 2. {% tool [Join two Datasets](join1) %} with following parameters:
>    - {% icon param-file %} *"Join"*: output of last **Replace Text**
>    - *"using column"*: `Column:1`
>    - {% icon param-collection %} *"with"*: `CoverM genome on data X, data Y, and others`
>    - *"and column"*: `Column:1`
>    - *"Keep lines of first input that do not join with second input"*: `Yes`
> 3. {% tool [Cut](Cut1) %} with following parameters:
>    - *"Cut columns"*: `c1-c8,c10`
>    - {% icon param-collection %} *"From"*: output of **Join two Datasets**
> 4. {% tool [Remove beginning](Remove beginning1) %} with following parameters:
>    - *"Remove first"*: `1`
>    - {% icon param-collection %} *"from"*: output of **Cut**
> 5. {% tool [Column join](toolshed.g2.bx.psu.edu/repos/iuc/collection_column_join/collection_column_join/0.0.3) %} with following parameters:
>    - {% icon param-collection %} *"Tabular files"*: output of **Remove beginning**
>    - *"Identifier column"*: `1`
>    - *"Add column name to header"*: `Yes`
> 6. {% tool [Cut](Cut1) %} with following parameters:
>    - *"Cut columns"*: `c1-c9,c17`
>    - {% icon param-file %} *"From"*: output of **Column join**
> 7. {% tool [Group](Grouping1) %} with following parameters:
>    - {% icon param-file %} *"Select data"*: output of last **Cut**
>    - *"Group by column"*: `Column:7`
>    - In *"Operation"*:
>       - In *"1: Operation"*:
>         - *"Type"*: `Count`
>         - *"On column"*: `Column: 9`
>       - In *"2: Operation"*:
>         - *"Type"*: `Count`
>         - *"On column"*: `Column: 10`
>       - In *"3: Operation"*:
>         - *"Type"*: `Sum`
>         - *"On column"*: `Column: 9`
>       - In *"4: Operation"*:
>         - *"Type"*: `Sum`
>         - *"On column"*: `Column: 10`
> 8. Create a new **tabular** dataset (`header`) from the following
>
>    ```text
>    Genus	MAGs number (SRR24759598)	MAGs number (SRR24759616)	MAGs abundance (SRR24759598)	MAGs abundance (SRR24759616)
>    ```
>
>    {% snippet faqs/galaxy/datasets_create_new_file.md name="header" format="tabular" %}
>
> 9. {% tool [Concatenate datasets](cat1) %} with following parameters:
>    - {% icon param-file %} *"Concatenate Dataset"*: the `header` dataset
>    - *"Dataset"*
>       - Click on {% icon param-repeat %} *"Insert Dataset"*
>         - {% icon param-file %} *"select"*: output of **Group**
> 10. {% tool [Sort](toolshed.g2.bx.psu.edu/repos/bgruening/text_processing/tp_sort_header_tool/9.5+galaxy2) %} with following parameters:
>    - {% icon param-file %} *"Sort Query"*: output of **Concatenate datasets**
>    - *"Number of header lines"*: `1`
>    - In *"Column selections"*:
>       - In *"1: Column selections"*:
>         - *"on column"*: `Column: 4`
>         - *"in"*: `Descending order`
> 10. Rename the output to `Genus counts and relative abundance`
{: .hands_on}

> <question-title></question-title>
> 
> ![This figure presents the composition and relative abundance of bacterial families and genera in the honeybee gut microbiome across different samples. Panel A features a multi-layered circular plot illustrating the taxonomic distribution of bacterial families (outer ring) and genera (inner ring), with color-coded segments representing distinct families such as Bartonellaceae (BAR), Neisseriaceae (NEI), Orbaceae (ORB), Lactobacillaceae (LAC), Leuconostocaceae (LEU), Bifidobacteriaceae (BIF), and unclassified taxa (Unk). Specific species within each genus, such as Snodgrassella alvi, Frischella perrara, Gilliamella apicola, and various Lactobacillus species, are also labeled. Panel B shows stacked bar charts depicting the relative abundance (%) of these bacterial taxa across multiple samples (C1-C3, P1-P3, T1-T3, TP1-TP3, N1-N3, NP1-NP3, NT1-NT3), where each color corresponds to a specific bacterial family or genus. The bar charts reveal variations in microbial community structure, highlighting the dominance of core bee gut bacteria and the presence of unclassified or less abundant taxa, providing insights into the diversity and dynamics of the honeybee gut microbiome.](./images/sbaghdi2024_figure2.png "(A) Average taxonomic composition of the gut microbiota of control bees from family to species (n = 3). Core microbiota, non-core microbiota, and unknown represent 14.1 ± 2.5%, 64.0 ± 4.8%, and 21.9 ± 2.4%, respectively. (B) Taxonomic composition from each sample at the species level, control (C1, C2, C3), Pediococcus acidilactici (P1, P2, P3), thiamethoxam (T1, T2, T3), thiamethoxam and P. acidilactici (TP1, TP2, TP3), Nosema ceranae (N1, N2, N3), N. ceranae and P. acidilactici (NP1, NP2, NP3), and N. ceranae and Thiamethoxam (NT1, NT2, NT3). Same color as in A except for Proteus spp. (P. mirabilis and P. penneri) (light brown), Providencia spp. (dark brown), and Arsenophonus spp. (black). Source: {% cite sbaghdi2024response %}")
>
> 1. Do the genera identified in {% cite sbaghdi2024response %} (Panel A of the figure) correspond to the genera found in the bins?
> 2. Do the relative abundances of bacterial genera in the bins correspond to those reported in {% cite sbaghdi2024response %} (Panel B), specifically for SRR24759598 (NP2) and SRR24759616 (N1)?
>
> > <solution-title></solution-title>
> >
> > 1. The table below compares the genera identified in {% cite sbaghdi2024response %} (Panel A of the figure) with those found in the bins:
> > 
> >    Genus | In {% cite sbaghdi2024response %}? | In bins?
> >    --- | ---
> >    *Bartonella* | Yes | Yes
> >    *Snodgrassella* | Yes | Yes
> >    *Frischella* | Yes | Yes
> >    *Gilliamella* | Yes | Yes
> >    *Lactobacillus* | Yes | Yes
> >    *Apilactobacillus* | Yes | Yes
> >    *Bombilactobacillus* | Yes | **No**
> >    *Fructobacillus* | Yes | Yes
> >    *Bifidobacterium* | Yes | Yes
> >    *Enterococcus* | **No** | Yes
> >    *Klebsiella* | **No** | Yes
> >    *Mobilisporobacter* | **No** | Yes
> >    *Novisyntrophococcus* | **No** | Yes
> >    *Proteus* | **No** (but in Panel B) | Yes
> >    *Serratia* | **No** | Yes
> > 
> >    Key Observations:
> >    - **Core bee gut genera** (*Bartonella*, *Snodgrassella*, *Frischella*, *Gilliamella*, *Lactobacillus*, *Apilactobacillus*, *Fructobacillus*, and *Bifidobacterium*) are consistent between {% cite sbaghdi2024response %} and the bins.
> >    - ***Bombilactobacillus***, a core genus in {% cite sbaghdi2024response %}, is not found in the bins.
> >    - **Additional genera** (*Enterococcus*, *Klebsiella*, *Mobilisporobacter*, *Novisyntrophococcus*, and *Serratia*) are present in the bins but not reported in {% cite sbaghdi2024response %}. *Proteus* is not reported Panel A but in Panel B in {% cite sbaghdi2024response %}.
> >
> > 2. When comparing the **relative abundances** of bacterial genera between {% cite sbaghdi2024response %} and the bins, several patterns and differences emerge:
> > 
> >    **Similarities:**
> >    - ***Snodgrassella*** and ***Bartonella*** are among the **most abundant genera** in both datasets, aligning with their known dominance in the honeybee gut microbiome.
> >    - ***Lactobacillus*** is present in both datasets, though its **relative abundance appears lower in the bins** compared to {% cite sbaghdi2024response %}.
> >    
> >    **Differences:**
> >    - ***Frischella*** is the **third most abundant genus** in **SRR24759616 (N1)** but does not hold the same rank in **SRR24759598 (NP2)**. This variation in abundance between samples is **consistent with observations in {% cite sbaghdi2024response %}**, where *Frischella* also shows **sample-specific fluctuations**.
> >    - ***Proteus***, while **not a core member** of the honeybee gut microbiome, appears **relatively abundant in the bins**. However, it is **not prominently reported** in {% cite sbaghdi2024response %}, suggesting potential **contamination or environmental exposure** in the bins.
> >    
> >    **Additional Observations:**
> >    - The **overall microbial community structure** (e.g., dominance of *Snodgrassella* and *Gilliamella*) is **qualitatively similar** between the two datasets.
> >    - **Quantitative differences** in abundance (e.g., *Lactobacillus* and *Frischella*) may reflect **sample-specific variations**, such as **treatment effects, environmental factors, or technical differences** in sequencing and analysis.
> >    
> >    *Limitations:** A **detailed comparison** would require **raw data or numerical values** from both {% cite sbaghdi2024response %} and the bins across all samples. This would allow for a **more precise assessment** of relative abundances and potential discrepancies.
> >
> >    While there is **general agreement** in the dominance of core genera (*Snodgrassella*, *Bartonella*, and *Gilliamella*) between {% cite sbaghdi2024response %} and the bins, **sample-specific variations** (e.g., *Frischella* and *Lactobacillus*) and the presence of **non-core genera** (e.g., *Proteus*) highlight the need for further investigation. Access to **raw data** would enable a more **accurate and comprehensive comparison**.
> >   
> {: .solution}
>
{: .question}

Taxonomic assignment using **GTDB-Tk** provides a **robust and standardized** approach to classifying MAGs, enabling researchers to **explore microbial diversity, compare communities, and interpret ecological roles**. By leveraging the **Genome Taxonomy Database**, GTDB-Tk ensures that taxonomic classifications are **accurate, reproducible, and up-to-date**, making it an indispensable tool in metagenomic analysis.
Accurate taxonomic classification of MAGs is essential for understanding the microbial diversity and functional potential within your metagenomic samples. By assigning taxonomic labels to your MAGs, we can explore their evolutionary relationships, ecological roles, and potential functions.

## Functional Annotation of MAGs

Functional annotation is a **critical step** in MAGs analysis, enabling us to **identify and characterize the genes** present in Metagenome-Assembled Genomes (MAGs). By assigning functional roles to these genes, we can uncover the **biological potential** of microbial communities, including their involvement in **metabolic pathways, environmental interactions, and host associations**. Functional annotation provides insights into the **ecological roles** of microbes and helps identify genes associated with **pathogenicity, antibiotic resistance, or beneficial functions** such as nutrient cycling and symbiosis.

For annotating the MAGs, several tools exists to do that: **Prokka** ({% cite seemann2014prokka %}), **Bakta** ({% cite Schwengers_2021 %}), etc. **[Bakta](https://github.com/oschwengers/bakta)** is a **fast and comprehensive** tool for the **annotation of bacterial and archaeal genomes**. It provides a **standardized and automated** pipeline for identifying **coding sequences (CDS), ribosomal RNA (rRNA), transfer RNA (tRNA), and other functional elements** within genomes. Bakta leverages **multiple databases** to assign functional annotations, including **Pfam, dbCAN, and Resfams**, ensuring a **broad and accurate** characterization of gene functions.

By default in the workflow, the **Bakta** was configured to **only check for rRNA genes**. While this provides a **quick overview** of ribosomal content, it provides limited insight into the **metabolic and functional capabilities** of the MAGs and **misses the full functional potential** of the genome **Full annotation** (including CDS, tRNA, CRISPR, and functional domains) is essential for **understanding the biological roles** of the microbes, such as their involvement in **metabolic pathways, virulence, or symbiosis**. Let's rerun **Bakta** to have all annotations.

> <hands-on-title>Relaunch Bakta</hands-on-title>
>
> 1. {% tool [Bakta](toolshed.g2.bx.psu.edu/repos/iuc/bakta/bakta/1.9.4+galaxy1) %} with following parameters:
>    - In *"Input/Output options"*:
>        - {% icon param-collection %} *"Select genome in fasta format"*: Contig file
>        - *"Bakta database"*: latest one
>        - *"AMRFinderPlus database"*: latest one
>    - In *"Workflow option to skip steps"*:
>        - *"Select steps to skip"*: Deselect all
>    - In *"Selection of the output files"*:
>        - *"Output files selection"*:
>          - `Annotation file in TSV`
>          - `Annotation and sequence in GFF3`
>          - `Feature nucleotide sequences as FASTA`
>          - `Summary as TXT`
>          - `Plot of the annotation result as SVG`
> 2. {% tool [MultiQC](toolshed.g2.bx.psu.edu/repos/iuc/multiqc/multiqc/1.27+galaxy4) %} with following parameters:
>   - *"Which tool was used generate logs?"*: `Bakta`
>   - {% icon param-files %} *"Output of Bakta"*: `Bakta on collection X: Summary (TXT)`
{: .hands_on}

> <question-title></question-title>
>
> 1. How do the number of contigs, total bases, and coding sequences (CDS) in the provided table reflect the quality and functional potential of the MAGs?
> 2. The MAGs `SRR24759598_bin_52557` and `SRR24759598_bin_131158` have a large number of bases and coding sequences (CDS), suggesting they are more complete and functionally diverse. What species do these bins correspond to, and is the number of predicted CDS consistent with expectations for these species?
> 3. The MAGs `SRR24759598_bin_1966` and `SRR24759598_bin_135536` have relatively few bases and coding sequences (CDS), which may suggest incomplete assemblies or genomes from microbes with inherently limited functional repertoires. What are the completeness percentages for these MAGs, their corresponding species, and do we expect such a low number of CDS for these species?
> 4. The Bakta annotation results for 16 metagenome-assembled genomes (MAGs) show the distribution of different feature types, including coding sequences (CDS), ribosomal RNA (rRNA), transfer RNA (tRNA), and other genomic elements. How can we interpret the relative proportions of these features, and what do they tell us about the quality and functional potential of these MAGs?
>    
>    ![This bar chart visualizes the distribution of genomic feature types across 16 metagenome-assembled genomes (MAGs) as annotated by Bakta. Each horizontal bar represents a single MAG, with the length of the bar corresponding to the total number of contigs in the assembly. The chart uses color-coded segments to indicate the relative proportions of different genomic features: coding sequences (CDS) in blue, which dominate the majority of each bar, followed by smaller segments representing ribosomal RNA (rRNA) in black, transfer RNA (tRNA) in green, transfer-messenger RNA (tmRNA) in orange, non-coding RNA (ncRNA) in purple, ncRNA regions in pink, hypothetical proteins in yellow, CRISPR arrays in teal, pseudogenes in red, small open reading frames (sORF) in light blue, and origin of replication (oriC) in light green. The chart highlights the functional diversity and completeness of the MAGs, with CDS being the most abundant feature, indicating a rich potential for metabolic and regulatory functions, while the presence of essential elements like rRNA, tRNA, and oriC suggests high-quality assemblies. The varying proportions of hypothetical proteins and CRISPR arrays also point to opportunities for novel gene discovery and adaptive immunity studies.](./images/bakta_feature_types.png)
>
> > <solution-title></solution-title>
> >
> > 1. Here's how these metrics can be interpreted:
> >    1. **Number of Contigs**:
> >      The **number of contigs** reflects the **fragmentation level** of the assembled genome. **Fewer contigs** (e.g., `SRR24759616_bin_101784_fasta` with 1,462 contigs) may indicate a **more fragmented assembly**, which can complicate downstream analyses such as functional annotation and comparative genomics. **More contigs** can also suggest **higher diversity or complexity** in the genome, but excessive fragmentation may indicate **poor assembly quality**.
> >    2. **Total Bases**
> >       The **total number of bases** represents the **estimated genome size** of each MAG. Larger genomes (e.g., `SRR24759598_bin_100002_fasta` with 3,036,552 bases) may encode **more genes and functional diversity**, while smaller genomes (e.g., `SRR24759598_bin_1966_fasta` with 1,243,253 bases) might represent **simpler or more specialized microbes**. The total bases can also help assess **completeness** when compared to expected genome sizes for known microbial taxa.
> >    3. **Number of Coding Sequences (CDS)**
> >       The **number of CDS** indicates the **functional potential** of the MAG. A **higher number of CDS** (e.g., `SRR24759598_bin_100002_fasta` with 2,645 CDS) suggests a **greater metabolic and functional capacity**, potentially encoding a wider range of proteins and pathways. A **lower number of CDS** (e.g., `SRR24759598_bin_1966_fasta` with 1,175 CDS) may indicate a **more specialized or streamlined genome**, or it could reflect **incomplete assembly or annotation**.
> >
> >    The **number of contigs, total bases, and CDS** provide valuable insights into the **quality and functional potential** of MAGs. While some MAGs appear **more complete and functionally rich**, others may require **additional sequencing or assembly improvements** to fully capture their biological roles. Understanding these metrics is essential for **prioritizing MAGs for downstream analyses** and ensuring robust interpretations of microbial functions.
> > 2. The MAGs in question correspond to the following species:
> >    - `SRR24759598_bin_52557` is classified as *Klebsiella oxytoca*.
> >    - `SRR24759598_bin_131158` is classified as *Serratia ureilytica*.
> > 
> >    Both bins exhibit **high completeness**:
> >    - `SRR24759598_bin_52557` (*Klebsiella oxytoca*): **100% completeness**
> >    - `SRR24759598_bin_131158` (*Serratia ureilytica*): **96.5% completeness**
> > 
> >    To assess whether the number of predicted CDS aligns with expectations, we can compare these MAGs to **reference genomes** available on NCBI:
> >    - *Klebsiella oxytoca* (`SRR24759598_bin_52557`, [NCBI: GCF_900636985.1](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_900636985.1/))
> >      - **Reference Genome Size:** ~5.9 Mb 
> >      - **MAG Genome Size:** 5,533,143 bases
> >      - **Expected Protein-Coding Genes:**
> >        - **RefSeq:** 5,287
> >        - **GenBank:** 5,276
> >      - **Predicted CDS (Bakta):** 5,085
> >    
> >    - *Serratia ureilytica* (`SRR24759598_bin_131158`, [NCBI: GCF_017309605.1](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_017309605.1/))
> >      - **Reference Genome Size:** ~5.1 Mb ()
> >      - **MAG Genome Size:** 4,873,360 bases
> >      - **Predicted CDS (Bakta):** 5,085- **Expected Protein-Coding Genes (NCBI):**
> >        - **RefSeq:** 4,683
> >        - **GenBank:** 4,674
> >      - **Predicted CDS (Bakta):** 4,564
> >      
> >    The number of CDS predicted for both ***Klebsiella oxytoca*** and ***Serratia ureilytica*** is **consistent with expectations** based on reference genomes. The slight differences can be attributed to **strain-level genetic diversity, assembly quality, or annotation methods**. Overall, these MAGs appear to be **well-assembled and functionally representative** of their respective species.
> >
> > 3. As for the above questions, let's look at the classification, completeness, predicted CDS and expectations from NCBI:
> >
> >    Sample | `SRR24759598_bin_1966` | `SRR24759598_bin_135536`
> >    --- | ---
> >    Classification | *Apilactobacillus apinorum* | *Fructobacillus fructosus*
> >    Completness | 81.3% | 91.9%
> >    MAG Genome Size | 1,243,253 | 1,332,365
> >    Reference Genome Size | 1.5 Mb | 1.4 Mb
> >    Predicted CDS | 1,175 | 1,296
> >    Expected Protein-Coding Genes | 1,341 (RefSeq); 1,343 (GenBank) | 1,369 (RefSeq); 1,381 (GenBank)
> >    
> >   **Interpretation:**
> >   1. **Completeness**: `SRR24759598_bin_1966` (*Apilactobacillus apinorum*) has a **completeness of 81.3%**, indicating it is **moderately complete** but may still lack some genomic regions `SRR24759598_bin_135536` (*Fructobacillus fructosus*) has a **completeness of 91.9%**, suggesting it is **nearly complete** and likely captures most of the genome.
> >   2. **Genome Size and CDS:** Both *Apilactobacillus apinorum* and *Fructobacillus fructosus* are **small-genome bacteria**, typically ranging from **1.3–1.5 Mb**. Their **smaller genome sizes** naturally result in **fewer CDS** compared to larger, more complex genomes. The **predicted CDS** for both MAGs (1,175 and 1,296, respectively) are **slightly lower than the NCBI reference values** (1,341–1,381). This discrepancy is likely due to:
> >   - **Incomplete assembly** (especially for `SRR24759598_bin_1966` at 81.3% completeness).
> >   - **Strain-specific variations** in gene content.
> >   - **Annotation differences** between Bakta and NCBI pipelines.
> >   3. **Expected CDS for These Species:** Both *Apilactobacillus apinorum* and *Fructobacillus fructosus* are **lactic acid bacteria (LAB)** with **streamlined genomes**, meaning they naturally have **fewer genes** compared to more metabolically versatile bacteria. The **predicted CDS values are reasonable** given their **genome sizes and completeness levels**, and they align closely with **NCBI reference expectations**.
> >
> >   The **low number of bases and CDS** in these MAGs is **consistent with the expected genome sizes and functional repertoires** of *Apilactobacillus apinorum* and *Fructobacillus fructosus*. While `SRR24759598_bin_1966` may benefit from further assembly refinement, both MAGs reflect the **typical genomic characteristics** of these small-genome, lactic acid bacteria.
> >
> > 4. The bar chart from Bakta annotation provides a **visual representation** of the **relative proportions of different genomic features** across 16 MAGs. Here's how we can interpret these results and what they reveal about the **quality and functional potential** of the MAGs:
> >   1. **Coding Sequences (CDS)**: The **blue segments**, representing coding sequences (CDS), dominate the bar chart for all MAGs. This is **expected**, as CDS make up the majority of functional genes in bacterial and archaeal genomes. A **high proportion of CDS** indicates a **rich functional repertoire**, suggesting that these MAGs encode a diverse set of proteins involved in **metabolism, environmental interactions, and host associations**.
> >   2. **Ribosomal RNA (rRNA)**: The **black segments** represent ribosomal RNA (rRNA) genes, which are essential for **protein synthesis** and are highly conserved across bacteria and archaea. The presence of rRNA genes is a **good indicator of genome completeness**, as these genes are typically present in **single or few copies** per genome. Their detection suggests that the MAGs are **sufficiently complete** to capture essential genomic elements.
> >   3. **Transfer RNA (tRNA)**: The **green segments** represent transfer RNA (tRNA) genes, which are crucial for **translation** and protein synthesis. Similar to rRNA, the presence of tRNA genes indicates that the MAGs are **functionally complete** and capable of synthesizing proteins.
> >   4. **Transfer-Messenger RNA (tmRNA)**: The **orange segments** represent transfer-messenger RNA (tmRNA), which plays a role in **rescuing stalled ribosomes** and tagging incomplete proteins for degradation. The presence of tmRNA suggests that these MAGs have **mechanisms for maintaining protein synthesis fidelity**, which is important for cellular robustness.
> >   5. **Non-Coding RNA (ncRNA)**: The **purple segments** represent non-coding RNA (ncRNA), which includes a variety of RNA molecules involved in **regulation, catalysis, and other cellular processes**. The presence of ncRNA indicates that these MAGs may have **regulatory elements** that contribute to complex cellular functions.
> >   6. **Hypothetical Proteins**: The **yellow segments** represent hypothetical proteins, which are predicted genes with **unknown functions**. A significant proportion of hypothetical proteins suggests there is **potential for discovering novel functions** and expanding our understanding of microbial biology.
> >   7. **CRISPR Arrays**: The **teal segments** represent CRISPR arrays, which are involved in **adaptive immunity** against foreign genetic elements like viruses and plasmids. The presence of CRISPR arrays indicates that these MAGs have **mechanisms for defending against mobile genetic elements**, which can be important for survival in dynamic environments.
> >   8. **Pseudogenes**: The **red segments** represent pseudogenes, which are **non-functional remnants of genes** that have lost their protein-coding ability.The presence of pseudogenes can provide insights into **genome evolution and adaptation**, reflecting past genetic events.
> >   9. **Small Open Reading Frames (sORF)**: The **light blue segments** represent small open reading frames (sORFs), which may encode **small peptides with regulatory or functional roles**.sORFs can contribute to the **functional diversity** of the genome, potentially encoding peptides involved in **signaling, regulation, or other specialized functions**.
> >   10. **Origin of Replication (oriC)**: The **light green segments** represent the origin of replication (oriC), which is essential for **DNA replication initiation**. The presence of oriC is an indicator of **genome integrity** and completeness, as it is a critical element for genome replication.
> >
> >    **Overall Interpretation:**
> >    - **Quality of MAGs:** The presence of **rRNA, tRNA, tmRNA, and oriC** suggests that these MAGs are **relatively complete and of high quality**, capturing essential genomic elements.
> >    - **Functional Potential:** The dominance of **CDS and the presence of regulatory elements (ncRNA, CRISPR, sORF)** indicate a **rich functional potential**, with capabilities for diverse metabolic and regulatory functions.
> >    - **Novelty and Exploration:** The presence of **hypothetical proteins and pseudogenes** highlights opportunities for **further exploration and discovery** of novel genetic functions and evolutionary insights.
> {: .solution}
>
{: .question}

Bakta annotations can be used to:
- **Identify Metabolic Pathways:** Determine which **biochemical pathways** are present in the MAGs, such as those involved in **carbon metabolism, nitrogen fixation, or secondary metabolite production**.
- **Detect Antibiotic Resistance Genes:** Use annotations from the **CARD database** to identify genes associated with **antibiotic resistance**, which is critical for understanding **potential threats to bee health** or **horizontal gene transfer** in microbial communities.
- **Explore Virulence Factors:** Investigate genes linked to **virulence or symbiosis**, providing insights into how microbes interact with their **host or environment**.
- **Compare Functional Potential:** Compare the functional profiles of **different MAGs** to identify **unique or conserved genes** that may explain ecological niches or functional specialization.

Functional annotation using **Bakta** is a **powerful approach** to uncover the **biological roles and functional potential** of MAGs. By rerunning Bakta with **complete annotation**, researchers can gain a **comprehensive understanding** of the metabolic, ecological, and pathological capabilities of microbial communities. This knowledge is **essential for comparative genomics, biotechnology applications, and understanding microbial contributions to ecosystem health**.

# Conclusion

In this tutorial, we have navigated the comprehensive workflow of Metagenomics Assembled Genomes (MAGs) building, transforming raw sequencing reads into biologically meaningful insights about microbial communities. Each step in the process is essential for ensuring the accuracy, completeness, and relevance of your results and contributes to a robust and comprehensive understanding of the diversity, abundance, and functional potential of the microbes present.
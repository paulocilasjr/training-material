---
layout: tutorial_hands_on

title: "Building Metagenomics Assembly Genomes (MAGs) from paired short metagenomics reads"
zenodo_link: ""
answer_histories:
  - label: "UseGalaxy.eu"
    history: https://usegalaxy.eu/u/berenice/h/bee-microbiome-for-mags-building-training-2025-11-14
    date: 2025-11-26
level: Intermediate
questions:
  - 
objectives:
  - 
time_estimation: "6H"
key_points:
  - 
edam_ontology:
  - topic_3174 # Metagenomics
  - topic_0196 # Sequence assembly
contributions:
  authorship:
  - bebatut
  funding:
  - elixir
  - ifb
subtopic: metagenomics
tags:
  - assembly
  - metagenomics
  - microgalaxy
---

Metagenomics has revolutionized our understanding of microbial communities by enabling the study of genetic material directly from environmental samples. One of the most powerful applications of metagenomics is the reconstruction of **Metagenome-Assembled Genomes (MAGs)**, i.e. near-complete or complete genomes of individual microorganisms recovered from complex microbial communities. MAGs provide invaluable insights into microbial diversity, function, and ecology, without the need for laboratory cultivation.

This tutorial is designed for anyone interested in reconstructing MAGs from paired short metagenomic reads. You will learn the essential steps, from quality control and read preprocessing to assembly, binning, refinement, and annotations of MAGs. By the end of this tutorial, you will be equipped with the knowledge, tools, and workflows to confidently generate high-quality MAGs from your own metagenomic datasets.

To guide through this process, we will use **Galaxy workflows** developed within the [FAIRyMAGs project](https://elixir-europe.org/how-we-work/scientific-programme/commissioned-services/science/bfsp/fairymags), maintained by the [Intergalactic Workflow Commission](). These workflows are available via the [IWC Workflow Library](https://iwc.galaxyproject.org/), as well as two workflow registries: [WorkflowHub](https://workflowhub.eu/) and [Dockstore](https://dockstore.org/).

### Case Study: Honey Bee Gut Microbiome

To illustrate the workflows, we will use public data from a study investigating the response of the **honey bee gut microbiota** to *Nosema ceranae* (a highly prevalent microsporidian parasite) under the influence of a probiotic and/or a neonicotinoid insecticides ({% cite sbaghdi2024response %}). The honey bee gut microbiome comprises a relatively simple core community dominated by **five bacterial lineages**, including *Snodgrassella alvi* and *Gilliamella apicola*, which play essential roles in digestion, immunity, and pathogen defense ({% cite motta2024honeybee %}). However, this microbiome is highly sensitive to abiotic stressors, such as pesticides, pollutants, and climate change ({% cite motta2024honeybee %}, {% cite ramsey2019varroa %}, {% cite alberoni2021neonicotinoids %}). For example, exposure to neonicotinoids has been linked to shifts in microbial diversity and increased susceptibility to opportunistic pathogens {% cite alberoni2021neonicotinoids %}.

While research often examines stressors in isolation, their synergistic effects remain understudied ({% cite paris2020honeybee %}, {% cite meixner2010historical %}). In the {% cite sbaghdi2024response %} study, western honey bees were experimentally infected with *N. ceranae* spores, exposed to the neonicotinoid thiamethoxam, and/or treated with the probiotic bacterium *Pediococcus acidilactici*, which is thought to enhance tolerance to *N. ceranae*. The study used deep shotgun metagenomic Illumina sequencing to analyze 21 samples, with taxonomic composition explored using MetaPhlAn4 ({% cite blanco2023extending %}).

> <comment-title></comment-title>
> Learn more about taxonomic profiling in our dedicated tutorial: [**Taxonomic Profiling and Visualization of Metagenomic Data**]({% link topics/microbiome/tutorials/taxonomic-profiling/tutorial.md %})
{: .comment}

In this tutorial, we will focus on **two representative samples** from this dataset to demonstrate the MAG reconstruction process.

Sample Alias | Treatment | Replicate | BioSample | Run
--- | --- | --- | --- | ---
IP2 | `NP`: infected with *N. ceranae* spores and treate with the probiotic | 2| [SAMN35523843](https://www.ebi.ac.uk/ena/browser/view/SAMN35523843) | [SRR24759598](https://www.ebi.ac.uk/ena/browser/view/SRR24759598)
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

We will use a [versatile quality control workflow]() to process our FASTQ files. This workflow is designed to:
1. **Trim and filter reads** using **fastp** ({% cite Chen_2018 %}), removing low-quality bases (default: minimum quality score of 15) and short reads (default: minimum length of 15 bases). These parameters can be adjusted based on your specific dataset and requirements.
2. Aggregate quality reports for all samples into a single, comprehensive overview using **MultiQC** ({% cite ewels2016multiqc %}), allowing to quickly visualize and compare the quality metrics across the datasets.

> <comment-title></comment-title>
> Learn more about sequencing data quality control in our dedicated tutorial: [**Quality Control**]({% link topics/sequence-analysis/tutorials/quality-control/tutorial.md %})
{: .comment}

Let's import the workflow from WorkflowHub:

{% snippet faqs/galaxy/workflows_run_wfh.md title="mags-building/main" wfhub_id="1352" version="3"%}

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

We will now focus on filtering out both host (bee) and potential human contamination from our metagenomic reads. This critical step ensures that our dataset is enriched for microbial sequences, improving the reliability of our MAGs and enabling more precise biological insights. We will use a dedicated [workflow]() to:
1. **Map reads against reference genomes** for the host (bee) and common contaminants using  **Bowtie2** ({% cite Langmead_2009 %}, {% cite Langmead_2012 %}), enabling precise identification and removal of non-microbial sequences.
2. Aggregate and visualize mapping results with **MultiQC** ({% cite ewels2016multiqc %}), providing a comprehensive overview of the filtering process and ensuring transparency in our data cleaning efforts.

Let's import the workflow from WorkflowHub:

{% snippet faqs/galaxy/workflows_run_wfh.md title="mags-building/main" wfhub_id="1352" version="3"%}

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
5. **Quality Assessment**: Evaluate the integrity of the MAGs with **QUAST** ({% cite Gurevich_2013 %}) for assembly statistics and **CheckM2** ({% cite chklovski2023checkm2 %}) for completeness and contamination estimates.
6. **Abundance Estimation**: Quantify the relative abundance of each MAG in our samples using **CoverM** ({% cite aroney2025coverm %}), providing insights into microbial population dynamics.
7. **Taxonomic Assignment**: Classify the MAGs using **GTDB-Tk** ({% cite Chaumeil2019 %}), aligning them with the Genome Taxonomy Database for accurate taxonomic placement.
8. **Functional Annotation**: Identify and annotate genes within our MAGs using **Bakta** ({% cite  %}), revealing their functional potential and biological roles.

All results are consolidated into a single **MultiQC** ({% cite ewels2016multiqc %}) report, allowing us to easily visualize and interpret the quality and characteristics of our MAGs.

Let's import the workflow from WorkflowHub:

{% snippet faqs/galaxy/workflows_run_wfh.md title="mags-building/main" wfhub_id="1352" version="3"%}

> <hands-on-title>Build, refine, and annotate MAGs</hands-on-title>
>
> 1. Run the workflow using the following parameters
>    - *"Choose Assembler"*: `MEGAHIT`
>
>      > <comment-title></comment-title>
>      > metaSPAdes is an alternative assembler.
>      >
>      > MEGAHIT is less computationally intensive and generate higher quality single and shorter contigs but shorter. metaSPAdes is very computationally intensive, but generates longer/more complete assemblies.
>      {: .comment}
>
>    - {% icon param-collection %} *"Short-reads"*: `Trimmed reads` collection
>    - *"Minimum length of contigs to output"*: `200`
>    - *"Read length (CONCOCT)"*: `150`
>
>      > <comment-title></comment-title>
>      > CONCOCT requires the read length for coverage. Best use fastQC to estimate the mean value
>      {: .comment}
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
>    - *"Trimmed reads from grouped samples"*: `Reads without bee and human reads`
>    - *"Trimmed reads"*: `Reads without bee and human reads`
>    - *"Contamination weight (Binette)"*: `2`
>
>      > <comment-title></comment-title>
>      > This weight is used for the scoring the bins. A low weight favor complete bins over low contaminated bins.
>      {: .comment}
>
>    - *"CheckM2 Database"*: most recent one
>
>      > <comment-title></comment-title>
>      > This database is used for quality assessment for Binette, dRep, and quality assessment of the final bins.
>      {: .comment}
>
>    - *"Minimum MAG completeness percentage"*: `75`
>
>      > <comment-title></comment-title>
>      > Minimum MAG completeness percentage for bin refinement (binette) and dereplication (drep)
>      {: .comment}
>
>    - *"Maximum MAG contamination percentage"*: `25`
>
>      > <comment-title></comment-title>
>      > Maximum MAG contamination percentage for dereplication.
>      {: .comment}
>
>    - *"Minimum MAG length"*: `50000`
>
>      > <comment-title></comment-title>
>      > Minimum MAG length for dereplication
>      {: .comment}
>
>    - *"ANI threshold for dereplication"*: `0.95`
>
>      > <comment-title></comment-title>
>      > ANI threshold to form secondary clusters of dereplication. An ANI value of ≥95-96% indicates genomes belong to the same species, an ANI of ≥98-99% suggests they are the same strain, and an ANI of ≤90-95% typically points to genomes from different genera.
>      {: .comment}
>
>    - *"GTDB-tk Database"*: most recent one
>    - *"Run GTDB-Tk on MAGs"*: `Yes`
>    - *"Bakta Database"*: most recent one
>    - *"AMRFinderPlus Database for Bakta"*: most recent one
>    - *"Run Bakta on MAGs"*: `Yes`
{: .hands_on}

To ensure clarity and efficiency, we will **walk through each step of the workflow** to carefully inspect the generated results. Given that executing the entire workflow can be time-consuming, we will also **provide the option to import an history with pre-computed results for each step**. This approach allows to focus on understanding the outputs and interpreting the findings without waiting for lengthy computations.

> <details-title>Import history with pre-computed results</details-title>
>
> > <hands-on-title>Import history with pre-computed results</hands-on-title>
> >
> > 1. Import the history in one of the following URLs:
> >    - [{{ page.answer_histories[0].label }}]({{ page.answer_histories[0].history }})
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
> Learn more about metagenomic binning and available tools in our dedicated tutorial: [**Binning of metagenomic sequencing data**]({% link topics/microbiome/tutorials/metagenomics-binnin/tutorial.md %})
{: .comment}

A widely adopted strategy is to **use multiple binning tools** to maximize accuracy, followed by **bin refinement** to consolidate and improve the results. This approach leverages the strengths of different algorithms, producing a more robust and reliable set of bins.

In this analysis, we employed **four complementary binning tools**, each offering distinct advantages for capturing microbial diversity:
- **MetaBAT2** ({% cite Kang_2019 %}): Known for its efficiency and accuracy in binning metagenomic contigs.
- **MaxBin2** ({% cite Wu_2015 %}): Utilizes probabilistic models to improve binning accuracy.
- **SemiBin** ({% cite Pan_2022 %}): Incorporates deep learning to enhance binning performance, especially for complex datasets.
- **CONCOCT** ({% cite Alneberg_2014 %}): Uses Gaussian mixture models to cluster contigs based on coverage and composition.

   CONCOCT involves several steps:

   <figure>
   <pre class="mermaid">
   flowchart LR
       input1[Contigs]
       input2[Input reads mapped on contigs]
       input3[Read length]
       1[CONCOCT: Cut up contigs]
       2[CONCOCT: Generate the input coverage table]
       3[CONCOCT]
       4[CONCOCT: Merge cut clusters]
       5[CONCOCT: Extract a fasta file]
       input1 --> 1
       1 --> 2
       input2 --> 2
       1 --> 3
       2 --> 3
       input3 --> 3
       3 --> 4
       4 --> 5
   </pre>
   </figure>

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
> > 3. For both samples, CONCOCT and MaxBin2 produce more bins than MetaBAT2 and SemiBin, but the distribution of contigs per bin is more variable for CONCOCT. SRR24759598 generally has slightly more bins across all tools compared to SRR24759616, which may reflect differences in microbial diversity or sequencing depth between the samples.
MetaBAT2 and SemiBin show similar trends in both samples, with relatively consistent bin sizes and fewer extreme outliers.
> >
> > 4. CONCOCT may be useful for capturing fine-scale diversity but may require additional refinement to merge small or fragmented bins. MetaBAT2 and SemiBin are likely to produce more reliable and consolidated bins, making them suitable for downstream analyses like genome reconstruction and functional annotation. MaxBin2 offers a balance between the number of bins and the number of contigs per bin but may require further validation due to its broader distribution.
> >
> {: .solution}
>
{: .question}

We can also evaluate the binning with **CheckM** ({% cite Parks2015 %}). CheckM is a software tool used in metagenomics binning to assess the completeness and contamination of genome bins. It compares the genome bins to a set of universal single-copy marker genes that are present in nearly all bacterial and archaeal genomes. By identifying the presence or absence of these marker genes in the bins, CheckM can estimate the completeness of each genome bin (i.e., the percentage of the total set of universal single-copy marker genes that are present in the bin) and the degree of contamination (i.e., the percentage of marker genes that are found in more than one bin).

> <hands-on-title>Assessing the completeness and contamination of bins for each binner</hands-on-title>
>
> 1. Import the **Bin quality check** workflow
>
> 2. Run the workflow with parameters:
>     - {% icon param-collection %} *"MetaBAT2 Bins"*: `MetaBAT2 on collection X and collection Y: Bin sequences`
>     - {% icon param-collection %} *"MaxBin2 Bins"*: `MaxBin2 on collection X and collection Y: Bins`
>     - {% icon param-collection %} *"SemiBin Bins"*: `SemiBin on collection X and collection Y: Reconstructed bins`
>     - {% icon param-collection %} *"CONCOCT Bins"*: `CONCOCT: Extract a fasta file on collection X and collection Y : Bins`
> 3. Inspect the `MultiQC for CheckM on bins right after binners` file
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
> >    - A significant proportion of high-completeness bins from CONCOCT (50% for SRR24759598 and 62.5% for SRR24759616) exhibit contamination levels above 50%. This suggests that while CONCOCT effectively captures near-complete genomes, it may also include substantial contamination from other organisms, potentially compromising the accuracy of downstream analyses.
> >    - MetaBAT2, MaxBin2, and SemiBin produce fewer contaminated high-completeness bins, indicating that their bins are generally more reliable and purer for further genomic studies.
> >    - These results highlight the importance of post-binning refinement to reduce contamination and improve the quality of high-completeness bins, especially when using tools like CONCOCT.
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
> > 4. The minimal completeness value observed in the refined bins is 76.92% for SRR24759598 and 75.19% for SRR24759616. These values align with the 75% completeness threshold set in the refinement tool, ensuring that only high-quality bins are retained.
> > 5. 4 bins for SRR24759598 and 2 bins for SRR24759616 exhibit a completeness of greater than 99%. This indicates that these bins are near-complete representations of their respective microbial genomes, suggesting high-quality genome reconstructions suitable for in-depth functional and taxonomic analyses.
> > 6. The maximum contamination values are 47.01% for SRR24759598 and 77.2% for SRR24759616. These values indicate that some bins still contain a significant proportion of sequences from multiple organisms, which can affect the accuracy of downstream analyses. High contamination levels suggest that further refinement or manual curation may be necessary to improve the purity of these bins and ensure reliable genomic interpretations.
> >
> {: .solution}
>
{: .question}


## Dereplication

Consolidate the MAGs across all input samples using **dRep** ({% cite olm2017drep %}), which clusters genomes based on CheckM2 quality metrics to eliminate redundancy.

## Quality Assessment

Evaluate the integrity of the MAGs with **QUAST** ({% cite Gurevich_2013 %}) for assembly statistics and **CheckM2** ({% cite chklovski2023checkm2 %}) for completeness and contamination estimates.

## Abundance Estimation

Quantify the relative abundance of each MAG in our samples using **CoverM** ({% cite aroney2025coverm %}), providing insights into microbial population dynamics.

## Taxonomic Assignment

Classify the MAGs using **GTDB-Tk** ({% cite Chaumeil2019 %}), aligning them with the Genome Taxonomy Database for accurate taxonomic placement.

## Functional Annotation

Identify and annotate genes within our MAGs using **Bakta** ({% cite  %}), revealing their functional potential and biological roles.
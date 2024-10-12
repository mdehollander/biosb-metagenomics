# Functional annotation - Viromics

We will now identify viral sequences from metagenomic data. You will be provided with a set of contigs assembled from infant fecal metagenomic samples (``/data/precomputed/virome/contigs.fa``). Our goals are to:

1. Determine the **viral origin** of the contigs (and assign their taxonomy).
2. Estimate the **completeness** of viral genomes.
3. Predict the potential **bacterial host**.
4. Functionally **annotate** each identified virus.


By the end of this exercise, our objective will be to identify the bacteriophage that satisfies all of the following criteria:  
> - Belongs to the ***Caudoviricetes*** class  
> - Is predicted to have a **complete genome**  
> - Is predicted to infect the ***Clostridium*** genus  
> - Has the ability to **integrate** into the bacterial genome  
  

**Note:** For this practical, several tools are required for the viral identification and annotation pipeline:  
> - **geNomad**  
> - **CheckV**  
> - **iPHOP**  


All these tools and the necessary databases have been preinstalled and are ready for you to use.


## Step 1 - Initial virus discovery

First, we will run **geNomad** to identify viral sequences in our dataset. We will utilize the ``--enable-score-calibration`` option to compute false discovery rates (FDR). For this practical exercise, no specific FDR thresholds will be applied. Further, we will add the ``--cleanup`` flag to remove intermediate files and the ``--disable-find-proviruses`` option to avoid geNomad performing an initial prunning to remove potential contaminant host regions from proviral sequences (we will do this in the next step with CheckV). 

**Note:** The following command takes about 2 minutes. You can also continue with the results in ``/data/precomputed/virome/genomad_results/``.
```bash
genomad end-to-end \
    --enable-score-calibration \
    --disable-find-proviruses \
    contigs.fa \
    geNomad_results \
    --cleanup \
    /data/databases/genomad/genomad_db/
```

You can find a description of the geNomad output files [here](https://portal.nersc.gov/genomad/quickstart.html#understanding-the-outputs).

Look into ``geNomad_results/contigs_summary/contigs_virus_summary.tsv``. 

??? done "1. How many of the 100 initial contigs are identified as viral?"
    55 contigs.
    
??? done "2. What is the length of the largest viral genome that we have identified?"
    297,029 bases (MGV_98132).

??? done "3. Regarding the viral taxonomy, which is the most common viral class among identified viruses?"
    *Caudoviricetes*.

??? done "4. Do we identify any single-stranded DNA virus (ssDNA)?"
    Yes, there are 13 ssDNA (belonging to *Monodnaviria* realm).     

**Note:** A useful resource where you can find all the information about the viral taxonomy is [here](https://ictv.global/taxonomy).  

## Step 2 - Host contamination removal and quality check

While geNomad performs well at identifying proviruses from metagenomic data, **CheckV** is specifically designed to identify host-virus boundaries with high precision. Further, CheckV allows to estimate the completeness of the predicted viral genomes based on their comparison to a database of complete viral genomes. Therefore, here we use CheckV to quality control the geNomad results and also to trim potential host regions left at the ends of proviruses. 

**Note:** This command will take about 10 minutes to complete. Again, you can continue with the results in ``/data/precomputed/virome/checkv_results/``.
```bash
checkv \
    end_to_end \
    geNomad_results/contigs_summary/contigs_virus.fna \
    CheckV_results \
    -d /data/databases/checkv-db-v1.4
```

The CheckV output is described [here](https://bitbucket.org/berkeleylab/checkv/src/master/#markdown-header-output-files). Look into ``CheckV_results/quality_summary.tsv``.

??? done "1. How many proviruses do you find and how many viruses?"
    9 proviruses, 46 viruses.

??? done "2. How many low, medium, and high quality viruses do we detect? How many complete viruses? How many of them have direct terminal repeats?"
    11 viruses with low-quality, 20 with medium-quality and 9 with high-quality.
    15 viruses are complete. From them, 10 have direct terminal repeats (DTRs).

**Note:** Depending on the project's goals, it may be advisable to select only viruses predicted to be of at least medium quality. However, for this exercise, we will proceed with all available sequences.

**Note:** CheckV provides 2 separate files with the identified proviral and viral sequences. As we want to work with both of them, we combine them into a single file: 

    cat CheckV_results/proviruses.fna CheckV_results/viruses.fna > CheckV_results/combined.fna


## Step 3 - Bacterial host assignment

We will use **iPHoP** for bacterial host assignment of the viruses. Although iPHoP provides both genus- and species-level host predictions, we will focus solely on genus-level assignments. This is because iPHoP offers high-confidence predictions at the genus level (with an estimated false discovery rate of less than 10%), while the confidence decreases at the species level.

**Note:** The following command can take around 1 hour. Therefore, you should continue with the results in ``/data/precomputed/virome/iphop_results/``.
```bash
mkdir iphop_results

iphop predict \
    --fa_file CheckV_results/combined.fna \
    --db_dir /data/databases/Aug_2023_pub_rw/ \
    --num_threads 8 \
    --out_dir iphop_results
```

The iPHoP output is described [here](https://bitbucket.org/MAVERICLab/vcontact2/wiki/Home](https://bitbucket.org/srouxjgi/iphop/src/main/#markdown-header-main-output-files)). 

**Note:** iPHoP allows to enrich the default database with custom MAGs to improve the host assignment of the viruses.

Look into ``iphop_results/Host_prediction_to_genus_m90.csv``. By default, all virus-host pairs for which the confidence score is higher than the selected cutoff (default = 90) are included. For this exercise, consider only the top hit for each virus (prediction with highest confidence score):

??? done "1. Is there any virus predicted to infect the **_Clostridium_** genus?"
    Yes, 2 viruses are predicted to infect **_Clostridium_** (CS_426 and CS_2284).

??? done "2. Considering the genome completeness estimated in the previous step, which of these **_Clostridium_** phages is predicted to have a complete genome?"
    Both phages have a complete genome (with DTRs).
    

## Step 4 - Functional annotation

Multiple tools can be used for the functional annotation of viral genomes. Here, we will use **geNomad** (annotate module) to retrieve the annotations for each of the proteins in our predicted viral genomes.

**Note:** The following command takes about 2 minutes. You can also continue with the results in ``/data/precomputed/virome/genomad_annotation_results/``.
```bash
genomad annotate \
    CheckV_results/combined.fna \
    geNomad_annotation_results \
    --cleanup \
    /data/databases/genomad/genomad_db/
```

The detailed explanation of this step can be found [here](https://portal.nersc.gov/genomad/pipeline.html#annotate). Check now the ``geNomad_annotation_results/combined_annotate/combined_genes.tsv`` file:

??? done "1. Does any of the identified viruses use an alternative genetic code?"
    No, all the viruses use the standard genetic code (translation table 11).

??? done "2. Are any of the identified viruses encoding proteins that enable integration into the bacterial genome (integrases)?"
    Yes, at least 6 viruses encode integrases (GPD_79092, ELGV_14024, CS_100, CS_659, CS_1726 and CS_2284)  

**Note:** Genetic code 11 (translation table 11) is the standard code used for Bacteria, Archaea, prokaryotic viruses and chloroplast proteins.

??? done "Which bacteriophage are we looking for?"
    **CS_2284**: A predicted complete *Clostridium* phage, belonging to the *Caudoviricetes* class, that can integrate into the host genome.








!!! important "Learning objectives"
    - Explore binning results from biofilm metagenomes in a greenhouse gas-emitting cave.
    - Seek evidence identifying the primary consumers (CO2 & CH4) within this extreme environment.

![Cave Microbiome Sampling](../assets/cave1-1.png)
(a) General overview of the cave. (b) Detailed images of the cave biofilms. The dashed line in panel a marks the stable gaseous chemocline between the volcanic gases (below the chemocline) and atmospheric air (above the chemocline). (c) A closer look on the biofilm where mark 7 shows the bare cave wall after biofilm sampling.

[Watch the video of bubbles drifting along the invisible stream of greenhouse gases (for fun).](https://www.gesslab.org/projects?wix-vod-video-id=a7aaef92d15549e6a5630d18e73484b7&wix-vod-comp-id=comp-jd73rsf5)

Data are precomputed and store at folder /data/precomputed/cave_data containing:
    - co-assembly (3 samples; 2 from biofilm and 1 from the laboratory)
    - Depth table
    - Binning results

!!! question "Exercise" 
    - Analyze the microbial bins to identify which organism is utilizing CH₄ (methane) for growth. What is the taxonomic classification of this organism? Investigate the presence of relevant gene clusters responsible for methane metabolism.
    - Investigate whether it is common for organisms within the identified taxon to utilize CH₄ for growth. If not, outline the steps and analyses you would perform to confirm and demonstrate this metabolic capability.
    - Perform a similar investigation to identify which organism is fixing CO₂ for growth. What is the taxonomy of this organism, and what gene clusters are involved in CO₂ fixation?
    
    
??? done "Tip 1: general directions"
	- Assess bin quality using CheckM to evaluate completeness and contamination levels.
	- Assign taxonomies to the bins with GTDB-Tk for precise classification.
	- Predict open reading frames (ORFs) using Prodigal and functionally annotate the bins through EggNOG for a deeper understanding of their metabolic capabilities.
	
??? done "Tip 2"
	- Investigate the gene annotation for methane monooxygenase and reductive TCA cycle and analyze the surrounding genomic regions to identify nearby genes and their associated protein functions.
	- Calculate the abundance of each bin by integrating the depth table with the binning results (use python/R).
	
	


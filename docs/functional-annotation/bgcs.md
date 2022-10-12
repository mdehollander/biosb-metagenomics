# Functional annotation - Biosynthetic gene clusters

## Step 1 -General overview: biosynthetic gene cluster identification

[Here](http://bioinformatics.nl/~medem005/antismash_relaxed/index.html) you can find antiSMASH results for the (simulated) metagenome that you analysed on Monday afternoon.
Clicking ‘Compact view’ helps to get a better overview for a metagenome output like this one.

??? done "1.	How many biosynthetic gene clusters (BGCs) did antiSMASH identify? Based on the results, which known compounds do you estimate this microbial community to be able to produce? Hint: take a look at the detailed knownclusterblast results for each cluster that has at least > 50% similarity on the gene level to a known cluster to assess this. Do (almost) all core enzymes encoded in the gene cluster show similarity to those in the reference gene cluster?"
    The bacillibactin, bacilysin, lichenysin and plantazolicin gene clusters look real. Others perhaps.

??? done "2.	One gene cluster looks like it is fragmented into two pieces. Which one? What could you do to try to assemble it into one piece?"
    The lichenysin gene cluster. It might be possible to assemble it by studying the assembly graph or using specialized tools such as biosyntheticSPAdes.

Using the ‘loose’ mode (see https://antismash.secondarymetabolites.org/), multiple putative BGCs can be identified by antiSMASH, which need to be analyzed manually to assess their value. [Here](http://bioinformatics.nl/~medem005/antismash_loose/index.html) you can find the antiSMASH results for the same genome, but now using the loose mode to predict more (putative) clusters.

??? done "3.	Take a look at some of the ‘newly added’ BGCs, and specifically look at the smCOG annotations and the knownclusterblast results. Can you identify some clusters that are very probable to encode the biosynthesis of an actual secondary metabolite? And can you find some clusters for which this is very unlikely? Note that some BGCs may be fragmented (visible by a note ‘Region on contig edge’) and may only show partial similarity to reference BGCs."
    Region 413.1 might encode a fragment of a real BGC, based on the fact that it lies on a contig edge, and the Knownclusterblast tab shows a match with part of a known BGC. Others are less likely real. Region 1068.1, for example, looks very doubtful, because it comprises only three identified enzyme-coding genes, separated by a large distance. 

## Step 2 - Disease-suppressive metabolites

Now go back to the original results.
Beneficial plant microbiota are sometimes able to protect plants against pathogens, a microbiome-associated phenotype known as disease suppression. One of the BGCs in this metagenome that might play a role for this is found in region 550.1; this BGC shows similarity to the bacilysin BGC from Bacillus velezensis FZB42 (previously known as Bacillus amyloliquefaciens FZB42).
Look up the cluster and check out the Knownclusterblast and MIBiG comparison tabs. 

??? done "4.	Do you think that region 550.1 is likely to encode the production of bacilysin? Why?"
    Only the transporter-encoding gene is missing, and all enzyme-coding genes seem to be similar. So yes, it is likely.

??? done "5.	In the knownclusterblast result, click on the MIBiG accession number of the first hit. This will take you to the reference entry of the known bacilysin BGC in the MIBiG repository. Check out the literature references. What do you learn about the biological activity of bacilysin? How might this molecule play a role in pathogen suppression?"
    Bacilysin is known to have antibacterial activity against bacterial phytopathogens such as Xanthomonas.

??? done "6.	Just downstream of the putative bacilysin gene cluster, you will find another large operon with many enzyme-coding genes. What do you think this operon might encode? Can you hypothesise a functional reason why these two operons are co-localized in the genome? Check out this paper for some hints (check section 3.4)."
    The paper suggests that bacilysin is produced by biofilm-producing bacilli. They might use it as protective metabolites, and the bacilli might provide double protection to plant roots by forming a protective coat while also producing antimicrobials.

## Step 3 - Peptidic fragments

Based on a mass spectrometry experiment, an apparently new natural product is identified from the strain. Based on fragmentation analysis, it appears to be a lipopeptide. A two-amino acid-long fragment is retrieved that is reconstructed based on mass shifts from tandem mass spectra, consisting of a valine and a leucine.

Peptide natural products can either be produced by multimodular enzymes called nonribosomal peptide synthetases (identified as ‘NRPS’ by antiSMASH) or through ribosomal synthesis and subsequent posttranslational modification (e.g., lanthipeptides, lasso peptides, thiopeptides, etc.). For both, chemical structure predictions may be provided (see the ‘NRPS/PKS modules’ or ‘Lanthipeptide/Lassopeptide’ tabs).

??? done "7.	Based on the antiSMASH results, which gene cluster do you think is most likely responsible for the biosynthesis of the peptide? What strategy did you use to find this out?"
    Region 1023.1. Two subsequent predicted amino acid substrates visible in the 'NRPS/PKS modules' tab match the monomers observed from the MS data.
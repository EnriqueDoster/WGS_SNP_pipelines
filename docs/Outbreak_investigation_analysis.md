

# Method used to compare results from phylogenetic trees for Lyve-SET
https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5346554/


Historically, it has been difficult to evaluate and compare SNP pipelines, and an even bigger challenge to compare them to workflows based on other algorithms (e.g., wgMLST). Each of the aforementioned pipelines produces output that can be used for interpreting the relationship between genomes in various forms such as distance matrices, MSAs, and dendrograms; however, they have different underlying algorithms and output formats. For example, each SNP pipeline uses a different read mapper and SNP caller and might produce a different format to describe their SNP calls. In comparing wgMLST and SNP workflows which are wholly different algorithms, one SNP might be located in an intergenic region, yielding zero allelic differences by wgMLST; on the other hand many SNPs might be located on a single gene, yielding the collapse of multiple SNPs into a single allelic difference.

A reasonable approach to pipeline comparison, therefore, might be at the phylogenetic level. A classic comparison method is the Robinson-Foulds metric, sometimes called the symmetric difference metric, where the number of internal branches that exist in one tree but not the other are counted (Robinson and Foulds, 1981). Another metric is Kuhner-Felsenstein, sometimes called “branch score” which is similar to Robinson-Foulds but calculates the Euclidean distance between each branch's length (Kuhner and Felsenstein, 1994). Both Robinson-Foulds and Kuhner-Felsenstein metrics are implemented in the Phylip package in the program treedist (Felsenstein, 1989) and in some programming language libraries such as Bio::Phylo (Vos et al., 2011). Both of these classical metrics rely on unrooted trees, and small differences between two trees can artificially magnify the distance between two trees. A more robust tree metric—the Kendall-Colijn—accounts for both tree topology and branch length (Kendall and Colijn, 2015). The Kendall-Colijn metric compares two rooted trees using Euclidean distances from tip to root with a coefficient λ to give more weight to either topology (λ = 0) or branch length (λ = 1). One more reasonable approach to pipeline comparison is assessing the distance matrices between two workflows. The Mantel test uses a generalized regression approach to identify correlations between two distance matrices (Smouse et al., 1986). Therefore, if the genome distances from one workflow vs. another workflow are consistently higher but correlate well, the Mantel test will yield a high correlation coefficient.

In this article, we describe the Lyve-SET workflow, demonstrate how it can aid in bacterial foodborne outbreak investigations, and propose methods of comparison with other phylogenetic workflows.


## Outbreak clusters
Twelve outbreak clusters of four major foodborne pathogens were queried from the PulseNet database (Table ​(Table3;3; Swaminathan et al., 2006). PulseNet is a national laboratory network that tracks the subtypes of bacteria causing foodborne illness cases to detect outbreaks. The inclusion or exclusion of isolates for each outbreak was determined using evidence gathered during outbreak investigations including WGS, molecular subtyping, demographic, and exposure data. Isolates from each outbreak were identified; however, some outbreak isolates were excluded when differing from the main outbreak clade by 200 or more hqSNPs. To place these outbreak genomes in a global context, we queried the NCBI k-mer trees from April 2016 (Accessions: PDG000000001.428, PDG000000002.629, PDG000000003.184, PDG000000004.427; Data Sheet 1). Each NCBI k-mer tree is generated by the NCBI Pathogen Detection Pipeline and is a dendrogram of all publically available genomes (https://www.ncbi.nlm.nih.gov/pathogens). Briefly, NCBI creates high-quality genome assemblies. The MinHash algorithm is applied to each genome, and a Jaccard distance is calculated between each pair of genomes. Then, NCBI creates a tree based on the Jaccard distances. Closely related genomes are refined into subclades using SNPs found among these related assemblies, and a subtree is created with FastME (Lefort et al., 2015). After locating the outbreak clade, we advanced one to three ancestral nodes to acquire a population of potentially related descendent genomes (Data Sheet 2). True positive (TP) isolates are those identified to be associated with the outbreak by PulseNet; true negative (TN) isolates are not associated with the outbreak. For each bioinformatics pipeline to calculate sensitivity (Sn) and specificity (Sp), we also needed to find misidentified genomes, namely the false positives (FP), and false negatives (FN). Sn is calculated as TP/(TP+FN); Sp is calculated as TN/(TN+FP).

## Statistical tests
Three categories of pipeline comparisons were performed: the Sn and Sp of outbreak isolates included in the target outbreak clade, tests of tree topology, and tests of variant positions and distances.

## Comparing trees 
If an isolate fell into the same well-supported clade as outbreak isolates (node confidence value ≥70%), it was counted as a positive. Otherwise, it was counted as a negative. Positives that retrospectively agree with the outbreak investigation were counted as TP; otherwise, FN. The target well-supported clade is defined as a having a confidence value >70% and being as complementary as possible for Sn and Sp for each tree. Sn was calculated as TP/(TP+FN). Sp was calculated as TN/(TN+FP). For the following statistical scripts, Perl v5.16.1 and R v3.3.0 were used.
To compare trees, we implemented the Kendall-Colijn and Robinson-Foulds tests (Robinson and Foulds, 1981; Kendall and Colijn, 2015). The Kendall-Colijn test was implemented in the R package Treescape v1.9.17. The background distribution of trees is a set of 105 random trees using the APE package in R (Paradis et al., 2004). Each random tree was created with the R function rtree, with the taxon names shuffled. The query tree was compared against the background distribution and then against the Lyve-SET tree. A Z-test was performed; a p < 0.05 indicates that the query tree is closely related to the Lyve-SET tree. The Robinson-Foulds metric, also known as the symmetric difference, was implemented in the Perl package Bio::Phylo and was compared against 105 random trees generated in BioPerl (Stajich et al., 2002; Vos et al., 2011). The query tree, i.e., an observed tree from wgMLST or from a SNP pipeline, was compared against the random distribution and against the Lyve-SET tree. A Z-test was performed to compare the distances against the random distribution and the distance vs. Lyve-SET. A significant p-value (α < 0.05) indicates that the query tree is more closely related to the Lyve-SET tree topology than would be expected by chance. Given that low-confidence nodes would not be considered during an outbreak investigation, we removed low-confidence nodes (bootstrap support <70%), potentially creating multifurcating trees, before performing the Kendall-Colijn test. From this transformation, 47 out of 63 trees became multifurcating for this comparison. Only one Lyve-SET tree, the three wgMLST trees, and 12 RealPhy trees remained binary. The Robinson-Foulds test does not tolerate multifurcation; therefore low-confidence nodes were not removed for those tests. Unless otherwise indicated, all trees were midpoint-rooted. All statistical scripts used in this study are available at https://github.com/lskatz/Lyve-SET-paper.

# Comparing distances and SNP locations
To compare genetic distances, we plotted each pairwise distance between genomes into a scatter plot, with the x-axis representing Lyve-SET SNPs and the y-axis representing the distance calculated from the other pipeline. This produced one scatter plot per outbreak dataset. Additionally, we used linear regression analysis on each dataset to create a trend line with a slope indicative of calculated distance per Lyve-SET SNP and an R2 value indicative of goodness-of-fit. We combined all datasets into graphs of each of the four species in this study. Because Lyve-SET is mainly used for outbreak datasets, we also produced scatter plots which only included outbreak-associated genomes such that we could limit the influence of non-outbreak-associated isolates. Jackson et al. (2016) reported an empirical 50-hqSNP distance between outbreak isolates. We observed similar maximum thresholds for all 12 outbreaks in this study for each species. Some distances in the outbreak-only scatter plots were outliers with a clear separation between <50 and >100 on the distance axis (Data Sheet 3); therefore in the context of analyses comparing only within outbreak-associated isolates, distances > 100 from non-Lyve-SET pipelines were removed.
We also assessed the correlation between the pairwise distance matrices directly using the Mantel test implemented in the R package Vegan v2.4.0 using the Spearman correlation and 1000 permutations (Mantel, 1967; Oksanen et al., 2007). Each query was compared against Lyve-SET.

To compare SNP positions, the set of SNPs from Lyve-SET was used as reference even though no one pipeline can predict with 100% confidence the correct locations of all SNPs. If a query SNP agreed with the position of the Lyve-SET SNP, it was considered as a TP; if the pipeline excluded a position as a SNP that Lyve-SET excluded, then it was a TN. Sn and Sp were calculated as in the test for Sn/Sp of outbreak isolates.


# Revelations from metagenomic read classification using `kraken2` 

#### Colin Wood and Chloe Herman,
September 25, 2024


### Quality Score 

Kraken2 converts bases that are less than the given quality threshold into an ambiguous base. This has the effect that the $k$ kmers that touch the ambiguous base will be classified as ambiguous as the kmer window is slid down the read. In general, any kmer that contains an ambiguous base will be classified as ambiguous (and won't be taken into account for the read classification in which the kmer resides).

### Read Merging

Merging reads before classifying with kraken2 seems to greatly improve the percentage that are classified. We hypothesize that this is because the border between two unmerged (but mergable) reads breaks up the kmer sliding process. If paired end reads are supplied to kraken unmerged kraken concatenates the reads by placing an "N" between the two and classifying the pair as a single read. This means that there are approximately $k$ kmers that *would have been* be classified if the reads were pre-merged. 

![[merged-reads.png|400]]

### Confidence

Kraken2 has a `confidence` parameter that trades off between classification sensitivity and specificity. The formula is explained below:

> A sequence label's score is a fraction C/$Q$, where C is the number of k-mers mapped to LCA values in the clade rooted at the label, and Q is the number of k-mers in the sequence that lack an ambiguous nucleotide (i.e., they were queried against the database).

Put differently, this read confidence proportion $RC$ can be thought of as:

$$ RC = \frac{|C_c|}{|C| + |U|} $$

where $C_c$ is the set of kmers in the clade rooted at the label, $C$ is the set of all classified kmers, and $U$ is the set of all unclassified kmers. If for some confidence parameter $P$, $RC < P$, then the label is adjusted back up the LCA tree until $P$ is exceeded. If $P$ can not be exceeded then the read becomes unclassified.  

See the [kraken2 manual](https://github.com/DerrickWood/kraken2/blob/master/docs/MANUAL.markdown#confidence-scoring) for an example.

**Taxonomic resolution**
We hypothesize that the confidence parameter affects the taxonomic resolution of a sample in the following fashion. Lower confidence values will result in greater taxonomic resolution because read classifications will be to deeper taxonomic levels. Note that there is inherently a trade off between false positives and false negatives--FPs increasing as confidence is lowered and NPs increasing as confidence is raised.

**Alpha diversity**
We hypothesize that the confidence parameter affects the alpha diversity of a sample in the following fashion. All else equal, a lower confidence parameter should result in a higher alpha diversity value for the simple reason that lower confidence values will result in deeper taxonomic classifications, thus increasing the chances that any two reads will be classified as distinct taxa.

**Beta diversity**
We hypothesize that the confidence parameter affects the beta diversity of two samples in the following fashion. Assuming that the two sets of taxa in these samples are at least somewhat disjoint, then lowering the confidence parameter will have the effect of raising the beta diversity distance. This is because the alpha diversities of each will raise (see above) making it more likely that unique taxa in sample 1 or sample 2 will be classified, thus increasing beta diversity distance.

**Predictions of differing confidence values on Chloe's FMT time series data**
- In the hypothetical case of **no engraftment**, we hypothesize that confidence value parameter can have no effect on the beta-diversity derived signal of engraftment. When comparing the pre-treatment beta diversity to post-treatment beta diversity, if no engraftment occurred then no change in beta diversity should be observed following FMT--thus adjusting the confidence parameter would have an equivalent effect on both beta diversities, and no signal can be artificially created. 
- In the hypothetical case of **any nonzero engraftment extent**, we hypothesize that lowering the confidence parameter will disproportionately raise the pre-treatment time point beta diversity because features are less likely to be shared here, and will have less of an effect on raising the beta diversity of the post-FMT time point because features are more likely to be shared. **Note** that this case is more complex than the no engraftment case, and should ideally be verified with simulations.

![[timepoints.png|400]]

### Adapter Trimming

We found that running kraken2 with adapters still present in our reads rendered the classification results completely unusable. If it is unknown whether reads contain adapters, the [FastQC software](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) is a useful tool to use because it searches many commonly used Illumina adapters. 


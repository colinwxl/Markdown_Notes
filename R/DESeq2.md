# Analyzing RNA-seq data with DESeq2
[2018-0905]
## Abstract
A basic task in the analysis of count data from RNA-seq is the detection of differentially expressed genes. An important analysis question is the quantification and statistical inference of systematic changes between conditions, as compared to within-condition variability. The package DESeq2 provides methods to test for differential expression by use of negative binomial generalized linear models; the estimates of dispersion and logarithmic fold changes incorporate data-driven prior distributions. This vignette explains the use of the package and demonstrates typical workflows.

## Publication
Love, M.I., Huber, W., Anders, S. (2014) Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2. Genome Biology, 15:550. [10.1186/s13059-014-0550-8](http://dx.doi.org/10.1186/s13059-014-0550-8)

## Quick start
This code chunk assumes that you have a count matrix called `cts` and a table of sample information called `coldata`. The `design` indicates how to model the samples, here, that we want to measure the effect of the condition, controlling for batch differences. The two factor variables `batch` and `condition` should be columns of `coldata`.
```R
dds <- DESeqDataSetFromMatrix(countData = cts,
                              colData = coldata,
                              design= ~ batch + condition)
dds <- DESeq(dds)
resultsNames(dds) # lists the coefficients
res <- results(dds, name="condition_trt_vs_untrt")
# or to shrink log fold changes association with condition:
res <- lfcShrink(dds, coef="condition_trt_vs_untrt", type="apeglm")
```
The following starting functions will be explained below:
- If you have transcript quantification files, as produced by Salmon, Sailfish, or kallisto, you would use DESeqDataSetFromTximport.
- If you have htseq-count files, the first line would use DESeqDataSetFromHTSeq.
- If you have a RangedSummarizedExperiment, the first line would use DESeqDataSet.

## Input data
### Why un-normalized counts?
As input, the DESeq2 package expects count data as obtained, e.g., from RNA-seq or another high-throughput sequencing experiment, in the form of a matrix of integer values. The value in the i-th row and the j-th column of the matrix tells how many reads can be assigned to gene i in sample j. Analogously, for other types of assays, the rows of the matrix might correspond e.g. to binding regions (with ChIP-Seq) or peptide sequences (with quantitative mass spectrometry). We will list method for obtaining count matrices in sections below.

The values in the matrix should be un-normalized counts or estimated counts of sequencing reads (for single-end RNA-seq) or <font color=red>fragments (for paired-end RNA-seq)</font>. The RNA-seq workflow describes multiple techniques for preparing such count matrices. It is important to provide count matrices as input for DESeq2’s statistical model (Love, Huber, and Anders 2014) to hold, as only the count values allow assessing the measurement precision correctly. The DESeq2 model internally corrects for library size, so transformed or normalized values such as counts scaled by library size should not be used as input.

### The DESeqDataSet
The object class used by the DESeq2 package to store the read counts and the intermediate estimated quantities during statistical analysis is the **DESeqDataSet**, which will usually be represented in the code here as an object `dds`.

A **DESeqDataSet** object must have an associated design formula. The design formula expresses the variables which will be used in modeling. The formula should be a tilde (~) followed by the variables with plus signs between them (it will be coerced into an formula if it is not already). The design can be changed later, however then all differential analysis steps should be repeated, as the design formula is used to estimate the dispersions and to estimate the log2 fold changes of the model.

*Note*: In order to benefit from the default settings of the package, you should put the variable of interest at the end of the formula and make sure the control level is the first level.

We will now show 4 ways of constructing a DESeqDataSet, depending on what pipeline was used upstream of DESeq2 to generated counts or estimated counts:
- From [transcript abundance files and tximport](http://www.bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#tximport)
- From a [count matrix](http://www.bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#countmat) <font color=red>(foucus)</font>
- From [htseq-count files](http://www.bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#htseq)
- From a [SummarizedExperiment](http://www.bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#se) object

### Count matrix input
Alternatively, the function *DESeqDataSetFromMatrix* can be used if you already have a matrix of read counts prepared from another source. Another method for quickly producing count matrices from alignment files is the *featureCounts* function (Liao, Smyth, and Shi 2013) in the Rsubread package. To use *DESeqDataSetFromMatrix*, the user should provide the counts matrix, the information about the samples (the columns of the count matrix) as a *DataFrame* or *data.frame*, and the design formula.

To demonstate the use of *DESeqDataSetFromMatrix*, we will read in count data from the [pasilla](http://bioconductor.org/packages/pasilla) package. We read in a count matrix, which we will name `cts`, and the sample information table, which we will name `coldata`. Further below we describe how to extract these objects from, e.g. *featureCounts* output.
```R
library("pasilla")
pasCts <- system.file("extdata",
                      "pasilla_gene_counts.tsv",
                      package="pasilla", mustWork=TRUE)
pasAnno <- system.file("extdata",
                       "pasilla_sample_annotation.csv",
                       package="pasilla", mustWork=TRUE)
cts <- as.matrix(read.csv(pasCts,sep="\t",row.names="gene_id"))
coldata <- read.csv(pasAnno, row.names=1)
coldata <- coldata[,c("condition","type")]

# We examine the count matrix and column data to see if they are consistent in terms of sample order.
head(cts,2)
coldata
```
Note that these are not in the same order with respect to samples!
```R
# chop off the "fb" of the row names of coldata
rownames(coldata) <- sub("fb", "", rownames(coldata))
all(rownames(coldata) %in% colnames(cts))
# reorder
cts <- cts[, rownames(coldata)]
all(rownames(coldata) == colnames(cts))

library("DESeq2")
dds <- DESeqDataSetFromMatrix(countData = cts,
                              colData = coldata,
                              design = ~ condition)
dds
```
If you have additional feature data, it can be added to the *DESeqDataSet* by adding to the metadata columns of a newly constructed object. (Here we add redundant data just for demonstration, as the gene names are already the rownames of the `dds`.)
```R
featureData <- data.frame(gene=rownames(cts))
mcols(dds) <- DataFrame(mcols(dds), featureData)
mcols(dds)
```
### Pre-filtering
While it is not necessary to pre-filter low count genes before running the DESeq2 functions, there are two reasons which make pre-filtering useful: by removing rows in which there are very few reads, we reduce the memory size of the `dds` data object, and we increase the speed of the transformation and testing functions within DESeq2. Here we perform a minimal pre-filtering to keep only rows that have at least 10 reads total. Note that more strict filtering to increase power is *automatically* applied via [independent filtering](http://www.bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#indfilt) on the mean of normalized counts within the *results* function.
```R
keep <- rowSums(counts(dds)) >= 10
dds <- dds[keep,]
```
### Note on factor levels
By default, R will choose a *reference level* for factors based on alphabetical order. Then, if you never tell the DESeq2 functions which level you want to compare against (e.g. which level represents the control group), the comparisons will be based on the alphabetical order of the levels. There are two solutions: you can either explicitly tell results which comparison to make using the `contrast` argument (this will be shown later), or you can explicitly set the factors levels. You should only change the factor levels of variables in the design before running the DESeq2 analysis, not during or afterward. Setting the factor levels can be done in two ways, either using factor:
```R
dds$condition <- factor(dds$condition, levels = c("untreated","treated"))
```
…or using relevel, just specifying the reference level:
```R
dds$condition <- relevel(dds$condition, ref = "untreated")
```
If you need to subset the columns of a *DESeqDataSet*, i.e., when removing certain samples from the analysis, it is possible that all the samples for one or more levels of a variable in the design formula would be removed. In this case, the *droplevels* function can be used to remove those levels which do not have samples in the current *DESeqDataSet*:
```R
dds$condition <- droplevels(dds$condition)
```
### Collapsing technical replicates
DESeq2 provides a function *collapseReplicates* which can assist in combining the counts from technical replicates into single columns of the count matrix. The term technical replicate implies multiple sequencing runs of the same library. You <font color=red>should not collapse biological replicates using this function</font>. See the manual page for an example of the use of *collapseReplicates*.

## Differential expression analysis
The standard differential expression analysis steps are wrapped into a single function, *DESeq*. The estimation steps performed by this function are described below, in the manual page for `?DESeq` and in the Methods section of the DESeq2 publication.

Results tables are generated using the function *results*, which extracts a results table with <font color=red>log2 fold changes, p values and adjusted p values</font>. With no additional arguments to results, the log2 fold change and Wald test p value will be for the **last variable** in the design formula, and if this is a factor, the comparison will be the **last level** of this variable over the **reference level** (see previous note on factor levels). However, the order of the variables of the design do not matter so long as the user specifies the comparison to build a results table for, using the `name` or `contrast` arguments of results.

Details about the comparison are printed to the console, directly above the results table. The text, `condition treated vs untreated`, tells you that the estimates are of the logarithmic fold change log2(treated/untreated).
```R
dds <- DESeq(dds)
res <- results(dds)
res
```
Note that we could have specified the coefficient or contrast we want to build a results table for, using either of the following equivalent commands:
```R
res <- results(dds, name="condition_treated_vs_untreated")
res <- results(dds, contrast=c("condition","treated","untreated"))
```
More information about extracting specific coefficients from a fitted *DESeqDataSet* object can be found in the help page `?results`. The use of the `contrast` argument is also further discussed below.

### Log fold change shrinkage for visualization and ranking
Shrinkage of effect size (LFC estimates) is useful for visualization and ranking of genes. To shrink the LFC, we pass the `dds` object to the function *lfcShrink*. Below we specify to use the *apeglm* method for effect size shrinkage (Zhu, Ibrahim, and Love 2018), which improves on the previous estimator.

We provide the `dds` object and the name or number of the coefficient we want to shrink, where the number refers to the order of the coefficient as it appears in `resultsNames(dds)`.
```R
resultsNames(dds)
resLFC <- lfcShrink(dds, coef="condition_treated_vs_untreated", type="apeglm")
resLFC
```
### Using parallelization
The above steps should take less than 30 seconds for most analyses. For experiments with complex designs and many samples (e.g. dozens of coefficients, ~100s of samples), one can take advantage of parallelized computation. Parallelizing `DESeq`, `results`, and `lfcShrink` can be easily accomplished by loading the BiocParallel package, and then setting the following arguments: `parallel=TRUE` and `BPPARAM=MulticoreParam(4)`, for example, splitting the job over 4 cores. Note that `results` for coefficients or contrasts listed in `resultsNames(dds)` is fast and will not need parallelization. As an alternative to `BPPARAM`, one can `register` cores at the beginning of an analysis, and then just specify `parallel=TRUE` to the functions when called.
```R
library("BiocParallel")
register(MulticoreParam(4))
```

### p-values and adjusted p-values
We can order our results table by the smallest p value:
```R
resOrdered <- res[order(res$pvalue),]
summary(res)
# How many adjusted p-values were less than 0.1?
sum(res$padj < 0.1, na.rm=TRUE)
```
The *results* function contains a number of arguments to customize the results table which is generated. You can read about these arguments by looking up `?results`. Note that the *results* function automatically performs independent filtering based on the mean of normalized counts for each gene, optimizing the number of genes which will have an adjusted p value below a given FDR cutoff, `alpha`. Independent filtering is further discussed below. By default the argument `alpha` is set to 0.1. If the adjusted p value cutoff will be a value other than 0.1, `alpha` should be set to that value:
```R
res05 <- results(dds, alpha=0.05)
summary(res05)
sum(res05$padj < 0.05, na.rm=TRUE)
```
### Independent hypothesis weighting
A generalization of the idea of p value filtering is to weight hypotheses to optimize power. A Bioconductor package, [IHW](http://bioconductor.org/packages/IHW), is available that implements the method of Independent Hypothesis Weighting (Ignatiadis et al. 2016). Here we show the use of IHW for p value adjustment of DESeq2 results. For more details, please see the vignette of the IHW package. The IHW result object is stored in the metadata.

Note: If the results of independent hypothesis weighting are used in published research, please cite:
Ignatiadis, N., Klaus, B., Zaugg, J.B., Huber, W. (2016) Data-driven hypothesis weighting increases detection power in genome-scale multiple testing. Nature Methods, 13:7. [10.1038/nmeth.3885](http://dx.doi.org/10.1038/nmeth.3885)

```R
library("IHW")
resIHW <- results(dds, filterFun=ihw)
summary(resIHW)
sum(resIHW$padj < 0.1, na.rm=TRUE)
metadata(resIHW)$ihwResult
```

## Exploring and exporting results
### MA-plot
In DESeq2, the function *plotMA* shows the log2 fold changes attributable to a given variable over the mean of normalized counts for all the samples in the *DESeqDataSet*. Points will be colored red if the adjusted p value is less than 0.1. Points which fall out of the window are plotted as open triangles pointing either up or down.
```R
plotMA(res, ylim=c(-2,2))
```
It is more useful visualize the MA-plot for the shrunken log2 fold changes, which remove the noise associated with log2 fold changes from low count genes without requiring arbitrary filtering thresholds.
```R
plotMA(resLFC, ylim=c(-2,2))
```
After calling *plotMA*, one can use the function *identify* to interactively detect the row number of individual genes by clicking on the plot. One can then recover the gene identifiers by saving the resulting indices:
```R
idx <- identify(res$baseMean, res$log2FoldChange)
rownames(res)[idx]
```
### Alternative shrinkage estimators
The moderated log fold changes proposed by Love, Huber, and Anders (2014) use a normal prior distribution, centered on zero and with a scale that is fit to the data. <font color=red>The shrunken log fold changes are useful for ranking and visualization, without the need for arbitrary filters on low count genes.</font> The normal prior can sometimes produce too strong of shrinkage for certain datasets. In DESeq2 version 1.18, we include two additional adaptive shrinkage estimators, available via the `type` argument of `lfcShrink`. For more details, see `?lfcShrink`.

The options for `type` are:
- `normal` is the the original DESeq2 shrinkage estimator, an adaptive Normal distribution as prior. This is currently the default, although the default will likely change to `apeglm` in the October 2018 release given `apeglm`’s superior performance.
- `apeglm` is the adaptive t prior shrinkage estimator from the apeglm package (Zhu, Ibrahim, and Love 2018).
- `ashr` is the adaptive shrinkage estimator from the ashr package (Stephens 2016). Here DESeq2 uses the ashr option to fit a mixture of Normal distributions to form the prior, with `method="shrinkage"`.

If the shrinkage estimator `apeglm` is used in published research, please cite:
>> Zhu, A., Ibrahim, J.G., Love, M.I. (2018) Heavy-tailed prior distributions for sequence count data: removing the noise and preserving large differences. bioRxiv. 10.1101/303255
If the shrinkage estimator `ashr` is used in published research, please cite:
>> Stephens, M. (2016) False discovery rates: a new deal. Biostatistics, 18:2. 10.1093/biostatistics/kxw041

In the LFC shrinkage code above, we specified `coef="condition_treated_vs_untreated"`. We can also just specify the coefficient by the order that it appears in `resultsNames(dds)`, in this case `coef=2`. For more details explaining how the shrinkage estimators differ, and what kinds of designs, contrasts and output is provided by each, see the extended section on shrinkage estimators.
```R
resultsNames(dds)
# because we are interested in treated vs untreated, we set 'coef=2'
resNorm <- lfcShrink(dds, coef=2, type="normal")
resAsh <- lfcShrink(dds, coef=2, type="ashr")
par(mfrow=c(1,3), mar=c(4,4,2,1))
xlim <- c(1,1e5); ylim <- c(-3,3)
plotMA(resLFC, xlim=xlim, ylim=ylim, main="apeglm")
plotMA(resNorm, xlim=xlim, ylim=ylim, main="normal")
plotMA(resAsh, xlim=xlim, ylim=ylim, main="ashr")
```
**Note:** We have sped up the `apeglm` method so it takes roughly about the same amount of time as `normal`, e.g. ~5 seconds for the `pasilla` dataset of ~10,000 genes and 7 samples. If fast shrinkage estimation of LFC is needed, but the posterior standard deviation is not needed, setting `apeMethod="nbinomC"` will produce a ~10x speedup, but the `lfcSE` column will be returned with `NA`. A variant of this fast method, `apeMethod="nbinomC*"` includes random starts.
**Note:** If there is unwanted variation present in the data (e.g. batch effects) it is always recommend to correct for this, which can be accommodated in DESeq2 by including in the design any known batch variables or by using functions/packages such as `svaseq` in [sva](http://bioconductor.org/packages/sva) (Leek 2014) or the `RUV` functions in [RUVSeq](http://bioconductor.org/packages/RUVSeq) (Risso et al. 2014) to estimate variables that capture the unwanted variation. In addition, the ashr developers have a specific method for accounting for unwanted variation in combination with ashr (Gerard and Stephens 2017).

### Plot counts
It can also be useful to examine the counts of reads for a single gene across the groups. A simple function for making this plot is plotCounts, which normalizes counts by sequencing depth and adds <font color=red>a pseudocount of 1/2</font> to allow for log scale plotting. The counts are grouped by the variables in `intgroup`, where more than one variable can be specified. Here we specify the gene which had the smallest p value from the results table created above. You can select the gene to plot by rowname or by numeric index.
```R
plotCounts(dds, gene=which.min(res$padj), intgroup="condition")
```
For customized plotting, an argument `returnData` specifies that the function should only return a *data.frame* for plotting with *ggplot*.
```R
d <- plotCounts(dds, gene=which.min(res$padj), intgroup="condition",
                returnData=TRUE)
library("ggplot2")
ggplot(d, aes(x=condition, y=count)) + 
  geom_point(position=position_jitter(w=0.1,h=0)) + 
  scale_y_log10(breaks=c(25,100,400))
```

### More information on results columns
Information about which variables and tests were used can be found by calling the function *mcols* on the results object.
```R
mcols(res)$description
```
**Note on p-values set to NA:** some values in the results table can be set to `NA` for one of the following reasons:
- If within a row, all samples have zero counts, the `baseMean` column will be zero, and the log2 fold change estimates, p value and adjusted p value will all be set to `NA`.
- If a row contains a sample with an extreme count outlier then the p value and adjusted p value will be set to NA. These outlier counts are detected by Cook’s distance. Customization of this outlier filtering and description of functionality for replacement of outlier counts and refitting is described below
- If a row is filtered by automatic independent filtering, for having a low mean normalized count, then only the adjusted p value will be set to `NA`. Description and customization of independent filtering is described below.

### Rich visualization and reporting of results
- ReportingTools
- regionReport
- Glimma
- pcaExplorer

### Exporting results to CSV files
A plain-text file of the results can be exported using the base R functions *write.csv* or *write.delim*. We suggest using a descriptive file name indicating the variable and levels which were tested.
```R
write.csv(as.data.frame(resOrdered), 
          file="condition_treated_results.csv")
# Exporting only the results which pass an adjusted p value threshold can be accomplished with the subset function, followed by the write.csv function.
resSig <- subset(resOrdered, padj < 0.1)
resSig
```

## Multi-factor designs
Experiments with more than one factor influencing the counts can be analyzed using design formula that include the additional variables. In fact, DESeq2 can analyze any possible experimental design that can be expressed with fixed effects terms (multiple factors, designs with interactions, designs with continuous variables, splines, and so on are all possible).

By adding variables to the design, one can control for additional variation in the counts. For example, if the condition samples are balanced across experimental batches, by including the `batch` factor to the design, one can increase the sensitivity for finding differences due to `condition`. There are multiple ways to analyze experiments when the additional variables are of interest and not just controlling factors (see section on interactions).

The data in the *pasilla* package have a condition of interest (the column condition), as well as information on the type of sequencing which was performed (the column type), as we can see below:
```R
colData(dds)
# We create a copy of the DESeqDataSet, so that we can rerun the analysis using a multi-factor design.
ddsMF <- dds
levels(ddsMF$type) <- sub("-.*", "", levels(ddsMF$type))
levels(ddsMF$type)
design(ddsMF) <- formula(~ type + condition)
ddsMF <- DESeq(ddsMF)
resMF <- results(ddsMF)
head(resMF)

resMFType <- results(ddsMF,
                     contrast=c("type", "single", "paired"))
head(resMFType)
```

## Data transformations and visualization
### Count data transformations
In order to test for differential expression, we operate on raw counts and use discrete distributions as described in the previous section on differential expression. However for other downstream analyses – e.g. for visualization or clustering – it might be useful to work with transformed versions of the count data.

Maybe the most obvious choice of transformation is the logarithm. Since count values for a gene can be zero in some conditions (and non-zero in others), some advocate the use of pseudocounts, i.e. transformations of the form:
$$y=log_2(n+n_0)$$

where $n$ represents the count values and $n_0$ is a positive constant.

In this section, we discuss two alternative approaches that offer more theoretical justification and a rational way of choosing parameters equivalent to $n_0$ above. One makes use of the concept of variance stabilizing transformations (VST) (Tibshirani 1988; Huber et al. 2003; Anders and Huber 2010), and the other is the *regularized logarithm* or *rlog*, which incorporates a prior on the sample differences (Love, Huber, and Anders 2014). Both transformations produce transformed data on the log2 scale which has been normalized with respect to library size or other normalization factors.

The point of these two transformations, the VST and the rlog, is to <font color=red>remove the dependence of the variance on the mean</font>, particularly the high variance of the logarithm of count data when the mean is low. Both VST and rlog use the experiment-wide trend of variance over mean, in order to transform the data to remove the experiment-wide trend. Note that we do not require or desire that all the genes have exactly the same variance after transformation. Indeed, in a figure below, you will see that after the transformations the genes with the same mean do not have exactly the same standard deviations, but that the experiment-wide trend has flattened. It is those genes with row variance above the trend which will allow us to cluster samples into interesting groups.

**Note on running time:** if you have many samples (e.g. 100s), the rlog function might take too long, and so the vst function will be a faster choice. The rlog and VST have similar properties, but the rlog requires fitting a shrinkage term for each sample and each gene which takes time. See the DESeq2 paper for more discussion on the differences (Love, Huber, and Anders 2014).

### Blind dispersion estimation
The two functions, vst and rlog have an argument `blind`, for whether the transformation should be blind to the sample information specified by the design formula. When `blind` equals `TRUE` (the default), the functions will re-estimate the dispersions using only an intercept. This setting should be used in order to compare samples in a manner wholly unbiased by the information about experimental groups, for example to perform sample QA (quality assurance) as demonstrated below.

However, blind dispersion estimation is not the appropriate choice if one expects that many or the majority of genes (rows) will have large differences in counts which are explainable by the experimental design, and one wishes to transform the data for downstream analysis. In this case, using blind dispersion estimation will lead to large estimates of dispersion, as it attributes differences due to experimental design as unwanted noise, and will result in overly shrinking the transformed values towards each other. By setting `blind` to `FALSE`, the dispersions already estimated will be used to perform transformations, or if not present, they will be estimated using the current design formula. Note that only the fitted dispersion estimates from mean-dispersion trend line are used in the transformation (the global dependence of dispersion on mean for the entire experiment). So setting `blind` to `FALSE` is still for the most part not using the information about which samples were in which experimental group in applying the transformation.

### Extracting transformed values
These transformation functions return an object of class *DESeqTransform* which is a subclass of *RangedSummarizedExperiment*. For ~20 samples, running on a newly created *DESeqDataSet*, rlog may take 30 seconds, while vst takes less than 1 second. The running times are shorter when using `blind=FALSE` and if the function *DESeq* has already been run, because then it is not necessary to re-estimate the dispersion values. The *assay* function is used to extract the matrix of normalized values.
```R
vsd <- vst(dds, blind=FALSE)
rld <- rlog(dds, blind=FALSE)
head(assay(vsd), 3)
```

### Variance stabilizing transformation
Above, we used a parametric fit for the dispersion. In this case, the closed-form expression for the variance stabilizing transformation is used by the vst function. If a local fit is used (option `fitType="locfit"` to *estimateDispersions*) a numerical integration is used instead. The transformed data should be approximated variance stabilized and also includes correction for size factors or normalization factors. The transformed data is on the log2 scale for large counts.

### Regularized log transformation
The function rlog, stands for regularized log, transforming the original count data to the log2 scale by fitting a model with a term for each sample and a prior distribution on the coefficients which is estimated from the data. This is the same kind of shrinkage (sometimes referred to as regularization, or moderation) of log fold changes used by the *DESeq* and *nbinomWaldTest*. The resulting data contains elements defined as:

$$log_2(q_{ij})=β_{i0}+β_{ij}$$

where $q_{ij}$ is a parameter proportional to the expected true concentration of fragments for gene i and sample j (see formula below), $β_{i0}$ is an intercept which does not undergo shrinkage, and $β_{ij}$ is the sample-specific effect which is shrunk toward zero based on the dispersion-mean trend over the entire dataset. The trend typically captures high dispersions for low counts, and therefore these genes exhibit higher shrinkage from the rlog.

Note that, as $q_{ij}$ represents the part of the mean value $μ_{ij}$ after the size factor $s_j$ has been divided out, it is clear that the rlog transformation inherently accounts for differences in sequencing depth. Without priors, this design matrix would lead to a non-unique solution, however the addition of a prior on non-intercept betas allows for a unique solution to be found.

### Effects of transformations on the variance
The figure below plots the standard deviation of the transformed data, across samples, against the mean, using the shifted logarithm transformation, the regularized log transformation and the variance stabilizing transformation. The shifted logarithm has elevated standard deviation in the lower count range, and the regularized log to a lesser extent, while for the variance stabilized data the standard deviation is roughly constant along the whole dynamic range.

Note that the vertical axis in such plots is the square root of the variance over all samples, so including the variance due to the experimental conditions. While a flat curve of the square root of variance over the mean may seem like the goal of such transformations, this may be unreasonable in the case of datasets with many true differences due to the experimental conditions.

```R
ntd <- normTransform(dds)
library("vsn")
meanSdPlot(assay(ntd))
meanSdPlot(assay(vsd))
meanSdPlot(assay(rld))
```

## Data quality assessment by sample clustering and visualization
Data quality assessment and quality control (i.e. the removal of insufficiently good data) are essential steps of any data analysis. These steps should typically be performed very early in the analysis of a new data set, preceding or in parallel to the differential expression testing.

We define the term *quality* as *fitness for purpose*. Our purpose is the detection of differentially expressed genes, and we are looking in particular for samples whose experimental treatment suffered from an anormality that renders the data points obtained from these particular samples detrimental to our purpose.

### Heatmap of the count matrix
```R
library("pheatmap")
select <- order(rowMeans(counts(dds,normalized=TRUE)),
                decreasing=TRUE)[1:20]
df <- as.data.frame(colData(dds)[,c("condition","type")])
pheatmap(assay(ntd)[select,], cluster_rows=FALSE, show_rownames=FALSE,
         cluster_cols=FALSE, annotation_col=df)

pheatmap(assay(vsd)[select,], cluster_rows=FALSE, show_rownames=FALSE,
         cluster_cols=FALSE, annotation_col=df)

pheatmap(assay(rld)[select,], cluster_rows=FALSE, show_rownames=FALSE,
         cluster_cols=FALSE, annotation_col=df)
```

### Heatmap of the sample-to-sample distances
Another use of the transformed data is sample clustering. Here, we apply the *dist* function to the transpose of the transformed count matrix to get sample-to-sample distances.
```R
sampleDists <- dist(t(assay(vsd)))
```
A heatmap of this distance matrix gives us an overview over similarities and dissimilarities between samples. We have to provide a hierarchical clustering `hc` to the heatmap function based on the sample distances, or else the heatmap function would calculate a clustering based on the distances between the rows/columns of the distance matrix.
```R
library("RColorBrewer")
sampleDistMatrix <- as.matrix(sampleDists)
rownames(sampleDistMatrix) <- paste(vsd$condition, vsd$type, sep="-")
colnames(sampleDistMatrix) <- NULL
colors <- colorRampPalette( rev(brewer.pal(9, "Blues")) )(255)
pheatmap(sampleDistMatrix,
         clustering_distance_rows=sampleDists,
         clustering_distance_cols=sampleDists,
         col=colors)
```
### Principal component plot of the samples
Related to the distance matrix is the PCA plot, which shows the samples in the 2D plane spanned by their first two principal components. This type of plot is useful for visualizing the overall effect of experimental covariates and batch effects.
```R
plotPCA(vsd, intgroup=c("condition", "type"))
```
It is also possible to customize the PCA plot using the ggplot function.
```R
pcaData <- plotPCA(vsd, intgroup=c("condition", "type"), returnData=TRUE)
percentVar <- round(100 * attr(pcaData, "percentVar"))
ggplot(pcaData, aes(PC1, PC2, color=condition, shape=type)) +
  geom_point(size=3) +
  xlab(paste0("PC1: ",percentVar[1],"% variance")) +
  ylab(paste0("PC2: ",percentVar[2],"% variance")) + 
  coord_fixed()
```

## Variations to the standard workflow
### Wald test individual steps
The function DESeq runs the following functions in order:
```R
dds <- estimateSizeFactors(dds)
dds <- estimateDispersions(dds)
dds <- nbinomWaldTest(dds)
```
### Contrasts
A contrast is a linear combination of estimated log2 fold changes, which can be used to test if differences between groups are equal to zero. The simplest use case for contrasts is an experimental design containing a factor with three levels, say A, B and C. Contrasts enable the user to generate results for all 3 possible differences: log2 fold change of B vs A, of C vs A, and of C vs B. The `contrast` argument of *results* function is used to extract test results of log2 fold changes of interest, for example:
```R
results(dds, contrast=c("condition","C","B"))
```
Log2 fold changes can also be added and subtracted by providing a `list` to the `contrast` argument which has two elements: the names of the log2 fold changes to add, and the names of the log2 fold changes to subtract. The names used in the list should come from `resultsNames(dds)`. Alternatively, a numeric vector of the length of `resultsNames(dds)` can be provided, for manually specifying the linear combination of terms. Demonstrations of the use of contrasts for various designs can be found in the examples section of the help page `?results`. The mathematical formula that is used to generate the contrasts can be found below.

### Interactions
Interaction terms can be added to the design formula, in order to test, for example, if the log2 fold change attributable to a given condition is different based on another factor, for example if the condition effect differs across genotype.

**Initial note:** Many users begin to add interaction terms to the design formula, when in fact a much simpler approach would give all the results tables that are desired. We will explain this approach first, because it is much simpler to perform. If the comparisons of interest are, for example, the effect of a condition for different sets of samples, a simpler approach than adding interaction terms explicitly to the design formula is to perform the following steps:
- combine the factors of interest into a single factor with all combinations of the original factors
- change the design to include just this factor, e.g. ~ group

Using this design is similar to adding an interaction term, in that it models multiple condition effects which can be easily extracted with *results*. Suppose we have two factors genotype (with values I, II, and III) and condition (with values A and B), and we want to extract the condition effect specifically for each genotype. We could use the following approach to obtain, e.g. the condition effect for genotype I:
```R
dds$group <- factor(paste0(dds$genotype, dds$condition))
design(dds) <- ~ group
dds <- DESeq(dds)
resultsNames(dds)
results(dds, contrast=c("group", "IB", "IA"))
```
**Adding interactions to the design:** The following two plots diagram genotype-specific condition effects, which could be modeled with interaction terms by using a design of `~genotype + condition + genotype:condition`.

The key point to remember about designs with interaction terms is that, unlike for a design `~genotype + condition`, where the condition effect represents the **overall** effect controlling for differences due to genotype, by adding `genotype:condition`, the main condition effect only represents the effect of condition for the reference level of genotype (I, or whichever level was defined by the user as the reference level). The interaction terms `genotypeII.conditionB` and `genotypeIII.conditionB` give the difference between the condition effect for a given genotype and the condition effect for the reference genotype.

This genotype-condition interaction example is examined in further detail in Example 3 in the help page for results, which can be found by typing `?results`. In particular, we show how to test for differences in the condition effect across genotype, and we show how to obtain the condition effect for non-reference genotypes.

### Time-series experiments
There are a number of ways to analyze time-series experiments, depending on the biological question of interest. In order to test for any differences over multiple time points, once can use a design including the time factor, and then test using the likelihood ratio test as described in the following section, where the time factor is removed in the reduced formula. For a control and treatment time series, one can use a design formula containing the condition factor, the time factor, and the interaction of the two. In this case, using the likelihood ratio test with a reduced model which does not contain the interaction terms will test whether the condition induces a change in gene expression at any time point after the reference level time point (time 0). An example of the later analysis is provided in our RNA-seq workflow.

### Likelihood ratio test
DESeq2 offers two kinds of hypothesis tests: <font color=red>the Wald test</font>, where we use the estimated standard error of a log2 fold change to test if it is equal to zero, and <font color=red>the likelihood ratio test (LRT)</font>. The LRT examines two models for the counts, a *full* model with a certain number of terms and a *reduced* model, in which some of the terms of the *full* model are removed. The test determines if the increased likelihood of the data using the extra terms in the full model is more than expected if those extra terms are truly zero.

The LRT is therefore useful for testing multiple terms at once, for example testing 3 or more levels of a factor at once, or all interactions between two variables. The LRT for count data is conceptually similar to an analysis of variance (ANOVA) calculation in linear regression, except that in the case of the Negative Binomial GLM, we use an analysis of deviance (ANODEV), where the *deviance* captures the difference in likelihood between a full and a reduced model.

The likelihood ratio test can be performed by specifying `test="LRT"` when using the *DESeq* function, and providing a reduced design formula, e.g. one in which a number of terms from `design(dds)` are removed. The degrees of freedom for the test is obtained from the difference between the number of parameters in the two models. A simple likelihood ratio test, if the full design was `~condition` would look like:
```R
dds <- DESeq(dds, test="LRT", reduced=~1)
res <- results(dds)
```
If the full design contained other variables, such as a batch variable, e.g. `~batch + condition` then the likelihood ratio test would look like:
```R
dds <- DESeq(dds, test="LRT", reduced=~batch)
res <- results(dds)
```
### Extended section on shrinkage estimators
Here we extend the discussion of shrinkage estimators. To repeat, the current default method in `lfcShrink` is `normal`, although it will likely change in the October 2018 release to `apeglm`, as this method has improved performance relative to the original DESeq2 estimator, as does `ashr` in some of our benchmarks. Below is a summary table of differences between methods available in `lfcShrink` via the `type` argument (and for further technical reference on use of arguments please see `?lfcShrink`):
method:	| `normal`$^1$ | `apeglm`$^2$ |	`ashr`$^3$ 
|:---:|:---:|:---:|:---:|
Good for ranking by LFC | ✓ | ✓	| ✓
Preserves size of large LFC	|	| ✓ | ✓
Can compute *s-values* (Stephens 2016) |  |	✓ |	✓
Allows use of `coef` | ✓ | ✓ | ✓
Allows use of `lfcThreshold` | ✓ | ✓ |
Allows use of `contrast` | ✓ |  | ✓
Can shrink interaction terms|  | ✓ | ✓
**References:** 1. Love, Huber, and Anders (2014); 2. Zhu, Ibrahim, and Love (2018); 3. Stephens (2016)
```R
resApeT <- lfcShrink(dds, coef=2, type="apeglm", lfcThreshold=1) # Error
plotMA(resApeT, ylim=c(-3,3), cex=.8)
abline(h=c(-1,1), col="dodgerblue", lwd=2)
```
Examples:
```R
# Three groups:
condition <- factor(rep(c("A","B","C"),each=2))
model.matrix(~ condition)
# to compare C vs B, make B the reference level, and select the last coefficient
condition <- relevel(condition, "B")
model.matrix(~ condition)

# Three groups, compare condition effects:
grp <- factor(rep(1:3,each=4))
cnd <- factor(rep(rep(c("A","B"),each=2),3))
model.matrix(~ grp + cnd + grp:cnd)
# to compare condition effect in group 3 vs 2, make group 2 the reference level, and select the last coefficient
grp <- relevel(grp, "2")
model.matrix(~ grp + cnd + grp:cnd)

# Two groups, two individuals per group, compare within-individual condition effects:
grp <- factor(rep(1:2,each=4))
ind <- factor(rep(rep(1:2,each=2),2))
cnd <- factor(rep(c("A","B"),4))
model.matrix(~grp + grp:ind + grp:cnd)
# to compare condition effect across group, add a main effect for 'cnd', and select the last coefficient
model.matrix(~grp + cnd + grp:ind + grp:cnd)
```
### Recommendations for single-cell analysis
Van den Berge et al. 2018

### Approach to count outliers
RNA-seq data sometimes contain isolated instances of very large counts that are apparently unrelated to the experimental or study design, and which may be considered outliers. There are many reasons why outliers can arise, including rare technical or experimental artifacts, read mapping problems in the case of genetically differing samples, and genuine, but rare biological events. In many cases, users appear primarily interested in genes that show a consistent behavior, and this is the reason why by default, genes that are affected by such outliers are set aside by DESeq2, or if there are sufficient samples, outlier counts are replaced for model fitting. These two behaviors are described below.

The *DESeq* function calculates, for every gene and for every sample, a diagnostic test for outliers called Cook’s distance. Cook’s distance is a measure of how much a single sample is influencing the fitted coefficients for a gene, and a large value of Cook’s distance is intended to indicate an outlier count. The Cook’s distances are stored as a matrix available in `assays(dds)[["cooks"]]`.

The *results* function automatically flags genes which contain a Cook’s distance above a cutoff for samples which have 3 or more replicates. The p values and adjusted p values for these genes are set to `NA`. At least 3 replicates are required for flagging, as it is difficult to judge which sample might be an outlier with only 2 replicates. This filtering can be turned off with `results(dds, cooksCutoff=FALSE)`.

With many degrees of freedom – i.,e., many more samples than number of parameters to be estimated – it is undesirable to remove entire genes from the analysis just because their data include a single count outlier. When there are 7 or more replicates for a given sample, the DESeq function will automatically replace counts with large Cook’s distance with the trimmed mean over all samples, scaled up by the size factor or normalization factor for that sample. This approach is conservative, it will not lead to false positives, as it replaces the outlier value with the value predicted by the null hypothesis. This outlier replacement only occurs when there are 7 or more replicates, and can be turned off with `DESeq(dds, minReplicatesForReplace=Inf)`.

The default Cook’s distance cutoff for the two behaviors described above depends on the sample size and number of parameters to be estimated. The default is to use the 99% quantile of the F(p,m-p) distribution (with p the number of parameters including the intercept and m number of samples). The default for gene flagging can be modified using the `cooksCutoff` argument to the *results* function. For outlier replacement, DESeq preserves the original counts in `counts(dds)` saving the replacement counts as a matrix named `replaceCounts` in `assays(dds)`. Note that with continuous variables in the design, outlier detection and replacement is not automatically performed, as our current methods involve a robust estimation of within-group variance which does not extend easily to continuous covariates. However, users can examine the Cook’s distances in `assays(dds)[["cooks"]]`, in order to perform manual visualization and filtering if necessary.

**Note on many outliers:** if there are very many outliers (e.g. many hundreds or thousands) reported by `summary(res)`, one might consider further exploration to see if a single sample or a few samples should be removed due to low quality. The automatic outlier filtering/replacement is most useful in situations which the number of outliers is limited. When there are thousands of reported outliers, it might make more sense to turn off the outlier filtering/replacement (*DESeq* with `minReplicatesForReplace=Inf` and *results* with `cooksCutoff=FALSE`) and perform manual inspection: First it would be advantageous to make a PCA plot as described above to spot individual sample outliers; Second, one can make a boxplot of the Cook’s distances to see if one sample is consistently higher than others (here this is not the case):
```R
par(mar=c(8,5,2,2))
boxplot(log10(assays(dds)[["cooks"]]), range=0, las=2)
```
### Dispersion plot and fitting alternatives
Plotting the dispersion estimates is a useful diagnostic. The dispersion plot below is typical, with the final estimates shrunk from the gene-wise estimates towards the fitted estimates. Some gene-wise estimates are flagged as outliers and not shrunk towards the fitted value, (this outlier detection is described in the manual page for *estimateDispersionsMAP*). The amount of shrinkage can be more or less than seen here, depending on the sample size, the number of coefficients, the row mean and the variability of the gene-wise estimates.
```R
plotDispEsts(dds)
```
#### Local or mean dispersion fit
A local smoothed dispersion fit is automatically substitited in the case that the parametric curve doesn’t fit the observed dispersion mean relationship. This can be prespecified by providing the argument `fitType="local"` to either *DESeq* or *estimateDispersions*. Additionally, using the mean of gene-wise disperion estimates as the fitted value can be specified by providing the argument `fitType="mean"`.

#### Supply a custom dispersion fit
Any fitted values can be provided during dispersion estimation, using the lower-level functions described in the manual page for *estimateDispersionsGeneEst*. In the code chunk below, we store the gene-wise estimates which were already calculated and saved in the metadata column `dispGeneEst`. Then we calculate the median value of the dispersion estimates above a threshold, and save these values as the fitted dispersions, using the replacement function for *dispersionFunction*. In the last line, the function *estimateDispersionsMAP*, uses the fitted dispersions to generate maximum a posteriori (MAP) estimates of dispersion.
```R
ddsCustom <- dds
useForMedian <- mcols(ddsCustom)$dispGeneEst > 1e-7
medianDisp <- median(mcols(ddsCustom)$dispGeneEst[useForMedian],
                     na.rm=TRUE)
dispersionFunction(ddsCustom) <- function(mu) medianDisp
ddsCustom <- estimateDispersionsMAP(ddsCustom)
```

### Independent filtering of results
The *results* function of the DESeq2 package performs independent filtering by default using the mean of normalized counts as a filter statistic. A threshold on the filter statistic is found which optimizes the number of adjusted p values lower than a significance level `alpha` (we use the standard variable name for significance level, though it is unrelated to the dispersion parameter α). The theory behind independent filtering is discussed in greater detail below. The adjusted p values for the genes which do not pass the filter threshold are set to `NA`.

The default independent filtering is performed using the *filtered_p* function of the [genefilter](http://bioconductor.org/packages/genefilter) package, and all of the arguments of *filtered_p* can be passed to the *results* function. The filter threshold value and the number of rejections at each quantile of the filter statistic are available as metadata of the object returned by *results*.

For example, we can visualize the optimization by plotting the `filterNumRej` attribute of the results object. The *results* function maximizes the number of rejections (adjusted p value less than a significance level), over the quantiles of a filter statistic (the mean of normalized counts). The threshold chosen (vertical line) is the lowest quantile of the filter for which the number of rejections is <font color=red>within 1 residual standard deviation</font> to the peak of a curve fit to the number of rejections over the filter quantiles:
```R
metadata(res)$alpha
metadata(res)$filterThreshold
plot(metadata(res)$filterNumRej, 
     type="b", ylab="number of rejections",
     xlab="quantiles of filter")
lines(metadata(res)$lo.fit, col="red")
abline(v=metadata(res)$filterTheta)
```
Independent filtering can be turned off by setting `independentFiltering` to `FALSE`.
```R
resNoFilt <- results(dds, independentFiltering=FALSE)
addmargins(table(filtering=(res$padj < .1),
                 noFiltering=(resNoFilt$padj < .1)))
```

### Tests of log2 fold change above or below a threshold
It is also possible to provide thresholds for constructing Wald tests of significance. Two arguments to the results function allow for threshold-based Wald tests: `lfcThreshold`, which takes a numeric of a non-negative threshold value, and `altHypothesis`, which specifies the kind of test. Note that the alternative hypothesis is specified by the user, i.e. those genes which the user is interested in finding, and the test provides p values for the null hypothesis, the complement of the set defined by the alternative. The `altHypothesis` argument can take one of the following four values, where β is the log2 fold change specified by the `name` argument, and x is the `lfcThreshold`.
- `greaterAbs` - |β|>x - tests are two-tailed
- `lessAbs` - |β|<x - p values are the maximum of the upper and lower tests
- `greater` - β>x
- `less` - β<−x

The four possible values of `altHypothesis` are demonstrated in the following code and visually by MA-plots in the following figures.
```R
par(mfrow=c(2,2),mar=c(2,2,1,1))
ylim <- c(-2.5,2.5)
resGA <- results(dds, lfcThreshold=.5, altHypothesis="greaterAbs")
resLA <- results(dds, lfcThreshold=.5, altHypothesis="lessAbs")
resG <- results(dds, lfcThreshold=.5, altHypothesis="greater")
resL <- results(dds, lfcThreshold=.5, altHypothesis="less")
drawLines <- function() abline(h=c(-.5,.5),col="dodgerblue",lwd=2)
plotMA(resGA, ylim=ylim); drawLines()
plotMA(resLA, ylim=ylim); drawLines()
plotMA(resG, ylim=ylim); drawLines()
plotMA(resL, ylim=ylim); drawLines()
```

### Access to all calculated values
All row-wise calculated values (intermediate dispersion calculations, coefficients, standard errors, etc.) are stored in the *DESeqDataSet* object, e.g. `dds` in this vignette. These values are accessible by calling *mcols* on `dds`. Descriptions of the columns are accessible by two calls to *mcols*. Note that the call to `substr` below is only for display purposes.
```R
mcols(dds,use.names=TRUE)[1:4,1:4]
substr(names(mcols(dds)),1,10)
mcols(mcols(dds), use.names=TRUE)[1:4,]
```
The mean values $μ_{ij}=s_jq_{ij} and the Cook’s distances for each gene and sample are stored as matrices in the assays slot:
```R
head(assays(dds)[["mu"]])
head(assays(dds)[["cooks"]])
```
The dispersions $α_i$ can be accessed with the dispersions function.
```R
head(dispersions(dds))
head(mcols(dds)$dispersion)
```
The size factors $s_j$ are accessible via sizeFactors:
```R
sizeFactors(dds)
```
For advanced users, we also include a convenience function `coef `for extracting the matrix $[β_{ir}]$ for all genes i and model coefficients r. This function can also return a matrix of standard errors, see `?coef`. The columns of this matrix correspond to the effects returned by *resultsNames*. Note that the *results* function is best for building results tables with p values and adjusted p values.
```R
head(coef(dds))
```
The beta prior variance $σ^2_r$ is stored as an attribute of the DESeqDataSet:
```R
attr(dds, "betaPriorVar")
```
General information about the prior used for log fold change shrinkage is also stored in a slot of the DESeqResults object. This would also contain information about what other packages were used for log2 fold change shrinkage.
```R
priorInfo(resLFC)
priorInfo(resNorm)
priorInfo(resAsh)
```
The dispersion prior variance $σ^2_d$ is stored as an attribute of the dispersion function:
```R
dispersionFunction(dds)
attr(dispersionFunction(dds), "dispPriorVar")
```
The version of DESeq2 which was used to construct the DESeqDataSet object, or the version used when DESeq was run, is stored here:
```R
metadata(dds)[["version"]]
```





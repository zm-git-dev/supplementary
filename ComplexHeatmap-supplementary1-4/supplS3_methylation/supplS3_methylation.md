
Supplementary S3. Correlations between methylation, expression and other genomic features
==============================================

**Author**: Zuguang Gu ( z.gu@dkfz.de )

**Date**: 2016-05-13

----------------------------------------



<style type="text/css">
h1 {
    line-height: 120%;
}
</style>

To successfully run this example, **ComplexHeatmap** >= 1.10.1. is required.
The newest version can be obtained by:


```r
library(devtools)
install_github("jokergoo/Complexheatmap")
```


Load all the packages.


```r
library(ComplexHeatmap)
library(circlize)
library(RColorBrewer)
```

In the following example, data is randomly generated based on patterns found in an unpublished analysis.
The code for generating the random data can be found at the end of this supplementary.


```r
res_list = readRDS("meth.rds")
type = res_list$type
mat_meth = res_list$mat_meth
mat_expr = res_list$mat_expr
direction = res_list$direction
cor_pvalue = res_list$cor_pvalue
gene_type = res_list$gene_type
anno_gene = res_list$anno_gene
dist = res_list$dist
anno_enhancer = res_list$anno_enhancer
```

The different sources of information and corresponding variables are:

1. `type`: the label which shows whether the sample is tumor or normal.
2. `mat_meth`: a matrix in which rows correspond to differetially methylated regions (DMRs).
    The value in the matrix is the mean methylation level in the DMR in every sample.
3. `mat_expr`: a matrix in which rows correspond to genes which are associated to the DMRs (i.e. the nearest gene to the DMR).
    The value in the matrix is the expression level for each gene in each sample. Expression is scaled for every gene across samples.
4. `direction`: direction of the methylation change (hyper meaning higher methylation in tumor samples, hypo means lower methylation in tumor samples).
5. `cor_pvalue`: p-value for the correlation test between methylation and expression of the associated gene.
6. `gene_type`: type of the genes (e.g. protein coding genes or lincRNAs).
7. `anno_gene`: annotation to the gene models (intergenic, intragenic or TSS).
8. `dist`: distance from DMRs to TSS of the assiciated genes.
9. `anno_enhancer`: fraction of the DMR that overlaps enhancers.

The data only includes DMRs for which methylation and expression of the associated gene are negatively correlated. 

The clustering of columns for the methylation matrix are calculated first 
so that columns in the expression matrix can be adjusted to have the same column order as in the methylation matrix.


```r
column_tree = hclust(dist(t(mat_meth)))
```

We first define two column annotations and then make the complex heatmaps.


```r
ht_global_opt(heatmap_legend_title_gp = gpar(fontsize = 8, fontface = "bold"), 
	heatmap_legend_labels_gp = gpar(fontsize = 8), heatmap_column_names_gp = gpar(fontsize = 8))

ha = HeatmapAnnotation(df = data.frame(type = type), 
    col = list(type = c("Tumor" = "pink", "Control" = "royalblue")))
ha2 = HeatmapAnnotation(df = data.frame(type = type), 
    col = list(type = c("Tumor" = "pink", "Control" = "royalblue")), 
	show_legend = FALSE)

ht_list = Heatmap(mat_meth, name = "methylation", col = colorRamp2(c(0, 0.5, 1), c("blue", "white", "red")),
	cluster_columns = column_tree, column_dend_reorder = FALSE, 
	top_annotation = ha, km = 5, column_title = "Methylation", column_title_gp = gpar(fontsize = 10), 
	row_title_gp = gpar(fontsize = 10)) +
	Heatmap(direction, name = "direction", col = c("hyper" = "red", "hypo" = "blue")) +
	Heatmap(mat_expr[, column_tree$order], name = "expression", 
        col = colorRamp2(c(-2, 0, 2), c("green", "white", "red")), cluster_columns = FALSE, 
        top_annotation = ha2, column_title = "Expression", column_title_gp = gpar(fontsize = 10)) +
	Heatmap(cor_pvalue, name = "-log10(cor_p)", col = colorRamp2(c(0, 2, 4), c("white", "white", "red"))) +
	Heatmap(gene_type, name = "gene type", col = structure(brewer.pal(length(unique(gene_type)), "Set3"), 
        names = unique(gene_type))) +
	Heatmap(anno_gene, name = "anno_gene", col = structure(brewer.pal(length(unique(anno_gene)), "Set1"), 
        names = unique(anno_gene))) +
	Heatmap(dist, name = "dist_tss", col = colorRamp2(c(0, 10000), c("black", "white"))) +
	Heatmap(anno_enhancer, name = "anno_enhancer", col = colorRamp2(c(0, 1), c("white", "orange")), 
        cluster_columns = FALSE, column_title = "Enhancer", column_title_gp = gpar(fontsize = 10))

draw(ht_list, newpage = FALSE, column_title = "Comprehensive correspondence between methylation, expression and other genomic features", 
    column_title_gp = gpar(fontsize = 12, fontface = "bold"), heatmap_legend_side = "bottom")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

```r
ht_global_opt(RESET = TRUE)
```
The complex heatmaps reveal that highly methylated DMRs are enriched in intergenic and intragenic regions and rarely overlap with enhancers. In contrast, lowly methylated DMRs are enriched for transcription start sites (TSS) and enhancers.

The following code generates the random data in `meth.rds`.


```r
type = c(rep("Tumor", 10), rep("Control", 10))

set.seed(888)

######################################
# generate methylation matrix
rand_meth = function(k, mean) {
    (runif(k) - 0.5)*min(c(1-mean), mean) + mean
}

mean_meth = c(rand_meth(300, 0.3), rand_meth(700, 0.7))
mat_meth = as.data.frame(lapply(mean_meth, function(m) {
    if(m < 0.3) {
        c(rand_meth(10, m), rand_meth(10, m + 0.2))
    } else if(m > 0.7) {
        c(rand_meth(10, m), rand_meth(10, m - 0.2))
    } else {
        c(rand_meth(10, m), rand_meth(10, m + sample(c(1, -1), 1)*0.2))
    }

}))
mat_meth = t(mat_meth)
rownames(mat_meth) = NULL
colnames(mat_meth) = paste0("sample", 1:20)

######################################
# generate directions for methylation
direction = rowMeans(mat_meth[, 1:10]) - rowMeans(mat_meth[, 11:20])
direction = ifelse(direction > 0, "hyper", "hypo")

#######################################
# generate expression matrix
mat_expr = t(apply(mat_meth, 1, function(x) {
    x = x + rnorm(length(x), sd = (runif(1)-0.5)*0.4 + 0.5)
    -scale(x)
}))
dimnames(mat_expr) = dimnames(mat_meth)

#############################################################
# matrix for correlation between methylation and expression
cor_pvalue = -log10(sapply(seq_len(nrow(mat_meth)), function(i) {
    cor.test(mat_meth[i, ], mat_expr[i, ])$p.value
}))

#####################################################
# matrix for types of genes
gene_type = sample(c("protein_coding", "lincRNA", "microRNA", "psedo-gene", "others"), 
    nrow(mat_meth), replace = TRUE, prob = c(6, 1, 1, 1, 1))

#################################################
# annotation to genes
anno_gene = sapply(mean_meth, function(m) {
    if(m > 0.6) {
        if(runif(1) < 0.8) return("intragenic")
    }
    if(m < 0.4) {
        if(runif(1) < 0.4) return("TSS")
    }
    return("intergenic")
})

############################################
# distance to genes
dist = sapply(mean_meth, function(m) {
    if(m < 0.6) {
        if(runif(1) < 0.8) return(round( (runif(1)-0.5)*1000000 + 500000 ))
    }
    if(m < 0.3) {
        if(runif(1) < 0.4) return(round( (runif(1) - 0.5)*1000 + 500))
    }
    return(round( (runif(1) - 0.5)*100000 + 50000))
})


#######################################
# annotation to enhancers
rand_enhancer = function(m) {
    if(m < 0.4) {
        if(runif(1) < 0.6) return(runif(1))
    } else if (runif(1) < 0.1) {
        return(runif(1))
    } 
    return(0)
}
anno_enhancer_1 = sapply(mean_meth, rand_enhancer)
anno_enhancer_2 = sapply(mean_meth, rand_enhancer)
anno_enhancer_3 = sapply(mean_meth, rand_enhancer)
anno_enhancer = data.frame(enhancer_1 = anno_enhancer_1, enhancer_2 = anno_enhancer_2, enhancer_3 = anno_enhancer_3)

#################################
# put everything into one object
res_list = list()
res_list$type = type
res_list$mat_meth = mat_meth
res_list$mat_expr = mat_expr
res_list$direction = direction
res_list$cor_pvalue = cor_pvalue
res_list$gene_type = gene_type
res_list$anno_gene = anno_gene
res_list$dist = dist
res_list$anno_enhancer = anno_enhancer

saveRDS(res_list, file = "meth.rds")
```

## Session info


```r
sessionInfo()
```

```
## R version 3.2.3 (2015-12-10)
## Platform: x86_64-apple-darwin13.4.0 (64-bit)
## Running under: OS X 10.11.4 (El Capitan)
## 
## locale:
## [1] C/en_US.UTF-8/C/C/C/C
## 
## attached base packages:
## [1] methods   grid      stats     graphics  grDevices utils     datasets 
## [8] base     
## 
## other attached packages:
## [1] RColorBrewer_1.1-2    circlize_0.3.7        ComplexHeatmap_1.10.1
## 
## loaded via a namespace (and not attached):
##  [1] dendextend_1.1.8     formatR_1.4          magrittr_1.5        
##  [4] evaluate_0.9         stringi_1.0-1        GlobalOptions_0.0.10
##  [7] whisker_0.3-2        GetoptLong_0.1.3     rjson_0.2.15        
## [10] tools_3.2.3          stringr_1.0.0        colorspace_1.2-6    
## [13] shape_1.4.2          knitr_1.13
```

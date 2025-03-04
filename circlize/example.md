


```r
library(circlize)
set.seed(12345)
```

## Figure 1A


```r
par(mar = c(1, 1, 1, 1))
circos.initializeWithIdeogram()

bed = generateRandomBed(nr = 200, nc = 4)
circos.genomicPosTransformLines(bed, posTransform = posTransform.default, horizontalLine = "top")
om = circos.par("track.margin")
oc = circos.par("cell.padding")
circos.par(track.margin = c(om[1], 0), cell.padding = c(0, 0, 0, 0))
f = colorRamp2(breaks = c(-1, 0, 1), colors = c("blue", "white", "red"))
circos.genomicTrackPlotRegion(bed, stack = TRUE, panel.fun = function(region, value, ...) {
	circos.genomicRect(region, value, col = f(value[[1]]), 
		border = f(value[[1]]), lwd = 0.1, posTransform = posTransform.default, ...)
}, bg.border = NA, track.height = 0.1)
circos.par(track.margin = om, cell.padding = oc)

bed = generateRandomBed(nr = 500, fun = function(k) runif(k)*sample(c(-1, 1), k, replace = TRUE))
circos.genomicTrackPlotRegion(bed, ylim = c(-1, 1), panel.fun = function(region, value, ...) {
	col = ifelse(value[[1]] > 0, "red", "green")
	circos.genomicPoints(region, value, col = col, cex = 0.5, pch = 16)
	cell.xlim = get.cell.meta.data("cell.xlim")
	for(h in c(-1, -0.5, 0, 0.5, 1)) {
		circos.lines(cell.xlim, c(h, h), col = "#00000040")
	}
}, track.height = 0.1)

bed = generateRandomBed(nr = 500, fun = function(k) rnorm(k, 0, 50))
circos.genomicTrackPlotRegion(bed, panel.fun = function(region, value, ...) {
	x = (region[[2]] + region[[1]]) / 2
	y = value[[1]]
	loess.fit = loess(y ~ x)
    loess.predict = predict(loess.fit, x, se = TRUE)
    d1 = c(x, rev(x))
    d2 = c(loess.predict$fit + loess.predict$se.fit, rev(loess.predict$fit - loess.predict$se.fit))
    circos.polygon(d1, d2, col = "#CCCCCC", border = NA)
    circos.points(x, y, pch = 16, cex = 0.5)
    circos.lines(x, loess.predict$fit)
}, track.height = 0.1)


bed_list = list(generateRandomBed(nr = 500, fun = function(k) runif(k)),
                generateRandomBed(nr = 500, fun = function(k) runif(k)))
col = c("#FF000040", "#0000FF40")
circos.genomicTrackPlotRegion(bed_list, ylim = c(-1, 1), panel.fun = function(region, value, ...) {
	i = getI(...)
	if(i == 1) {
		circos.genomicLines(region, value, area = TRUE, area.baseline = 0, col = "orange", border = NA, ...)
	} else {
		circos.genomicLines(region, -value, area = TRUE, area.baseline = 0, col = "yellow", border = NA, ...)
	}
}, track.height = 0.1)

region1 = generateRandomBed(nr = 1000); region1 = region1[sample(nrow(region1), 20), ]
region2 = generateRandomBed(nr = 1000); region2 = region2[sample(nrow(region2), 20), ]
circos.genomicLink(region1, region2, col = sample(10, 20, replace = TRUE))

highlight.chromosome("chr1")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png)

```r
circos.clear()
```

## Figure 1B



```r
col = c('#E41A1C', '#A73C52', '#6B5F88', '#3780B3', '#3F918C', '#47A266',
    '#53A651', '#6D8470', '#87638F', '#A5548D', '#C96555', '#ED761C',
    '#FF9508', '#FFC11A', '#FFEE2C', '#EBDA30', '#CC9F2C', '#AD6428',
    '#BB614F', '#D77083', '#F37FB8', '#DA88B3', '#B990A6', '#999999')

par(mar = c(1, 1, 1, 1))
circos.initializeWithIdeogram(plotType = NULL)

circos.trackPlotRegion(ylim = c(0, 1), bg.border = NA, track.height = 0.05,
    panel.fun = function(x, y) {
        xlim = get.cell.meta.data("xlim")
        chr = get.cell.meta.data("sector.index")
        circos.text(mean(xlim), 0.5, labels = chr, facing = "clockwise", niceFacing = TRUE)
    })

bed = generateRandomBed(nr = 200, fun = function(k) runif(k))
circos.genomicTrackPlotRegion(bed, bg.border = NA, panel.fun = function(region, value, ...) {
    i = get.cell.meta.data("sector.numeric.index")
    circos.genomicLines(region, value, area = TRUE, border = NA, baseline = 0, col = col[i])
}, track.height = 0.1)

circos.trackPlotRegion(ylim = c(0, 1), bg.border = col, bg.col = col, panel.fun = function(x, y) {
    chr = get.cell.meta.data("sector.index")
    circos.axis(h = "bottom", labels = NULL, sector.index = chr, direction = "inside", major.tick.percentage = 0.2)
}, track.height = 0.05)

region1 = data.frame(chr = c("chr1", "chr1"),
                     start = c(12345678, 22222222),
                     end = c(12345678, 22222222))
region2 = data.frame(chr = c("chr1", "chr1"),
                     start = c(87654321, 99999999),
                     end = c(87654321, 99999999))                     
circos.genomicLink(region1, region2, h = 0.2)

circos.clear()

par(mar = c(1, 1, 1, 1), new = TRUE)
    
circos.par("canvas.xlim" = c(-2, 2), "canvas.ylim" = c(-2, 2), clock.wise = FALSE,
    cell.padding = c(0, 0, 0, 0), gap.degree = 180)
circos.initializeWithIdeogram(chromosome.index = "chr1", plotType = c("ideogram", "axis"))

text(0, 0.6, "chr1")

circos.genomicLink(region1, region2, h = 0.5)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png)

```r
circos.clear()       
```

## Figure 1C



```r
cytoband = read.cytoband()
df = cytoband$df
chromosome = cytoband$chromosome
chr.len = cytoband$chr.len

df_zoom = df[df[[1]] %in% c("chr7", "chr8"), ]
df_zoom[[1]] = paste0(df_zoom[[1]], "_zoom")
df = rbind(df, df_zoom)

bed = generateRandomBed(nr = 1000)
bed_zoom = bed[bed[[1]] %in% c("chr7", "chr8"), ]
bed_zoom[[1]] = paste0(bed_zoom[[1]], "_zoom")
bed = rbind(bed, bed_zoom)

par(mar = c(1, 1, 1, 1))
circos.par(start.degree = 90)
circos.initializeWithIdeogram(df, sort.chr = FALSE, sector.width = c(chr.len/sum(chr.len), 0.5, 0.5))
circos.genomicTrackPlotRegion(bed, panel.fun = function(region, value, ...) {
	circos.genomicPoints(region, value, pch = 16, cex = 0.8)
})

circos.link("chr7", get.cell.meta.data("cell.xlim", sector.index = "chr7"),
            "chr7_zoom", get.cell.meta.data("cell.xlim", sector.index = "chr7_zoom"), 
			col = "#0000FF10", border = NA)
circos.link("chr8", get.cell.meta.data("cell.xlim", sector.index = "chr8"),
            "chr8_zoom", get.cell.meta.data("cell.xlim", sector.index = "chr8_zoom"), 
			col = "#FF000010", border = NA)
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png)

```r
circos.clear()       
```

## Figure 1D



```r
par(mar = c(1, 1, 1, 1))
    
circos.par("start.degree" = 150, "gap.degree" = c(240), "cell.padding" = c(0, 0, 0, 0))
circos.initializeWithIdeogram(chromosome.index = "chr1")
circos.trackPlotRegion(ylim = c(0, 1), bg.border = NA, track.height = 0.2)

		
chr.xlim = get.cell.meta.data("xlim", sector.index = "chr1")

for(h in c(0, 0.25, 0.5, 0.75, 1)) {
	circos.lines(chr.xlim, c(h, h), col = "white", sector.index = "chr1")
}

# just generate data
xrange = get.cell.meta.data("xrange", sector.index = "chr1")
x = seq(chr.xlim[1], chr.xlim[2], length = 1000)
y = rnorm(1000, 0.5, 0.05)
a1 = 0.2
a2 = a1 + 0.1
a3 = 0.6
a4 = a3 + 0.2
y[x > chr.xlim[1] + xrange*a1 & x < chr.xlim[1] + xrange*a2] = y[x > chr.xlim[1] + xrange*a1 & x < chr.xlim[1] + xrange*a2] + 0.3
y[x > chr.xlim[1] + xrange*a3 & x < chr.xlim[1] + xrange*a4] = y[x > chr.xlim[1] + xrange*a3 & x < chr.xlim[1] + xrange*a4] - 0.2
x = x[y >=0 & y<= 1]
y = y[y >=0 & y<= 1]

		
circos.points(x, y, col = "#00000040", sector.index = "chr1", pch = 16, cex = 0.7)

circos.rect(chr.xlim[1] + xrange*a1, 0, chr.xlim[1] + xrange*a2, 1, col = "#FF000020", border = "red", sector.index = "chr1")
circos.rect(chr.xlim[1] + xrange*a3, 0, chr.xlim[1] + xrange*a4, 1, col = "#00FF0020", border = "green", sector.index = "chr1")

circos.clear()


text(0, 0.4, "Treatment", cex = 2, font = 2)

par(new = TRUE)

circos.par("start.degree" = 150, "gap.degree" = c(240), "cell.padding" = c(0, 0, 0, 0),
	"canvas.ylim" = c(0, 2))
circos.initializeWithIdeogram(chromosome.index = "chr1")
circos.trackPlotRegion(ylim = c(0, 1), bg.border = NA, track.height = 0.2)

		
chr.xlim = get.cell.meta.data("xlim", sector.index = "chr1")

for(h in c(0, 0.25, 0.5, 0.75, 1)) {
	circos.lines(chr.xlim, c(h, h), col = "white", sector.index = "chr1")
}


xrange = get.cell.meta.data("xrange", sector.index = "chr1")
x = seq(chr.xlim[1], chr.xlim[2], length = 1000)
y = rnorm(1000, 0.5, 0.05)
x = x[y >=0 & y<= 1]
y = y[y >=0 & y<= 1]
		
circos.points(x, y, col = "#00000040", sector.index = "chr1", pch = 16, cex = 0.7)
circos.clear()

text(0, 0.4, "Control", cex = 2, font = 2)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png)

```r
par(new = FALSE)
```


## Figure 1E


```r
# dend: as dendrogram object, usually returned by hclust and as.dendrogram
# maxy: maximum height of the tree
circos.dendrogram = function(dend, maxy) {
  labels = as.character(labels(dend))
    x = seq_along(labels) - 0.5
    names(x) = labels

    is.leaf = function(object) (is.logical(L <- attr(object, "leaf"))) && L
	
	# recursive function to draw the tree
    draw.d = function(dend, maxy) {
        leaf = attr(dend, "leaf")
        d1 = dend[[1]]
        d2 = dend[[2]]
        height = attr(dend, 'height')
        midpoint = attr(dend, 'midpoint')

        if(is.leaf(d1)) {
            x1 = x[as.character(attr(d1, "label"))]
        } else {
            x1 = attr(d1, "midpoint") + x[as.character(labels(d1))[1]]
        }
        y1 = attr(d1, "height")

        if(is.leaf(d2)) {
            x2 = x[as.character(attr(d2, "label"))]
        } else {
            x2 = attr(d2, "midpoint") + x[as.character(labels(d2))[1]]
        }
        y2 = attr(d2, "height")

        circos.lines(c(x1, x1), maxy - c(y1, height), straight = TRUE)
        circos.lines(c(x1, x2), maxy - c(height, height))
        circos.lines(c(x2, x2), maxy - c(y2, height), straight = TRUE)

        if(!is.leaf(d1)) {
            draw.d(d1, maxy)
        }
        if(!is.leaf(d2)) {
            draw.d(d2, maxy)
        }
    }
    
    draw.d(dend, maxy)
}


mat = matrix(rnorm(100*10), nrow = 10, ncol = 100)
factors = rep(letters[1:2], 50)
par(mar = c(1, 1, 1, 1))
circos.par(cell.padding = c(0, 0, 0, 0), gap.degree = 5)
circos.initialize(factors, xlim = c(0, 50))
maxy = 0

f = colorRamp2(breaks = c(-1, 0, 1), colors = c("green", "black", "red"))

circos.trackPlotRegion(ylim = c(0, 10), bg.border = NA, panel.fun = function(x, y) {
  sector.index = get.cell.meta.data("sector.index")
    m = mat[, factors == sector.index]
    
    dend.col = as.dendrogram(hclust(dist(t(m))))

    maxy = ifelse(maxy > attr(dend.col, "height"), maxy, attr(dend.col, "height"))
    assign("maxy", maxy, envir = .GlobalEnv)

    m2 = m[, labels(dend.col)]
    nr = nrow(m2)
    nc = ncol(m2)
    for(i in 1:nr) {
        for(j in 1:nc) {
            circos.rect(j-1, nr-i, j, nr-i+1, border = f(m2[i, j]), col = f(m2[i, j]))
        }
    }
    
})
circos.trackPlotRegion(ylim = c(0, maxy), bg.border = NA, track.height = 0.3, panel.fun = function(x, y) {
    sector.index = get.cell.meta.data("sector.index")
    m = mat[, factors == sector.index]
    
    dend.col = as.dendrogram(hclust(dist(t(m))))

    circos.dendrogram(dend.col, maxy)
    
})

circos.clear()

x = seq(-10, 10, length.out=100)/40
col =f(seq(-2, 2, length.out = length(x-1)))
for(i in seq_along(x)) {
	if(i == 1) next
	rect(x[i-1], -0.05, x[i], 0.05, col = col[i], border = col[i])
}

text(x[1], -0.08, "-2", adj = c(0.5, 1), cex = 1.2)
text(x[ceiling(length(x)/2)], -0.08, "0", adj = c(0.5, 1), cex = 1.2)
text(x[length(x)], -0.08, "2", adj = c(0.5, 1), cex = 1.2)
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png)

## Figure 1F


```r
par(mar = c(1, 1, 1 ,1))
load("gencode_TP_gene.RData")
df = data.frame( names(gencode),
                 sapply(gencode, function(x) x$start ),
				 sapply(gencode, function(x) x$end ) )
circos.genomicInitialize(df, sector.name = sapply(gencode, function(x) x$name))
n = max(sapply(gencode, function(x) length(x$transcript)))

circos.genomicTrackPlotRegion(ylim = c(0, 1), bg.col = c("#FF000040", "#00FF0040", "#0000FF40"), bg.border = NA, track.height = 0.05)

circos.genomicTrackPlotRegion(ylim = c(0.5, n + 0.5), panel.fun = function(region, value, ...) {
	gi = get.cell.meta.data("sector.index")
	tr = gencode[[gi]]$transcript
	for(i in seq_along(tr)) {
		region = data.frame(sapply(tr[[i]]$exon, function(x) x$start),
		                    sapply(tr[[i]]$exon, function(x) x$end))
		circos.lines(c(tr[[i]]$start, tr[[i]]$end), c(n-i, n-i), col = "#CCCCCC")
		circos.genomicRect(region, ytop = n-i+0.4, ybottom = n-i-0.4, col = "orange", border = NA)
	}
}, bg.border = NA, track.height = 0.3)
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png)

```r
circos.clear()
```

## Session info


```r
sessionInfo()
```

```
## R version 3.2.3 (2015-12-10)
## Platform: x86_64-apple-darwin13.4.0 (64-bit)
## Running under: OS X 10.11.6 (El Capitan)
## 
## locale:
## [1] C/en_US.UTF-8/C/C/C/C
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] circlize_0.3.9 markdown_0.7.7 knitr_1.14     colorout_1.1-2
## 
## loaded via a namespace (and not attached):
##  [1] colorspace_1.2-6     magrittr_1.5         formatR_1.4         
##  [4] tools_3.2.3          GlobalOptions_0.0.10 stringi_1.1.1       
##  [7] grid_3.2.3           stringr_1.1.0        shape_1.4.2         
## [10] mime_0.5             evaluate_0.9
```

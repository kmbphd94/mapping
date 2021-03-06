---
title: "Estimated QTL effects"
teaching: 0
exercises: 0
questions:
- "How do I find the estimated effects of a QTL on a phenotype?"
objectives:
- Obtain estimated QTL effects.
- Plot estimated QTL effects.
keypoints:
- "."
- "."
source: Rmd
---



The `scan1()` function returns only LOD scores. To
obtain estimated QTL effects, use the function `scan1coef()`. This function takes a single phenotype and the genotype probabilities for a single chromosome and returns a matrix with the estimated coefficients at each putative QTL location along
the chromosome.

For example, to get the estimated effects on chromosome 2 for the liver phenotype, we'd do the following:


~~~
c2eff <- scan1coef(pr[,"2"], iron$pheno[,"liver"])
~~~
{: .r}

The result is a matrix,  positions &times;  genotypes. An attribute, `"map"` contains the positions of the calculations. To plot the effects, use the function `plot_coef()` from the [qtl2plot](https://github.com/rqtl/qtl2plot). There is again an S3 method function `plot.scan1coef()`, so one can just type `plot()`.
Use the argument `columns` to indicate which coefficient columns to plot.


~~~
par(mar=c(4.1, 4.1, 1.1, 2.6), las=1)
col <- c("slateblue", "violetred", "green3")
plot(c2eff, map["2"], columns=1:3, col=col)
last_coef <- unclass(c2eff)[nrow(c2eff),] # pull out last coefficients
for(i in seq(along=last_coef))
    axis(side=4, at=last_coef[i], names(last_coef)[i], tick=FALSE, col.axis=col[i])
~~~
{: .r}

The default is to provide phenotype averages for each genotype group. If instead you want additive and dominance effects, you can provide a square matrix of _contrasts_, as follows:


~~~
c2effB <- scan1coef(pr[,"2"], iron$pheno[,"liver"],
                    contrasts=cbind(mu=c(1,1,1), a=c(-1, 0, 1), d=c(-0.5, 1, -0.5)))
~~~
{: .r}

The result will then contain the estimates of `mu`, `a`, and `d`. Here's a plot of the additive and dominance effects, which are in the
second and third columns.


~~~
par(mar=c(4.1, 4.1, 1.1, 2.6), las=1)
plot(c2effB, map["2"], columns=2:3, col=col)
last_coef <- unclass(c2effB)[nrow(c2effB),2:3] # last two coefficients
for(i in seq(along=last_coef))
    axis(side=4, at=last_coef[i], names(last_coef)[i], tick=FALSE, col.axis=col[i])
~~~
{: .r}

If you provide a kinship matrix to `scan1coef()`, it fits a linear mixed model (LMM) to account for a residual polygenic effect. Here let's use the kinship matrix from the LOCO method.


~~~
c2eff_pg <- scan1coef(pr[,"2"], iron$pheno[,"liver"], kinship_loco[["2"]])
~~~
{: .r}

Here's a plot of the estimates.


~~~
par(mar=c(4.1, 4.1, 1.1, 2.6), las=1)
col <- c("slateblue", "violetred", "green3")
plot(c2eff_pg, map["2"], columns=1:3, col=col, ylab="Phenotype average")
last_coef <- unclass(c2eff_pg)[nrow(c2eff_pg),]
for(i in seq(along=last_coef))
    axis(side=4, at=last_coef[i], names(last_coef)[i], tick=FALSE, col.axis=col[i])
~~~
{: .r}

You can also get estimated additive and dominance effects, using a matrix of contrasts.


~~~
c2effB_pg <- scan1coef(pr[,"2"], iron$pheno[,"liver"], kinship_loco[["2"]],
                       contrasts=cbind(mu=c(1,1,1), a=c(-1, 0, 1), d=c(-0.5, 1, -0.5)))
~~~
{: .r}

Here's a plot of the results.


~~~
par(mar=c(4.1, 4.1, 1.1, 2.6), las=1)
plot(c2effB_pg, map["2"], columns=2:3, col=col)
last_coef <- unclass(c2effB_pg)[nrow(c2effB_pg),2:3]
for(i in seq(along=last_coef))
    axis(side=4, at=last_coef[i], names(last_coef)[i], tick=FALSE, col.axis=col[i])
~~~
{: .r}

Another option for estimating the QTL effects is to treat them as random effects and calculate Best Linear Unbiased Predictors (BLUPs). This is particularly valuable for multi-parent populations
such as the Collaborative Cross and Diversity Outbred mice, where the large number of possible genotypes at a QTL lead to considerable variability in the effect estimates. To calculate BLUPs, use `scan1blup()`; it takes the same arguments as `scan1coef()`, including
the option of a kinship matrix to account for a residual polygenic effect.


~~~
c2blup <- scan1blup(pr[,"2"], iron$pheno[,"liver"], kinship_loco[["2"]])
~~~
{: .r}

Here is a plot of the BLUPs (as dashed curves) alongside the standard estimates. Note that


~~~
par(mar=c(4.1, 4.1, 1.1, 2.6), las=1)
col <- c("slateblue", "violetred", "green3")
ylim <- range(c(c2blup, c2eff))+c(-1,1)
plot(c2eff, map["2"], columns=1:3, col=col, ylab="Phenotype average", ylim=ylim,
     xlab="Chr 2 position")
plot(c2blup, map["2"], columns=1:3, col=col, add=TRUE, lty=2)
last_coef <- unclass(c2eff)[nrow(c2eff),]
for(i in seq(along=last_coef))
    axis(side=4, at=last_coef[i], names(last_coef)[i], tick=FALSE, col.axis=col[i])
~~~
{: .r}

The `scan1coef` function can also provide estimated QTL effects for binary traits, with `model="binary"`. (However, `scan1blup` has not yet been implemented for binary traits.)


~~~
c2eff_bin <- scan1coef(pr[,"2"], bin_pheno[,"liver"], model="binary")
~~~
{: .r}

Here's a plot of the effects. They're a bit tricky to interpret, as their basically log odds ratios.


~~~
par(mar=c(4.1, 4.1, 1.1, 2.6), las=1)
col <- c("slateblue", "violetred", "green3")
plot(c2eff_bin, map["2"], columns=1:3, col=col)
last_coef <- unclass(c2eff_bin)[nrow(c2eff_bin),] # pull out last coefficients
for(i in seq(along=last_coef))
    axis(side=4, at=last_coef[i], names(last_coef)[i], tick=FALSE, col.axis=col[i])
~~~
{: .r}

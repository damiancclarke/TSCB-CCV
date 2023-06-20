![Stata](https://img.shields.io/badge/stata-2013-green) ![GitHub Starts](https://img.shields.io/github/stars/Daniel-Pailanir/tscb-ccv?style=social) ![GitHub license](https://img.shields.io/github/license/Daniel-Pailanir/sdid)

# Two-Stage Cluster Bootstrap and Causal Cluster Variance for Stata
This repository contains a Stata implementation of the Two-Stage Cluster Bootstrap (TSCB) estimator and the Causal Cluster Variance (CCV) estimator described in  [Abadie et al (2023)](#references).  These programs return standard errors for regression analysis of some outcome on a treatment of interest using either simple OLS, or fixed effects models, while accounting for clustering by group.  Unlike standard cluster-robust inference, these estimators are based on a design-based approach, where uncertainty owes to the sampling process and the treatment assignment mechanism.  As the interest in this setting is in estimating some average treatment effect for a particular population, inference is conducted with respects to the finite population of interest (for example, all states in a country), rather than infinite super-populations flowing from some data-generating process.  In many situations, a considerable proportion of clusters, or even all clusters, may be sampled, and in these cases standard errors based on TSCB or CCV can be considerably smaller than traditional model-based cluster-robust standard errors (ie Stata's `s vce(cluster clustvar)` implementation).

Uncertainty related to estimated regression parameters in this setting owes to the following elements: 
1. The Sampling process:
   - The proportion of clusters sampled (may be 100%).  Referred to as *qk* below.
   - The proportion of individuals sampled within each cluster (may be 100%).  Referred to as *pk* below.
2. The Treatment assignment mechanism: The proportion of each cluster assigned to receive treatment
3. The Heterogeneity in treatment effects across clusters

For example, consider the following three circumstances, based on US Census data and returns to college education discussed in [Abadie et al (2023)](#references). These are based on three simulations: the top panel considers inference on the full sample of individuals in the 2000 US census data, and hence *pk=1* and where all states are observed, hence *qk=1*.  The second panel considers the same case, but where returns to college have been simulated to have larger variation (ie there is more treatment effect heterogeneity across clusters), and there is more variation in the variation of treatment assignment across clusters.  The third panel is identical to top panel, however here only 26 of 52 clusters are sampled *(qk=0.5)*. 

**6 PANELS OF GRAPHS:** Top two show something along the lines of Figures 1a and 1b from overleaf, middle two show variation from design where sigma_tau and sigma_k are increased a lot, and bottom two show something along the lines of Figures 4 panels (a) and (b).

The implications of this variation for standard errors can be considerable.  In the plot below, a range of point estimates and 95% confidence intervals are displayed corresponding to simulations described in section VI of [Abadie et al (2023)](#references), where we additionally vary the proportion of clusters sampled.  Here we observe that in these cases, confidence intervals based on TSCB and Causal Cluster Variance estimates achieve good coverage with respects to the aymptotic variance of interest.  They are additionally considerably shorter than confidence intervals based on traditional (model based) cluster robust standard errors, particularly in the case when not all clusters are sampled.

**FIGURE HERE THAT DANIEL SET-UP IN SIMULATIONS**

The **Causal Cluster Variance** estimator is a closed-form variance estimate for treatment effects which is based on a refinement to the standard cluster-robust variance estimator.  The computational implementation of this estimator follows equation (13) of [Abadie et al (2023)](#references) in cases where all clusters are sampled, or equation (14) in cases where all clusters are not sampled.
The **Two-Stage Cluster Bootstrap** estimator is a bootstrap-based variance estimator for treatment effects where bootstrap resamples have alternative treatment assignment probabilities than in the original sample, while allowing for the case where a large fraction of clusters are observed.  The comuptational implementation of this estimator follows Algorithm 1 of [Abadie et al (2023)](#references).

Both cases additionally admit for fixed effects estimators, following section V of [Abadie et al (2023)](#references).  Details on code installation and implementation are below, with full documentation available in help files installed with the programs.


## TSCB: Two-Stage Cluster Bootstrap
[tscb.ado](tscb.ado) - Implements the Two-Stage Cluster Bootstrap variance, reporting standard errors for OLS or fixed effects models.  It additionally reports typical cluster robust and heteroscedasticity robust standard errors. This code follow algorithm 1 of [Abadie et al (2023)](#references).  Additional details can be found after installation by typing `help tscb` in Stata.

To install directly into Stata:
```s
net install tscb, from("https://raw.githubusercontent.com/daniel-pailanir/tscb-ccv/master") replace
```

### Syntax
```s
tscb Y W M [if] [in], qk() seed() reps() fe
```

Where Y is an outcome variable, W a binary treatment variable and M is a variable indicating the group over which clustering is calculated. We provide an example based on the 2000 US Census, discussed in the introduction of [Abadie et al (2023)](#references).

### OLS Estimates 
```s
webuse set www.damianclarke.net/stata/

webuse "census2000_5pc.dta", clear

* run TSCB
tscb ln_earnings college state, qk(1) seed(2022) reps(150)
```
The code returns the following results

```
Two-Stage Cluster Bootstrap replications (150).
----+--- 1 ---+--- 2 ---+--- 3 ---+--- 4 ---+--- 5
..................................................     50
..................................................     100
..................................................     150

OLS regression with Two-Stage Cluster Bootstrap Variance
                                                Number of obs     =  2,632,838
                                                R-squared         =     0.0567

------------------------------------------------------------------------------
 ln_earnings |      Coef.   Std. Err.      z    P>|z|     [95% Conf. Interval]
-------------+----------------------------------------------------------------
     college |  0.4656426  0.0034122   136.46   0.000    0.4589548   0.4723304
-------------+----------------------------------------------------------------
 Robust SE   |             0.0011619   400.76   0.000    0.4633653   0.4679199
 Cluster SE  |             0.0268800    17.32   0.000    0.4129588   0.5183264
------------------------------------------------------------------------------
```


## CCV: Causal Cluster Variance
[ccv.ado](ccv.ado) - Implements the Causal Cluster Variance, reporting standard errors for OLS or fixed effects models.  It additionally reports typical cluster robust and heteroscedasticity robust standard errors. This code implements equations (13), (14), or (20) of [Abadie et al (2023)](#references), depending whether all clusters are sampled or not, and whether OLS or FE models are desired.  Additional details can be found after installation by typing `help ccv` in Stata.

To install directly into Stata:
```s
net install ccv, from("https://raw.githubusercontent.com/daniel-pailanir/tscb-ccv/master") replace
```

### Syntax
```s
ccv Y W M [if] [in], qk() pk() seed() reps() fe
```

Where Y is an outcome variable, W a binary treatment variable and M a variable indicating the group over which clustering is calculated.  Below, an example is provided using the 5% sample of the 2000 US Census, described in the introduction of Abadie et al. (2023).

### OLS Estimates
```s
webuse set www.damianclarke.net/stata/

webuse "census2000_5pc.dta", clear

* run CCV
ccv ln_earnings college state, pk(0.05) qk(1) seed(2022) reps(8)
```
The code returns the following results

```
Causal Cluster Variance with (8) sample splits.
----+--- 1 ---+--- 2 ---+--- 3 ---+--- 4 ---+--- 5
........
OLS regression with Causal Cluster Variance
                                                Number of obs     =  2,632,838
                                                R-squared         =     0.0567

------------------------------------------------------------------------------
 ln_earnings |      Coef.   Std. Err.      z    P>|z|     [95% Conf. Interval]
-------------+----------------------------------------------------------------
     college |  0.4656426  0.0035393   131.56   0.000    0.4587057   0.4725795
-------------+----------------------------------------------------------------
 Robust SE   |             0.0011619   400.76   0.000    0.4633653   0.4679199
 Cluster SE  |             0.0268800    17.32   0.000    0.4129588   0.5183264
------------------------------------------------------------------------------
```

## References
**When Should You Adjust Standard Errors for Clustering?**, Alberto Abadie, Susan Athey, Guido W Imbens, Jeffrey M Wooldridge, The Quarterly Journal of Economics, 138(1):1-35, 2023.



---
title: "Quantifying Recurrences in State Space"
author: "Fred Hasselman & Maarten Wijnants"
date: "1/14/2018"
output: 
  html_document: 
    fig_caption: yes
    highlight: pygments
    keep_md: yes
    number_sections: no
    theme: spacelab
    toc: yes
    toc_float: true
    collapsed: false
    smooth_scroll: true

---


# **Phase Space Reconstruction and RQA** 

You can use `R` to run `RQA` analyses. These assignments assume you'll use `R`, but you can also find a `Matlab` toolbox for `RQA` here: [CRP toolbox](http://tocsy.pik-potsdam.de/CRPtoolbox/)

You'll need:

```r
library(rio)
library(crqa)
```

```
## Loading required package: Matrix
```

```
## Loading required package: tseriesChaos
```

```
## Loading required package: deSolve
```

```
## Loading required package: fields
```

```
## Loading required package: spam
```

```
## Loading required package: dotCall64
```

```
## Loading required package: grid
```

```
## Spam version 2.1-1 (2017-07-02) is loaded.
## Type 'help( Spam)' or 'demo( spam)' for a short introduction 
## and overview of this package.
## Help for individual functions is also obtained by adding the
## suffix '.spam' to the function name, e.g. 'help( chol.spam)'.
```

```
## 
## Attaching package: 'spam'
```

```
## The following objects are masked from 'package:base':
## 
##     backsolve, forwardsolve
```

```
## Loading required package: maps
```

```
## Loading required package: plot3D
```

```
## Loading required package: pracma
```

```
## 
## Attaching package: 'pracma'
```

```
## The following object is masked from 'package:deSolve':
## 
##     rk4
```

```
## The following objects are masked from 'package:Matrix':
## 
##     expm, lu, tril, triu
```

```r
library(fractal)
```

```
## Loading required package: splus2R
```

```
## 
## Attaching package: 'splus2R'
```

```
## The following object is masked from 'package:pracma':
## 
##     peaks
```

```
## Loading required package: ifultools
```

```r
library(casnet)
```

```
## 
## Attaching package: 'casnet'
```

```
## The following object is masked from 'package:pracma':
## 
##     repmat
```

**R-packages for Phase Space Reconstruction**

We'll use package `fractal` and `rgl` to reconstruct some phase spaces.

* If you have sourced the functions in `nlRtsa_SOURCE.R` you can install and load these packages by running: `in.IT(c("fractal","rgl"))`. The function `in.IT()` will first check if the requested packages are installed on the machine and only install them if they are not present. 

## Reconstruct the Lorenz attractor {#RQA} 

* Package `fractal` includes the 3 dimensions of the Lorenz system in the chaotic regime. 
    + Run this code `rgl::plot3d(lorenz)` with both packages loaded to get an interactive 3D plot ^[You'll need the [X Window System](http://www.x.org/wiki/) for interactive 3D plotting. This Linux desktop system comes installed in some way or form on most Mac and Windows systems.You can test if it is present by running `rgl::open3d()`, which will try to open an interactive plotting device]
 of this strange attractor.


We'll reconstruct the attractor based on just dimension `X` of the system using functions from package `fractal`, be sure to look at the manual pages of these functions.

\BeginKnitrBlock{rmdimportant}<div class="rmdimportant">Package `fractal` and package `nonlinearTseries` use functions with similar names, do not load them together.</div>\EndKnitrBlock{rmdimportant}

* Use `lx <- lorenz[1:2048,1]` to reconstruct the phase space based on `lx`. 
    + Find an optimal embedding lag using `timeLag()`, use `method = "mutual"`.
    + Find the embedding dimension, using `FNN()`
    + Embed the timeseries using `embedSeries()`.
    + Plot the reconstructed phase space. (You'll need to use `as.matrix()` on the object created by `embedSeries()`)
        - Use  `rgl::plot3d()` to plot the reconstructed space.

## Reconstruct the Predator-Prey model

Use the same procedure as above to reconstruct the state space of the predator-prey system. (Look [at the solutions](#ppdsol) to get a `Foxes` or `Rabbit` series).

 * You should get a 2D state space, so 3D plotting might be a bit too much for this system.


## (auto-) Recurrence Quantification Analysis
There are several packages which can perform (C)RQA analysis, we'll use [`crqa`](https://cran.r-project.org/web/packages/crqa/index.html) because it can perform both continuous and categorical analyses. If you only have continuous data, you migh be better off using package [`nonlinearTseries`](https://cran.r-project.org/web/packages/nonlinearTseries/index.html), in this course we will only use package `crqa`.

\BeginKnitrBlock{rmdimportant}<div class="rmdimportant"> Package `crqa()` was designed to run categorical Cross-Recurrence Quantification Analysis (see [Coco & Dale (2014)](http://journal.frontiersin.org/Journal/10.3389/fpsyg.2014.00510/abstract) and for R code see appendices in [Coco & Dale (2013)](http://arxiv.org/abs/1310.0201)). We can trick it to run auto-RQA by providing the same timeseries for `ts1` and `ts2` and setting the parameter `side` to either `"upper"` or `"lower"`</div>\EndKnitrBlock{rmdimportant}

 * Perform an RQA on the reconstructed state space of the Lorenz system.
    + You'll need a radius (also called: threshold) in order to decide which points are close together (recurrent).
        - `crqa` provides a function which will automatically select the best parameter settings: `optimizeParam()`
        - Best way to ensure you are using the same parameters in each function is to create some lists with parameter settings (check the `crqa` manual to figure out what these parameters mean):



```r
# General settings for `crqa()`
par0 <- list(rescale = 1,
             normalize = 0,
             mindiagline = 2,
             minvertline = 2,
             tw = 0,
             whiteline = FALSE,
             recpt = FALSE,
             side = "lower",
             checkl = list(do = FALSE, thrshd = 3, datatype = "categorical",pad = TRUE)
             )

# Settings for `optimizeParam()`
par <- list(lgM =  20, steps = seq(1, 6, 1),
           radiusspan = 100, radiussample = 40,
           normalize = par0$normalize, 
           rescale = par0$rescale, 
           mindiagline = par0$mindiagline, minvertline = par0$minvertline,
           tw = par0$tw, 
           whiteline = par0$whiteline, 
           recpt = par0$recpt, 
           fnnpercent = 10, typeami = "mindip")
```

* Get the optimal parameters using a radius which will give us 2\%-5\% recurrent points.


```r
ans <- optimizeParam(ts1 = lx, ts2 = lx, par = par, min.rec = 2, max.rec = 5)
```

* Run the RQA analysis using the same settings with which the parameters were found.


```r
crqaOutput <- crqa(ts1 = lx, ts2 = lx, 
                  delay = ans$delay, 
                  embed = ans$emddim, 
                  radius = ans$radius, 
                  normalize = par0$normalize,
                  rescale = par0$rescale, 
                  mindiagline = par0$mindiagline, minvertline = par0$minvertline,
                  tw = par0$tw, 
                  whiteline = par0$whiteline, 
                  recpt = par0$recpt, 
                  side = par0$side, checkl = par0$checkl
                  )
```

* The output of `crqa` is a list with recurrence measures, the last entry is the recurrence plot. It is represented as a so-called `sparse-matrix`. 
    + This representation severely decreases the amount of memory occupied by the recurrence matrix. It is basically a list of indices of cells that contain a $1$. The $0$ do not need to be stored.
    + In order to plot this matrix you could use `image()`, but this does not produce the recurrence plot as they are usually displayed, the y-axis needs to be flipped.
    + We created a function which will take as input the list output of `crqa`, which wil be used to plot the recurrence matrix. If you have  sourced the `nlRtsa` functions you can call `plotRP.crqa(crqaOutput)`.
    
    
[| Jump to solutions |](#PSRsol)


## RQA of circle-tracing data

* Analyse the [circle tracing data](https://github.com/FredHasselman/DCS/tree/master/assignmentData/RQA_circletrace) we recorded during the lecture.
* Study what happens to the RQA measures if you shuffle the temporal order (see e.g. the solution to th [HRV assignments](#hrvsol)).
* Package `fractal` contains a function `surrogate`. This will create so-called *constrained* realisations of the time series. Look at the help pages of the function, or study the *Surrogates Manual* of the [TISEAN software](http://www.mpipks-dresden.mpg.de/~tisean/Tisean_3.0.1/index.html) and create two surrogate series, one based on `phase` and one on `aaft`.
     + Look at the RQA measures and think about which $H_0$ should probably be rejected.
     + If you want to be more certain, you'll have to create a test (more surrogates). The [TISEAN manual](http://www.mpipks-dresden.mpg.de/~tisean/Tisean_3.0.1/docs/surropaper/node5.html#SECTION00030000000000000000) provides all the info you need to construct such a test:
     
     > "For a minimal significance requirement of 95\% , we thus need at least 19 or 39 surrogate time series for one- and two-sided tests, respectively. The conditions for rank based tests with more samples can be easily worked out. Using more surrogates can increase the discrimination power."
  

[| Jump to solutions |](#RQAsol)


# **Categorical and Cross-RQA (CRQA)** 



You can use `R` or `Matlab` to run `RQA` analyses. These assignments assume you'll use `R`. You can find a `Matlab` toolbox for `RQA` here: [CRP toolbox](http://tocsy.pik-potsdam.de/CRPtoolbox/)

For `R` you'll probably need:

```r
library(rio)
library(crqa)
library(fractal)
library(devtools)
source_url("https://raw.githubusercontent.com/FredHasselman/DCS/master/functionLib/nlRtsa_SOURCE.R")
```


## Assignment: CRQA and Diagonal Profile {#CRQA}

1. Create two variables for CRQA analysis, or use the $x$ and $y$ coordinates we roecorded during the lecture:

```r
y1 <- sin(1:900*2*pi/67)
y2 <- sin(.01*(1:900*2*pi/67)^2)

# Here are the circle trace data
xy <- import("https://raw.githubusercontent.com/FredHasselman/DCS/master/assignmentData/RQA_circletrace/mouse_circle_xy.csv") 

y1 <- xy$x
y2 <- xy$y
```

2. You have just created two sine(-like) waves. We’ll examine if and how they are coupled in a shared phase space. As a first step plot them.

3. Find an embedding delay (using mutual information) and an embedding dimension (if you calculate an embedding dimension using package `fractal` for each signal seperately, as a rule of thumb use the highest embedding dimension you find in further analyses).


```r
# General settings for `crqa()`
par0 <- list(rescale = 0,
             normalize = 0,
             mindiagline = 2,
             minvertline = 2,
             tw = 0,
             whiteline = FALSE,
             recpt = FALSE,
             side = "both",
             checkl = list(do = FALSE, thrshd = 3, datatype = "categorical",pad = TRUE)
             )

# Settings for `optimizeParam()`
par <- list(lgM =  20, steps = seq(1, 6, 1),
           radiusspan = 100, radiussample = 40,
           normalize = par0$normalize, 
           rescale = par0$rescale, 
           mindiagline = par0$mindiagline, minvertline = par0$minvertline,
           tw = par0$tw, 
           whiteline = par0$whiteline, 
           recpt = par0$recpt, 
           fnnpercent = 10, typeami = "mindip")
```

4. We can now create a cross recurrence matrix. Fill in the values you decided on. You can choose a radius automatically, look in the `crqa` manual.
* Get the optimal parameters using a radius which will give us 2\%-5\% recurrent points.

> Note: There is no rescaling of data, the sines were created in the same range. You can plot a matrix using `image()`. You could also check package `nonlinearTseries`. If you sourced the `nlRtsa` library, use `plotRP.crqa()`



```r
(ans <- optimizeParam(ts1 = y1, ts2 = y2, par = par, min.rec = 2, max.rec = 5))
```

5. Run the CRQA and produce a plot of the recurrence matrix.



6. Can you understand what is going on? 
  * For the simulated data: Explain the the lack of recurrent points at the beginning of the time series.
  * For the circle trace: How could one see these are not determisnistic sine waves?



7. Examine the synchronisation under the diagonal LOS. Look in the manual of `crqa` or  [Coco & Dale (2014)](http://journal.frontiersin.org/Journal/10.3389/fpsyg.2014.00510/abstract). 
 * To get the diagonal profile from the recurrence plot, use `spdiags()`.
 * Make a plot of the diagonal profile. How far is the peak in RR removed from 0 (Line of Synchronisation)? 





8. Perform the same steps with a shuffled version (or use surrogate analysis!) of the data of timeseries $y1$. You can use the embedding parameters you found earlier. 

> NOTE: If you generate surrogate timeseries, make sure the RR is the same for all surrogates. Try to keep the RR in the same range by using the `min.rec` and `max.rec` settings of `optimizeParam` for each surrogate.

[| Jump to solutions |](#CRQAsol)


## Categorical CRQA {#catCRQA}

1. Package CRQA contains two categorical trial series from the "Friends"study.
   * Lookup `RDts1` and `RDts2` in the help file. 
   * Also lookup the articles discussing the study to get an idea about these series [here](http://cognaction.org/rdmaterials/php.cv/pdfs/article/richardson_dale_2005.pdf) and [here](http://cognaction.org/rdmaterials/php.cv/pdfs/article/richardson_dale_kirkham.pdf))  

2.  In the paper accompanying the package by [Coco & Dale (2014)](http://journal.frontiersin.org/Journal/10.3389/fpsyg.2014.00510/abstract) demonstrate the analysis of these series. 
    * Recreate this analysis, e.g. Figure 10 in the article including the CRQA measures.

[| Jump to solutions |](#catCRQAsol)
---
title: ggplot2 2.3.0 — upcoming release
author: Mara Averick
date: '2018-05-21'
slug: ggplot2-2-3-0
description: >
    What you need to know about upcoming changes for ggplot2 2.3.0.
categories: [package]
tags:
  - ggplot2
  - tidyverse
photo:
  url: https://unsplash.com/photos/OhQhkkVJYhI
  author: chuttersnap
---


A new release of ggplot2 (2.3.0) is on the horizon, and we'd love to have people 
try it out, give us feedback, and [report issues](https://github.com/tidyverse/ggplot2/issues) 
before we submit it to CRAN. This version is the culmination of over a year and 
a half of development, not all of which will be captured here. So, please see [NEWS](https://github.com/tidyverse/ggplot2/blob/master/NEWS.md) for a fuller 
account of updates and changes.

In addition to highlighting a few features and improvements, we also want to 
share a bit about our release-preparation process for ggplot2, which has over 
2,000 reverse dependencies.


```r
# install.packages("devtools")
devtools::install_github("tidyverse/ggplot2")
```

If needed, you can restore the release version by installing from CRAN:


```r
install.packages("ggplot2")
```

## Breaking changes

### Our process

Regression testing, visual and otherwise, allows us to keep track of changes
throughout the development process. Before submitting an update to CRAN,
we need to ensure that the packages that depend on ours still pass `R CMD
check`, and inform reverse-dependency maintainers in the event that they do not.[^1] 
We use [revdepcheck](https://github.com/r-lib/revdepcheck) to help automate this 
process, and distinguish between "false positives" (pre-existing failures), and 
issues introduced in the development version of your package. 

At this point, we begin to manually examine the failures. "Newly broken" issues
are extracted from the problems report and put into a Google sheet where (with
the great help of Thomas Lin Pedersen, Claus Wilke, and Kara Woo) we identified
clusters of errors to tackle. For example, 97 packages returned a warning due
to a namespace clash between `dplyr::vars()` and `ggplot2::vars()`. That’s
something we clearly need to fix, so we open an
[issue](https://github.com/tidyverse/ggplot2/issues/2585) to track progress.

Where reverse-dependency errors result from a deliberate design change, we 
collect "symptomatic" error messages, and (hopefully) fixes to offer the 
maintainers of affected packages. These go in the "Breaking changes" section of 
the NEWS file, so that if you encounter a new error you can easily search for 
the error message and find the resolution. Because ggplot2 is an old and much 
used package, we try to keep breaking changes to a minimum. However, sometimes 
we deem it important to make a change that break existing code. Typically this 
is for one of two reasons:

 * We discover that code that previously appeared to work was actually hiding a 
 potential error. For example, in this version of ggplot2 we ensure that input 
 data frames don’t have duplicate column names. Previously ggplot2 just took the 
 first column with a name, which might have concealed a bug.  

 * We decided that it’s worth breaking a small amount of existing code in the 
 interests of improving future code. For example, this version of ggplot2 adds 
 support for [tidy evaluation](https://github.com/tidyverse/ggplot2/blob/master/NEWS.md#tidy-evaluation) 
 making ggplot2 more compatible with the rest of the tidyverse. This breaks a 
 small number of packages that use ggplot2 but should affect very little data 
 analysis code, so we decided that it was worth it.  

### What to do if you run into a breaking change

If you run into an error when running code that works with ggplot2 
2.2.1 (the version on CRAN as of this writing), you should look for it in the
["Breaking changes"](https://github.com/tidyverse/ggplot2/blob/master/NEWS.md#breaking-changes) 
section of news. If you _can't_ find it in the news, please file an issue - 
either it's an inadvertent change, or we need to describe it better in the news 
(or make it easier to find when searching for keywords). As per usual, please 
do a search of [existing issues](https://github.com/tidyverse/ggplot2/issues) 
before filing a new one to avoid duplication.

You can also ask for help in [community.rstudio.com](https://community.rstudio.com/).

## New features

### Tidy evaluation

You can now use quasiquotation in `aes()`, `facet_wrap()`, and `facet_grid()`. 
To support quasiquotation in facetting we've added a new helper that works 
similarly to `aes()`: `vars()`, short for variables, and instead of 
`facet_grid(x + y ~ a + b)` you can now write `facet_grid(vars(x, y), vars(a, b))`. 
The formula interface won't go away; but the new `vars()` interface should be 
much easier to program with.


```r
x_var <- quo(wt)
y_var <- quo(mpg)
group_var <- quo(cyl)

ggplot(mtcars, aes(!!x_var, !!y_var)) + 
  geom_point() + 
  facet_wrap(vars(!!group_var))
```

<img src="/articles/2018-05-ggplot2-2-3-0_files/figure-html/label-both-1.png" width="2100" />

### sf

ggplot2 now has full support for simple features through 
[sf](https://r-spatial.github.io/sf/) using `geom_sf()` and `coord_sf()`; it now 
automatically aligns CRS across layers, sets up the correct aspect ratio, and 
draws a graticule.


```r
nc <- sf::st_read(system.file("shape/nc.shp", package = "sf"))
#> Reading layer `nc' from data source `/Library/Frameworks/R.framework/Versions/3.4/Resources/library/sf/shape/nc.shp' using driver `ESRI Shapefile'
#> Simple feature collection with 100 features and 14 fields
#> geometry type:  MULTIPOLYGON
#> dimension:      XY
#> bbox:           xmin: -84.32385 ymin: 33.88199 xmax: -75.45698 ymax: 36.58965
#> epsg (SRID):    4267
#> proj4string:    +proj=longlat +datum=NAD27 +no_defs
ggplot(nc) +
  geom_sf() +
  annotate("point", x = -80, y = 35, colour = "red", size = 4)
```

<img src="/articles/2018-05-ggplot2-2-3-0_files/figure-html/sf-1.png" width="2100" />

### Calculated aesthetics

The new `stat()` function offers a cleaner, and better-documented syntax for 
calculated-aesthetic variables. This replaces the older approach of surrounding 
the variable name with `...`. Instead of using `aes(y = ..count..)`, you can use 
`aes(y = stat(count))`. 

This is particularly nice for more complex calculations, as `stat()` only needs 
to be specified once; e.g. `aes(y = stat(count / max(count)))` rather than 
`aes(y = ..count.. / max(..count..))`.

### Viridis

Several new functions have been added to make it easy to use Viridis colour 
scales: `scale_colour_viridis_c()` and `scale_fill_viridis_c()` for continuous, 
and `scale_colour_viridis_d()` and `scale_fill_viridis_d()` for discrete. 
Viridis is also now used as the default colour and fill scale for ordered 
factors.

## And more...

There are several other improvements, which will be described in further detail 
once the ggplot 2.3.0 is released on CRAN (currently targeted for June 25th). 
For now, please kick the tires and let us know what you think!

[^1]: [CRAN Repository Policy](https://cran.r-project.org/web/packages/policies.html), Version $Revision: 3874. CRAN Repository Maintainers. <https://cran.r-project.org/web/packages/policies.html>

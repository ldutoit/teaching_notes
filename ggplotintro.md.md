Data visualisation with ggplot
================
Ludovic Dutoit, summary of @carpentries.org content

The code below aims to be a very short refresher and summarises what we
have done together in class.

The whole lesson is well-written and available at:
<https://datacarpentry.org/R-ecology-lesson/04-visualization-ggplot2.html>

The [R graph gallery](R%20graph%20gallery) also contains loads of
creative and clean plots with their associated code

## A simple plot

loading the data and the library. The data is species observation on
different type of field plots in different years.

``` r
library("tidyverse")
surveys_complete <- read_csv("data/surveys_complete.csv")
```

The essence of the ggplot is of the following
    format.

    ggplot(data = <DATA>, mapping = aes(<MAPPINGS>)) +  <GEOM_FUNCTION>()

data and mapping will inform ggplot on what we try to plot on the x and
y axis, but without GEOM\_FUNCTION, no point, line or fancy
histograms.

``` r
ggplot(data = surveys_complete, mapping = aes(x = weight, y = hindfoot_length))
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

Let’s plot actual points by adding our first geom,
geom\_point()

``` r
ggplot( data = surveys_complete, mapping = aes(x= weight, y = hindfoot_length)) + geom_point()
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

The whole core is a bit tedious to write, so we can save it into an
object to be able to plot quickly the same data, varying all the
geom.

``` r
surveys_plot <-ggplot(data = surveys_complete, mapping = aes(x = weight, y = hindfoot_length))
surveys_plot + geom_point()
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

Let’s use some blue points with some transparency

    surveys_plot  + geom_point(alpha = 0.1, col = "blue")

Let’s color by species\_id

``` r
surveys_plot + geom_point(alpha = 0.1, aes(color = species_id))
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## Boxplot

We will now plot weight against species\_id, one boxplot per species.
Let’s save the data and mapping into an
object:

``` r
boxplot_weight <- ggplot(data = surveys_complete, mapping = aes( x= species_id, y=weight)) 
boxplot_weight
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-6-1.png)<!-- --> Now
we can add the actual\_boxplot and other geom\_functions:

``` r
boxplot_weight + geom_boxplot()
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

and with jitter, two plots on one just by adding an extra
+

``` r
boxplot_weight +  geom_boxplot() + geom_jitter(alpha=0.3, color= "tomato")
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

## Time series

Let’s create a plot of genus through time. We can use dplyr and the pipe
( %\>%) to count the number of occurences of each genus year by year.

``` r
yearly_counts <- surveys_complete %>%
  count(year, genus)
```

Let’s have a quick look:

``` r
head(yearly_counts)
```

    ## # A tibble: 6 x 3
    ##    year genus               n
    ##   <dbl> <chr>           <int>
    ## 1  1977 Chaetodipus         3
    ## 2  1977 Dipodomys         222
    ## 3  1977 Onychomys           1
    ## 4  1977 Perognathus        22
    ## 5  1977 Peromyscus          2
    ## 6  1977 Reithrodontomys     2

It seem to work.

Let’s try a new geom, geom\_line() to plot lines.

``` r
ggplot(data = yearly_counts, mapping = aes(x= year, y = n)) +
  geom_line()
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

Oops, all the genera together does not work well, let’s split and have
one line per genus. That can be easily done in
ggplot():

``` r
ggplot(data = yearly_counts, aes(x= year, y = n, group = genus)) + geom_line()
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

let’s add a color
key:

``` r
ggplot(data = yearly_counts, aes(x = year, y =n, color = genus)) + geom_line()
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

## Faceting

Let’s use **faceting** and add the facet\_wrap which allows us to split
plots into subplots using a specific
variable:

``` r
ggplot(data = yearly_counts, aes(x= year, y =n )) + geom_line() + facet_wrap(facets = vars(genus))
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

Let’s plot each sex for each year in each genus. Group\_by() will group
observation by year, genus and then sex before being counted using
tally().

``` r
yearly_sex_counts <- surveys_complete %>% 
  group_by(year, genus, sex)  %>% 
  tally()
head(yearly_sex_counts)
```

    ## # A tibble: 6 x 4
    ## # Groups:   year, genus [4]
    ##    year genus       sex       n
    ##   <dbl> <chr>       <chr> <int>
    ## 1  1977 Chaetodipus F         3
    ## 2  1977 Dipodomys   F       103
    ## 3  1977 Dipodomys   M       119
    ## 4  1977 Onychomys   F         1
    ## 5  1977 Perognathus F        14
    ## 6  1977 Perognathus M         8

Now let’s plot it, notice how we pass sex in the color argument and
create subplots using the facet\_wrap()
function.

``` r
ggplot(data = yearly_sex_counts, mapping = aes( x = year, y=n, color= sex))+ geom_line() + facet_wrap(facets = vars(genus) ) 
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

One can also organise the facets a bit more specifically using
facet\_grid: An example with one line male, one line
female

``` r
ggplot(data = yearly_sex_counts, mapping = aes(x= year, y=n, color=sex)) + geom_line() + facet_grid(rows=vars(sex), cols = vars(genus)) 
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

## A little bit more customisation

There is a lot more plotting that can be done using iterative building
to add functions one after the other separated by the + sign.

One can add
[themes](https://ggplot2.tidyverse.org/reference/ggtheme.html) to change
the general
layout.

``` r
ggplot(data = yearly_sex_counts, mapping = aes( x = year, y=n, color= sex))+ geom_line() + facet_wrap(facets = vars(genus) ) + theme_bw() 
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

For more content, I highly encourage you to refer to the [original
lesson](https://datacarpentry.org/R-ecology-lesson/04-visualization-ggplot2.html).

Let’s play with vertical axes, and title for axes as
well.

``` r
ggplot(data = yearly_sex_counts, mapping = aes( x = year, y=n, color= sex))+ geom_line() + facet_wrap(facets = vars(genus) ) +labs(title = "Observed genera through time", x= "Year of observation", y = "Number of individuals") + theme_bw() + theme(text=element_text(size=16), axis.text.x = element_text(colour = "grey20", angle = 90, size = 12))
```

![](ggplotintro.md_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->

## Last but not least Save your plot\!

Put it in an
object:

``` r
combo_plot <- ggplot(data = yearly_sex_counts, mapping = aes( x = year, y=n, color= sex))+ geom_line() + facet_wrap(facets = vars(genus) ) +labs(title = "Observed genera through time", x= "Year of observation", y = "Number of individuals") + theme_bw() + theme(text=element_text(size=16), axis.text.x = element_text(colour = "grey20", angle = 90, size = 12))
```

Then save it…

``` r
ggsave("combo_plot_final.png", combo_plot, width =10, dpi =300)
```

    ## Saving 10 x 5 in image

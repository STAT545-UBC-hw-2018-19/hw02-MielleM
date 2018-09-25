---
title: "MielleM hw002"
output:
  html_document:
    keep_md: true
---
#### To start off, let's get tidyverse and gapminder loaded
Load gapminder data and tidyverse package

```r
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.2.1 ──
```

```
## ✔ ggplot2 3.0.0     ✔ purrr   0.2.5
## ✔ tibble  1.4.2     ✔ dplyr   0.7.6
## ✔ tidyr   0.8.1     ✔ stringr 1.3.1
## ✔ readr   1.1.1     ✔ forcats 0.3.0
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

```r
library(gapminder)
library(scales)
```

```
## 
## Attaching package: 'scales'
```

```
## The following object is masked from 'package:purrr':
## 
##     discard
```

```
## The following object is masked from 'package:readr':
## 
##     col_factor
```


##Smell test the data

Question | Answer
---------|-------
Object type | data.frame
Class | tibble, or tbl_df
Columns | 6
Rows | 1704

Column | Type
-------|-----
country | factor
continent | factor
year | integer
lifeExp | double (numeric)
pop | integer
gdpPercap | double (numeric)

Most attributes of the tibble (e.g. number of observations) can be individually obtained by specific commands, rather than the larger summary created by glimpse(gapminder). If only one piece of information was needed, it would make more sense to target that specifically. 

The class command gives the type and class of gapminder. The glimpse command gives the number of observations, as well as the number, name, and data type of each variable. This is the neatest way to summarize what the tibble contains. 

```r
class(gapminder)
```

```
## [1] "tbl_df"     "tbl"        "data.frame"
```

```r
glimpse(gapminder)
```

```
## Observations: 1,704
## Variables: 6
## $ country   <fct> Afghanistan, Afghanistan, Afghanistan, Afghanistan, ...
## $ continent <fct> Asia, Asia, Asia, Asia, Asia, Asia, Asia, Asia, Asia...
## $ year      <int> 1952, 1957, 1962, 1967, 1972, 1977, 1982, 1987, 1992...
## $ lifeExp   <dbl> 28.801, 30.332, 31.997, 34.020, 36.088, 38.438, 39.8...
## $ pop       <int> 8425333, 9240934, 10267083, 11537966, 13079460, 1488...
## $ gdpPercap <dbl> 779.4453, 820.8530, 853.1007, 836.1971, 739.9811, 78...
```


##Explore individual variables


### Exploring a quantitative variable: GDP per capita  

#### With a five number summary:

```r
gdp <- gapminder::gapminder$gdpPercap

summary(gdp)
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
##    241.2   1202.1   3531.8   7215.3   9325.5 113523.1
```
From the function above, we can see that GDP per capita ranges from $241.20 to $113,523.10, with a median value of $3,531.80.
The median and the mean ($7,215.30) are fairly different values, suggesting that this data is not normally distributed. 

#### Let's draw a kernel density plot check our assumption that the data is not normally distributed. 


```r
ggplot(gapminder, aes(gdpPercap)) +
  geom_density(color = "dark grey", fill = "light blue")+
  xlab("GDP per capita")
```

![](MielleMhw002_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

It appears that the data is not normally distributed, with the mode close to the median and a strong right skew. 


#### The same five number summary (minus the mean) from above, but drawn as a box plot
 

```r
boxplot(gdp)
```

![](MielleMhw002_files/figure-html/unnamed-chunk-5-1.png)<!-- -->


#### The box plot makes it difficult to see much of a trend in the GDP data. Let's create a box plot for each continent so we can compare. 


```r
g <- ggplot(gapminder, aes(continent, gdpPercap))
```



```r
g +
  geom_boxplot() +
  scale_y_continuous(labels = comma) +
  xlab("Continent") +
  ylab("GDP per capita")
```

![](MielleMhw002_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

Let's try log-transforming GDP so the data doesn't appear so bunched at the bottom of the graph.

```r
g + 
  scale_y_log10() +
  geom_boxplot() +
  xlab("Continent") +
  ylab("GDP per capita")
```

![](MielleMhw002_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

Asia appears to have the widest range of GDP values, and Oceania has the smallest. Europe has a somewhat normal-looking distribution.  



### Exploring a qualitative variable: Continent  

#### First, let's make a basic histogram containing the data count for each continent. 


```r
c <- ggplot(gapminder, aes(continent))

c + 
  geom_bar()
```

![](MielleMhw002_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

This histogram looks pretty boring with all the grey. Plus, it doesn't help tell the story that I want to show with the graph. I'm trying to emphasize the differences between the continents, so I'd like to make them all different colors. 

#### First, we can check ColorBrewer for a list of available preset color palettes. 

```r
RColorBrewer::display.brewer.all()
```

![](MielleMhw002_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

#### Once I've selected the desired palette, I can use it to color the bar graphs. 

```r
c <- ggplot(gapminder, aes(continent, fill=continent))
c + 
  geom_bar() +
  geom_text(stat = "count", aes(label = ..count.., y = ..count..)) +
  scale_fill_brewer(palette = "Dark2")
```

![](MielleMhw002_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

In this histogram, I've also included the counts. It looks like Africa has significantly more data points than any other continent, the Americas, Asia, and Europe have a fairly similar number, and Oceania has significantly fewer associated observations. 

The two histograms above are for all the observations, from the 1950's onward. Because the number of countries in each continent has stayed relatively stable, a histogram for an individual year should look fairly similar to the histograms for all the data. Let's test this assumption with the most recent year that data is available. 

#### First, let's find the most recent data available. 


```r
tail(sort(gapminder$year), 1)
```

```
## [1] 2007
```


#### Then, let's filter the data to this year, and create a new histogram for 2007. 


```r
gapminder %>%
  filter(year == 2007) %>% 
  ggplot(aes(continent, fill = continent)) +
  geom_bar() +
  geom_text(stat = "count", aes(label = ..count.., y = ..count..)) +
  scale_fill_brewer(palette = "Dark2")
```

![](MielleMhw002_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

As expected, this histogram is pretty similar to the one created with all the data. 



## Exploring various plot types


#### Here, I've created a line graph showing the progression of GDP over time. 

Each line represents a country, and are colored by continent. As you can see, it's pretty busy and therefor difficult to interpret. 


```r
v <- ggplot(gapminder, aes(year, gdpPercap, color = continent)) 

v + 
  geom_line(aes(group=country), alpha=0.3) +
  scale_color_brewer(palette = "Dark2") +
  theme_light()
```

![](MielleMhw002_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

To simplify, the graph below only displays data from countries with a population over 50,000,000. 


```r
gapminder %>%
  filter(pop > 50000000) %>% 
  ggplot(aes(year, gdpPercap, color = continent)) +
  geom_line(aes(group=country), alpha = 0.75) +
  scale_color_brewer(palette = "Dark2") +
  scale_y_continuous(labels = comma) +
  xlab("Year") +
  ylab("GDP per capita") +
  theme_light()
```

![](MielleMhw002_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

#### Visualizing three variables in one graph

I can visualize three variables by creating a plot displaying the distribution of GDP per capita by continent as both a violin plot and jitter plot. The dots of the jitter plot are colored to reflect life expectancy, with lower values represented by darker color. 


Here, I've filtered the years to 1980 and before. 

```r
gapminder %>%
  filter(year < 1980) %>% 
  ggplot(aes(continent, gdpPercap, color=lifeExp)) +
  geom_jitter() +
  geom_violin(alpha=0.6) +
  xlab("Continent") +
  ylab("GDP per capita") +
  theme_light()
```

![](MielleMhw002_files/figure-html/unnamed-chunk-16-1.png)<!-- -->


Here is the same graph, but with years after 1980. Note that the legend and y axis scale is slightly different. 

```r
gapminder %>%
  filter(year > 1980) %>% 
  ggplot(aes(continent, gdpPercap, color=lifeExp)) +
  geom_jitter() +
  geom_violin(alpha=0.6) +
  xlab("Continent") +
  ylab("GDP per capita") +
  theme_light()
```

![](MielleMhw002_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

From these two graphs, we can see that there has been an increase in both life expectancy and GDP per capita in the years before and after 1980. 




## Extra goodies

> Q: Evaluate this code and describe the result. Presumably the analyst’s intent was to get the data for Rwanda and Afghanistan. Did they succeed? Why or why not? If not, what is the correct way to do this?


```r
filter(gapminder, country == c("Rwanda", "Afghanistan"))
```

```
## # A tibble: 12 x 6
##    country     continent  year lifeExp      pop gdpPercap
##    <fct>       <fct>     <int>   <dbl>    <int>     <dbl>
##  1 Afghanistan Asia       1957    30.3  9240934      821.
##  2 Afghanistan Asia       1967    34.0 11537966      836.
##  3 Afghanistan Asia       1977    38.4 14880372      786.
##  4 Afghanistan Asia       1987    40.8 13867957      852.
##  5 Afghanistan Asia       1997    41.8 22227415      635.
##  6 Afghanistan Asia       2007    43.8 31889923      975.
##  7 Rwanda      Africa     1952    40    2534927      493.
##  8 Rwanda      Africa     1962    43    3051242      597.
##  9 Rwanda      Africa     1972    44.6  3992121      591.
## 10 Rwanda      Africa     1982    46.2  5507565      882.
## 11 Rwanda      Africa     1992    23.6  7290203      737.
## 12 Rwanda      Africa     2002    43.4  7852401      786.
```

A: If the analyst was trying to select both Rwanda and Afghanistan data, using this segment of code would not produce the desired result. This filter uses the logical function "=="" which is equivalent to the English word "and", and is only returning one of the two countries for each year that data exists. The correct function would be: 

```r
filter(gapminder, country %in% c("Rwanda", "Afghanistan"))
```

```
## # A tibble: 24 x 6
##    country     continent  year lifeExp      pop gdpPercap
##    <fct>       <fct>     <int>   <dbl>    <int>     <dbl>
##  1 Afghanistan Asia       1952    28.8  8425333      779.
##  2 Afghanistan Asia       1957    30.3  9240934      821.
##  3 Afghanistan Asia       1962    32.0 10267083      853.
##  4 Afghanistan Asia       1967    34.0 11537966      836.
##  5 Afghanistan Asia       1972    36.1 13079460      740.
##  6 Afghanistan Asia       1977    38.4 14880372      786.
##  7 Afghanistan Asia       1982    39.9 12881816      978.
##  8 Afghanistan Asia       1987    40.8 13867957      852.
##  9 Afghanistan Asia       1992    41.7 16317921      649.
## 10 Afghanistan Asia       1997    41.8 22227415      635.
## # ... with 14 more rows
```

 "%in%" is equivalent to the English word "or", and returns all observations with "Rwanda" or "Afghanistan". 
   





























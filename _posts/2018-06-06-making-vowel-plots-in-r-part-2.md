---
layout: post
title:  "Making vowel plots in R (Part 2)"
date:   2018-06-06 10:12:00 -0400
tags: [How-to Guides, Methods, Phonetics, R, Skills, Data Viz]
---

This is Part 2 of a four-part series of blog posts on how to make vowel plots in R. In [Part 1]({% post_url 2018-03-18-making-vowel-plots-in-r-part-1 %}), we looked primarily at how to plot individual data points as a scatterplot. This time, I'll focus entirely on trajectory data, that is, formant measurements per vowel at multiple points along its duration. Today, I'll cover three things: how to prepare FAVE output for trajectory plots, plotting trajectories in the F1-F2 space, and in the time-Hz space (like what you see in Praat). For both kinds of plots, we'll see how to show all tokens as well as averages per vowel.

As a sociolinguist working on English vowels, I work with FAVE data a lot and I know lots of other people do too. So for this tutorial to be applicable to as many other people as possible, I'll work directly with FAVE output. This way, you can apply the code here to your own data and hopefully get the same results. If you use Praat to extract your own formant measurements, some of the data wrangling may not directly apply to you.

Finally, I should mention that in the first tutorial I downplayed the data processing because I didn't need to do much. Today though, data processing is crucial for making these kinds of plots. I'll try to explain what I'm doing along the way, but if you haven't seen this kind of code before, it might be worth it to peruse [*R for Data Science*](http://r4ds.had.co.nz), which is free online. In fact, even if you have seen this code before, I'd highly recommend reading the book. It's a good one. 

# Data processing

So, to start, let's load the packages required for this tutorial:

```r
library(tidyverse)
```

The [`tidyverse`](https://www.tidyverse.org) library is actually a whole suite of packages that work harmoneously together. It includes ones like `ggplot2`, `dplyr`, and `tidyr`, which you might have used before. Instead of loading each package individually, I load them together at once using `library(tidyverse)`. The bonus is that it comes with a couple other packages that you might not be aware of, like `stringr` and `readr`. 

Note though that `tidyverse` requires a relatively recent version of R. If you haven't updated R since before about November 2017, you may have trouble installing and loading it. You can update R and it'll work fine, or you might have better luck loading `tidyr`, `dplyr`, `ggplot`, and `readr` individually. [This](https://www.r-bloggers.com/updating-r/) is a quick tutorial (not mine) on how to update R.

## Read in the data

We'll work with the same dataset that I used last time: a recording of me reading about 300 sentences while sitting at my kitchen table.

```r
my_vowels_raw <- read.csv("data/joey.csv") 
```

<div class="sidenote">If you read your data in using readr::read_csv(), which is the tidyverse way to go, it actually handles the original column names (F1@20%, etc.) just fine. Since R doesn't like those, you'll have to use ticks (`: that thing that shares a key with the tilde on my keyboard) around it: select(vowel, word, t, `F1@20%`:`F2@80%`).</div>

I've already removed most of the really bad outliers, but there are a few things I still need to do to get this data ready. First, I'll keep just the vowels with primary stress and remove /ɹ/ with the `filter` function. Then, I'll make all the words lowercase with `mutate`. I'll also create a new column that just numbers the rows sequentially which will give each vowel it's own unique identifier. For simplicity, I'm also going to remove some columns that we don't need for today---or rather, keep just the columns I need---using `select`. All I want for now are the ID I just created, the vowel, the word, and all the F1.20., F2.20., F1.35., etc. columns. This last group are formant measurements at 20%, 35%, 50%, 65%, and 80% into the vowels' durations. These five points give us the trajectory data we need. (I'll toss out the plain `F1`, `F2`, and `F3` columns because the time that those measurements come from varies from vowel to vowel.)

<div class="sidenote">Also, you can also use the time (the t column) as the unique id instead of creating new ones, but I like creating sequential numbers better.</div>

```r
my_vowels <- my_vowels_raw %>%
    filter(stress == 1, 
           vowel != "ER") %>%
    mutate(word = tolower(word),
           id = 1:nrow(.)) %>%
    select(id, vowel, word, F1.20.:F2.80.) %>%
    print()

## # A tibble: 1,695 x 13
##       id  vowel    word F1.20. F2.20. F1.35. F2.35. F1.50. F2.50. F1.65.
##    <int> <fctr>   <chr>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
##  1     1     AY   timer  742.2 1406.0  507.5 1022.6  580.7 1311.6  518.8
##  2     2     AA barring  540.2 1029.1  568.1  923.1  589.1  969.1  580.3
##  3     3     IH  injury  277.5 2294.8  329.1 2277.8  394.6 2242.6  391.9
##  4     4     EY  change  400.1 2145.7  400.1 2145.7  428.1 2209.8  442.7
##  5     5     AA   heart  861.7 2192.4  680.8 1632.6  540.9 1027.0  537.8
##  6     6     OW    bold  398.9 1382.8  449.2  855.3  462.6  707.9  431.2
##  7     7     AO    also  859.8 1847.5  447.9  990.9  538.9 1007.9  583.9
##  8     8     EY    play  462.5 1585.0  456.1 1601.3  411.3 1684.8  379.3
##  9     9     AW without  483.0 1518.5  594.3 1391.8  605.3 1210.0  547.2
## 10    10     AA    todd  775.3 1336.7  740.5 1162.1  614.6 1065.2  637.6
## # ... with 1,685 more rows, and 3 more variables: F2.65. <dbl>,
## #   F1.80. <dbl>, F2.80. <dbl>
```

As a review, here's what this looks like with a scatterplot of the formants at the midpoints.

```r
ggplot(my_vowels, aes(x = F2.50., y = F1.50., color = vowel)) + 
    geom_point() + 
    stat_ellipse(level = 0.67) + 
    scale_x_reverse() + scale_y_reverse() +
    theme_classic() + 
    theme(legend.position = "none")
```
![](/images/plots/vowel_plots_2/plot1.png)

Okay, so we've got some data. Let's figure out how we can plot these as trajectories.

# F1-F2 Trajectory Plot

The first plot that I want to show how to do is similar to the ones we've done before in the F1-F2 vowel space. But the only difference is that instead of dots we draw lines showing the paths of each vowel. To do that, we need to wrangle with the data a little bit, and then learn how to draw lines using `ggplot`. 

## Data processing

For now, let's just focus on one vowel, my /aɪ/ vowel, since there is some trajectory there.

```r
ay <- my_vowels %>%
    filter(vowel == "AY") %>%
    print()

## # A tibble: 149 x 13
##       id  vowel         word F1.20. F2.20. F1.35. F2.35. F1.50. F2.50.
##    <int> <fctr>        <chr>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>  <dbl>
##  1     1     AY        timer  742.2 1406.0  507.5 1022.6  580.7 1311.6
##  2    12     AY         time  991.4 1812.8  914.6 1302.7  666.4 1487.3
##  3    18     AY        prize  546.2 1105.5  577.2 1063.1  586.2 1299.1
##  4    29     AY          buy  670.6 1186.4  713.9 1264.9  511.9 1832.2
##  5    38     AY         wire  534.7  943.8  644.8 1118.6  625.2 1409.9
##  6    57     AY       flight  523.8 1108.9  534.8 1174.0  511.2 1354.7
##  7    71     AY        apply  662.8 1208.8  562.8 1001.1  635.1 1118.2
##  8    81     AY       behind  905.0 1338.9  645.4 1146.4  603.1 1162.7
##  9    91     AY firefighters 1140.9 1902.2  477.8 1070.8  585.0 1353.1
## 10    92     AY      arrived  602.1 1120.1  656.2 1116.3  687.8 1284.1
## # ... with 139 more rows, and 4 more variables: F1.65. <dbl>,
## #   F2.65. <dbl>, F1.80. <dbl>, F2.80. <dbl>
```

And we'll plot it for good measure.

```r
ggplot(ay, aes(x = F2.50., y = F1.50.)) + 
    geom_point() + 
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot2.png)

But, midpoints for diphthongs don't say as much as their entire trajectories do. We have the trajectory data, but it's spread out across all those columns (`F1.20.`, etc.). The `ggplot` function can only plot two columns at a time (the *x*- and *y*-axes), but our trajectory data is spread out over ten columns. We need to somehow reshape our data so that *all* the F1 data is in one column and *all* the F2 data is in another column.

What we need to do is essentially squish our data to cram 10 columns' worth of information into just two. Our data currently is in what's called a "wide" format because we have lots of columns. What we need to do is convert into a "tall" format, which has fewer columns, but more rows. For example, here's a simplified version of what our data looks like now:

```r
##    vowel F1.20. F1.80. F2.20. F2.80.
##    <chr>  <dbl>  <dbl>  <dbl>  <dbl>
## 1 vowel1    500    550   1000   1150
## 2 vowel2    600    650   1500   1550
## 3 vowel3    700    750   2000   2050
```

And this is what we want it to end up like:

```r
##    vowel percent    F1    F2
## *  <chr>   <chr> <dbl> <dbl>
## 1 vowel1      20   500  1000
## 2 vowel1      80   550  1150
## 3 vowel2      20   600  1500
## 4 vowel2      80   650  1550
## 5 vowel3      20   700  2000
## 6 vowel3      80   750  2050
```


So how do we do that? Unfortunately, because we have F1 *and* F2 measurements, it takes a bit more finagling than one simple function can provide, but I'll walk you through it.

<span class="sidenote">Edit: tidyr has been updated since this tutorial was posted. For more elegant and powerful code that accomplishes the same thing that this section covers, but based on a newer version of tidyr, see <a href="/blog/reshaping-vowel-formant-data-with-tidyr">this blog post.</a></span>
They key to this is to use the `gather` function from the `tidyr` library. This function---it's like black magic, I swear---takes multiple columns and condenses them down into just two. This function has two required arguments:

1. The first argument of `gather` is the arbitrary name of a new column that will contain the various column names you'll be condensing down. Since the *names* of columns we want to gather are `F1.50.`, `F2.50.`, etc. I'll call this new column `formant_percent` since it will have information about what formant at what percent into the vowel's duration we're on. 

1. The second argument name of a new column that you want to create that will contain all the values in the columns you want to combine together. Since all the cells in the `F1.50.`, `F2.50.`, etc. columns currently have formant measurements in Hertz, I like to call this column `hz`. 

Finally, as additional arguments to `gather`, you tell it what columns you would like to condense down. By default, it does all of them, but we want to keep the `vowel`, `word`, and `t` columns. You could type `F1.20., F2.20., F1.35., F2.35., F1.50., F2.50., F1.65., F2.65., F1.80., F2.80.`, but that's a lot. Instead, I'll just use the shortcut `starts_with` from `dplyr`. Since all those columns start with `"F"`, this works out well.

So the code ends up looking like this:

```r
ay %>%
    gather(formant_percent, hz, starts_with("F"))

## # A tibble: 1,490 x 5
##       id  vowel         word formant_percent     hz
##    <int> <fctr>        <chr>           <chr>  <dbl>
##  1     1     AY        timer          F1.20.  742.2
##  2    12     AY         time          F1.20.  991.4
##  3    18     AY        prize          F1.20.  546.2
##  4    29     AY          buy          F1.20.  670.6
##  5    38     AY         wire          F1.20.  534.7
##  6    57     AY       flight          F1.20.  523.8
##  7    71     AY        apply          F1.20.  662.8
##  8    81     AY       behind          F1.20.  905.0
##  9    91     AY firefighters          F1.20. 1140.9
## 10    92     AY      arrived          F1.20.  602.1
## # ... with 1,480 more rows
```

Okay, so even though there are just 149 tokens of /aɪ/, now we have 1,490 rows in our dataframe. That's because each vowel is now spread across ten rows, one for each of the ten formant measurements. It might not be clear right now that that's what it did, but we can sort the data by ID and you can get a better picture:

```r
ay %>%
    gather(formant_percent, hz, starts_with("F")) %>%
    arrange(id)

## # A tibble: 1,490 x 5
##       id  vowel  word formant_percent     hz
##    <int> <fctr> <chr>           <chr>  <dbl>
##  1     1     AY timer          F1.20.  742.2
##  2     1     AY timer          F2.20. 1406.0
##  3     1     AY timer          F1.35.  507.5
##  4     1     AY timer          F2.35. 1022.6
##  5     1     AY timer          F1.50.  580.7
##  6     1     AY timer          F2.50. 1311.6
##  7     1     AY timer          F1.65.  518.8
##  8     1     AY timer          F2.65. 1502.9
##  9     1     AY timer          F1.80.  375.6
## 10     1     AY timer          F2.80. 1757.3
## # ... with 1,480 more rows
```

There we go. Now it's clear that the ten formant measurements from this one word are now spread out in ten rows instead of the original ten columns.

So, this is *closer* to what we wanted, but we actually squished it a little too much! We actually want F1 and F2 to be in *separate* columns, and right now they're all in one. 

<div class="sidenote">It took me a lot of trial and error to figure out how to do this. Hopefully, this will save you the hassle.</div>

So one way to do this is to split up the `formant_percent` column into two: `formant` and `percent`. It's strange to have two pieces of information (what formant and the timepoint) in one column. We can use the `separate` function for this. As its first argument, we'll tell it what column we want to separate (here, it's `formant_percent`). Then, we'll add the `into` argument, which is just a list of new, arbitrary column names that the split data will now be in, which for us is `"formant"` and "`percent"`. By default, `separate` will split the text at non-word characters. So it detects the periods in `F1.50.` and assumes you want to split the string into `F1` and `50`. 

```r
ay %>%
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"))

## Warning: Too many values at 1490 locations: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
## 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...
```

Now, it's going to give you some warnings because of the extra period at the end of `F1.50.`. Essentially, it thinks that last period is a separator and it's trying to create a third column but you only told it to do two. We can add the `extra = "drop"` argument to get rid of that warning.

```r
ay %>%
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop")

## # A tibble: 1,490 x 6
##       id  vowel         word formant percent     hz
##  * <int> <fctr>        <chr>   <chr>   <chr>  <dbl>
##  1     1     AY        timer      F1      20  742.2
##  2    12     AY         time      F1      20  991.4
##  3    18     AY        prize      F1      20  546.2
##  4    29     AY          buy      F1      20  670.6
##  5    38     AY         wire      F1      20  534.7
##  6    57     AY       flight      F1      20  523.8
##  7    71     AY        apply      F1      20  662.8
##  8    81     AY       behind      F1      20  905.0
##  9    91     AY firefighters      F1      20 1140.9
## 10    92     AY      arrived      F1      20  602.1
## # ... with 1,480 more rows
```
    
Okay, so now we have one column saying what formant the measurement is for, and another column saying the timepoint. We need to reverse the squish a little bit, and put the formants into two columns. So the opposite of `gather` is `spread`, and as arguments, you first tell it what column contains the text that will serve as the column names, and then what column contains the numbers you want to be in the cells of those new columns.

```r
ay %>%
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop") %>%
    spread(formant, hz)

## # A tibble: 745 x 6
##       id  vowel  word percent    F1     F2
##  * <int> <fctr> <chr>   <chr> <dbl>  <dbl>
##  1     1     AY timer      20 742.2 1406.0
##  2     1     AY timer      35 507.5 1022.6
##  3     1     AY timer      50 580.7 1311.6
##  4     1     AY timer      65 518.8 1502.9
##  5     1     AY timer      80 375.6 1757.3
##  6    12     AY  time      20 991.4 1812.8
##  7    12     AY  time      35 914.6 1302.7
##  8    12     AY  time      50 666.4 1487.3
##  9    12     AY  time      65 591.7 1702.4
## 10    12     AY  time      80 492.3 1680.1
## # ... with 735 more rows
```

*Violà!* Our data is ready. Now each row represents a single time point, and there is one column for F1 and one column for F2. 
Do you remember why we bothered to do all that in the first place? Well, because `ggplot` needs two columns (F1 and F2) for the *x*- and *y*-axes. But our measurements were spread out over 10 columns. So we collapsed them down to two. This is the dataset we want to stick with, so we'll save it into a new dataframe called `ay_tall`. 

```r
ay_tall <- ay %>%
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop") %>%
    spread(formant, hz)
```

So now comes the fun part. Our data is set, now we just need to work in `ggplot2`. 

## Drawing lines in the F1-F2 space

The function that we need is `geom_path`. We'll use this instead of `geom_point`, which is what we used for scatterplots. To save you from scrolling, here's the code for the scatterplot:

```r
ggplot(ay, aes(x = F2.50., y = F1.50.)) + 
    geom_point() + 
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot3.png)

The last two lines transfer over just fine, so we can leave them. But we need to make a few changes to the beginning portion. First, we change the dataset from `ay` to `ay_tall`. We'll then have to change `F2.50` and `F1.50`---since we no longer have those columns---into just `F2` and `F1` which are the corresponding columns in `ay_tall`. We can change `geom_point` to `geom_path` and bada-boom!

```r
ggplot(ay_tall, aes(x = F2, y = F1)) + 
    geom_path() + 
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot4.png)

Oops. That looks awful. So what happened? As it turns out, `geom_path` literally just connected the dots in the order that they appear in the dataset. It drew one continuous line because we didn't tell it to do otherwise. What we need is an additional argument, the `group` argument, that says to draw one line for every group. So what should the groups be? Well, we want one line per token, so we want a column that has a unique vowel per token. Aha! This is why we created that `id` column at the beginning! So in the `aes` function, add `group = id` and then let'er rip.

```r
ggplot(ay_tall, aes(x = F2, y = F1, group = id)) + 
    geom_path() + 
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot5.png)

Okay, so still not great. It's just because we have a lot of data. I'll filter the data down to just the 13 tokens of me saying the word "time":

```r
time_tokens <- ay_tall %>%
    filter(word == "time")
ggplot(time_tokens, aes(x = F2, y = F1, group = id)) + 
    geom_path() + 
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot6.png)

Okay, so less messy. But still not great. For one, we can't even tell which direction the lines are going. We can add some arrows with the `arrow` argument in `geom_path`. I'll let you look up the help for arrows (hint: `?grid::arrow`), but here is some code that has worked for me. It says to put an arrow at the end of the line (as opposed to the beginning), make it a filled triangle, and make it a tenth of an inch in size. 

```r
ggplot(time_tokens, aes(x = F2, y = F1, group = id)) + 
    geom_path(arrow = arrow(ends = "last", 
                            type = "closed",
                            length = unit(0.1, "inches"))) +
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot7.png)

So we're getting there. Let's see if we can add some color. Maybe I can make the arrows go from one color to another as they progress along the vowel's trajectory. We can add another aesthetic, `color`, which will vary with the values in the `percent` column.

```r
ggplot(time_tokens, aes(x = F2, y = F1, group = id, color = percent)) + 
    geom_path(arrow = arrow(ends = "last", type = "closed", length = unit(0.1, "inches"))) +
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot8.png)

Since we created that `percent` column using `gather` and `separate` and stuff, even though they're numbers it'll treat them as factors in a categorical variable. You can easily convert it into numeric using `as.numeric` in `mutate`. I'll also change the colors to make it more striking using `scale_color_distiller` (see more info on this at [colorbrewer2.org](colorbrewer2.org)):

<div class="sidenote">The problem now is that you've got arrowheads at the end of each segment. Four lines per vowel means four arrowheads. I haven't quite figured out how to get just one arrowhead when you use color like this. </div>

```r
time_tokens <- time_tokens %>% 
    mutate(percent = as.numeric(percent))
ggplot(time_tokens, aes(x = F2, y = F1, group = id, color = percent)) + 
    geom_path(arrow = arrow(ends = "last", type = "closed", length = unit(0.1, "inches"))) +
    scale_x_reverse() + scale_y_reverse() +
    scale_color_distiller(palette = "YlOrBr") +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot9.png)



So, we've seen how to make trajectory plots with one line per token. As you can in in my data, there's a bit of noise. This is something I've noticed for a lot of the trajectory data I gather. So in the next section, we'll see how to do essentially the same thing, except we'll look at averages so we get one line per vowel. 

## Trajectories for all vowels

So maybe you're not interested in individual tokens because your plots look like spaghetti, and would rather look at all the tokens for that vowel averaged together. No problem. We'll create a new dataset that will just contain the averages for each vowel at each time point. This will involve a bit more wrangling, but the payoff is worth it.

### Data wrangling

So let's think about what kind of dataset we want. Starting with `my_vowel`, which is essentially the raw FAVE output, we have ten columns for each vowel. We want to reduce this down to just 14 rows (one for each vowel), with the average formants at each time point. To accomplish this, we can use `summarize`. This is a handy function that will help you, well, summarize your data. For example, let's say we want to get the average F1 at the 20% point for the entire dataset. We can create a new column called `mean_F1.20.` and set it equal to the mean of the values in the `F1.20.` column:

```r
my_vowels %>%
    summarize(mean_F1.20. = mean(F1.20.))

## # A tibble: 1 x 1
##   mean_F1.20.
##         <dbl>
## 1    507.6195
```

Similarly, we can do this for all the columns:

```r
my_vowels %>%
    summarize(mean_F1.20. = mean(F1.20.),
              mean_F2.20. = mean(F2.20.),
              mean_F1.35. = mean(F1.35.),
              mean_F2.35. = mean(F2.35.),
              mean_F1.50. = mean(F1.50.),
              mean_F2.50. = mean(F2.50.),
              mean_F1.65. = mean(F1.65.),
              mean_F2.65. = mean(F2.65.),
              mean_F1.80. = mean(F1.80.),
              mean_F2.80. = mean(F2.80.))

## # A tibble: 1 x 10
##   mean_F1.20. mean_F2.20. mean_F1.35. mean_F2.35. mean_F1.50. mean_F2.50.
##         <dbl>       <dbl>       <dbl>       <dbl>       <dbl>       <dbl>
## 1    507.6195    1570.627    481.2257    1534.637    453.9625    1504.667
## # ... with 4 more variables: mean_F1.65. <dbl>, mean_F2.65. <dbl>,
## #   mean_F1.80. <dbl>, mean_F2.80. <dbl>
```

Ugh, that's a lot of typing though, and it's so repetitive. Isn't there a shortcut? Yes! I only just discovered this myself, but there's a function called `summarize_at`, which will perform a function on all the columns you specify. To select what columns to use, you have to wrap it up in `vars` (I'm not sure why, but that's okay) but you can just list the columns directly like we did with `select` earlier (`F1.20.:F2.80.`). The next argument in `summarize_at` is the name of the function you want to perform on all of these. We want the average, so we'll just type `mean`. You can add additional arguments to `summarize_at`, which get passed to the function we're calling (`mean`), so for good measure I like to add `na.rm = TRUE` just to make the code a little more robust at handling `NA`s in our data.

```r
my_vowels %>%
  summarize_at(vars(F1.20.:F2.80.), mean, na.rm = TRUE)

## # A tibble: 1 x 10
##     F1.20.   F2.20.   F1.35.   F2.35.   F1.50.   F2.50.   F1.65.   F2.65.
##      <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>
## 1 507.6195 1570.627 481.2257 1534.637 453.9625 1504.667 448.0943 1508.114
## # ... with 2 more variables: F1.80. <dbl>, F2.80. <dbl>
```

So in just one line of code we can do what took us ten before. Thanks, `summarize_at`! 

But wait. We don't really want to know the average formant values for the entire dataset. What we really want is the average per vowel. How can we do the same thing per vowel? Well, luckily there's a function exactly for that, the `group_by` function. With `group_by`, we specify a column, it'll essentially divide the dataset into chunks and the `summarize_at` function will apply to each chunk. 

```r
my_vowels %>%
  group_by(vowel) %>%
  summarize_at(vars(F1.20.:F2.80.), mean, na.rm = TRUE)

## # A tibble: 14 x 11
##     vowel   F1.20.   F1.35.   F1.50.   F1.65.   F1.80.   F2.20.    F2.35.
##    <fctr>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>    <dbl>     <dbl>
##  1     AA 573.2929 583.2788 587.0858 597.9699 581.6735 1283.990 1186.2407
##  2     AE 499.1188 534.7812 560.5129 576.3792 565.8208 1714.746 1732.9178
##  3     AH 529.6103 531.5987 543.9808 525.9526 517.0308 1336.436 1319.5897
##  4     AO 549.8008 517.4583 494.4575 495.5433 491.4542 1216.797 1076.0517
##  5     AW 579.0298 616.7234 606.0255 559.1447 497.7362 1427.706 1324.6426
##  6     AY 615.7604 611.4342 583.8933 534.1812 466.9772 1324.593 1263.7034
##  7     EH 512.0701 509.8524 496.6524 507.9016 505.0321 1684.090 1682.0080
##  8     EY 420.9294 416.2355 390.6325 384.1218 377.9279 1825.290 1889.6604
##  9     IH 459.5978 438.4554 391.5892 391.3194 392.4827 1784.809 1792.2971
## 10     IY 531.0859 372.5659 299.5356 313.9888 340.6327 1968.225 2015.3127
## 11     OW 501.0168 478.6775 437.8241 419.0843 420.6838 1327.719 1171.7665
## 12     OY 416.1231 438.4769 452.9000 441.4077 412.4769 1134.077  954.2231
## 13     UH 477.7791 430.5116 395.7256 370.2721 360.1605 1390.484 1328.3279
## 14     UW 409.7179 373.2411 331.0063 323.7223 329.2982 1562.488 1492.3125
## # ... with 3 more variables: F2.50. <dbl>, F2.65. <dbl>, F2.80. <dbl>
```

<div class="sidenote">You can actually add as many columns as you want to group_by. This is good for getting averages per vowel split up by following segment, for example. For example, try raw_data %>% group_by(vowel, fol_seg) and then the summarize_at function.</div>

Note that now we have a column we didn't have before, `vowel`, which shows the group variable. Now we have the dataset we want: the average formant measurements at each time point for all 14 vowels. Let's save this as a new dataframe. 

```r
my_vowel_means <- my_vowels %>%
  group_by(vowel) %>%
  summarize_at(vars(F1.20.:F2.80.), mean, na.rm = TRUE)
```

Okay, so we have this dataset. It's essentially the parallel to `my_vowels`, only instead of individual tokens it has the averages. But, remember that we need to turn the data into a tall format for it to plot the way we want? So, we have to take this `my_vowel_means` data and turn it tall. Fortunately, you can copy and paste the code we used before:

```r
my_vowel_means_tall <- my_vowel_means %>%
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop") %>%
    spread(formant, hz) %>%
    print()

## # A tibble: 70 x 4
##     vowel percent       F1       F2
##  * <fctr>   <chr>    <dbl>    <dbl>
##  1     AA      20 573.2929 1283.990
##  2     AA      35 583.2788 1186.241
##  3     AA      50 587.0858 1131.847
##  4     AA      65 597.9699 1134.865
##  5     AA      80 581.6735 1165.675
##  6     AE      20 499.1188 1714.746
##  7     AE      35 534.7812 1732.918
##  8     AE      50 560.5129 1661.970
##  9     AE      65 576.3792 1612.307
## 10     AE      80 565.8208 1601.208
## # ... with 60 more rows
```

Okay cool. We're now ready to plot.


### Plotting

Because we've been consistent with our code and variable names and stuff, the plotting code is also going to be very similar to what we saw before. All we need to do is change two things. First, we'll change the data from `time_tokens` that we had earlier to to our new `my_vowel_means_tall`. Then, our group variable is now `vowel` instead of `id` because we want it to draw a separate line for each vowel.

```r
ggplot(my_vowel_means_tall, aes(x = F2, y = F1, group = vowel)) + 
    geom_path(arrow = arrow(ends = "last", type = "closed", length = unit(0.1, "inches"))) +
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot10.png)

Because we have so many vowels, maybe we can add a little bit of color. Let's have the color of the lines vary by the vowel as well:

```r
ggplot(my_vowel_means_tall, aes(x = F2, y = F1, group = vowel, color = vowel)) + 
    geom_path(arrow = arrow(ends = "last", type = "closed", length = unit(0.1, "inches"))) +
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot11.png)

Okay, so now we have a trajectory plot. Now, if you look closely, some of this data might not be so good. For example, look at /i/, the blue line to the far left. It's quite unlikely that my vowel actually starts that low. Perhaps there's some bad data . Looking at some of the other vowels, like /u/, the pink one in the top center, it probably doesn't start that low either.  

As it turns out, the problem was when we took the average. Because there are quite few outliers in this data still, and because outliers all tend to be towards the low front portion of the vowel space, it's pulling a lot of the measurements in that direction. The mean is quite sensitive to outliers. What we can try instead is to take the median, which is less sensitive. 

So, the only change that we need to do is in `summarize_at` where we change the `mean` to `median`. In fact, we don't even need to create a new object if you don't want: we can pipe everything all at once, straight into `ggplot`! So in this code, I take the beginning dataset, create a summary of it, and then pipe it into `ggplot`. Note that when we pipe something into `ggplot`, you don't need to specify the data anymore (the first argument), so I leave that blank.

```r
my_vowels %>%
    # Summarize by vowel
    group_by(vowel) %>%
    summarize_at(vars(F1.20.:F2.80.), median, na.rm = TRUE) %>%
    
    # Turn it into tall data
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop") %>%
    spread(formant, hz) %>%
    
    # Plot it!
    ggplot(aes(F2, F1, color = vowel, group = vowel)) +
    geom_path(arrow = arrow(ends = "last", type = "closed", length = unit(0.1, "inches"))) +
    scale_x_reverse() + scale_y_reverse() +
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot12.png)

The result is a much cleaner picture: /i/ and /e/ sort of hook to the left while /u/ and /o/ start centralized and move to the back. My diphthongs follow their expected trajectories by starting centralized and sort of scooping down and up again. While *cot* and *caught* are somewhat close at their onsets, they have very different trajectories, clearly differentiating the phonemes. Finally, /ɪ/ is quite monophthongal, showing very little change in trajectory at all.

Right now, things are a little hard to read because there are 14 colors and a lot of them are very similar to one another. Also, the legend has the vowels in alphabetical order, which isn't too helpful. In the previous tutorial, we went over how to change that, so I won't comment on it much here. The only difference is I'll set the factor levels using `mutate` in the `dplyr` package rather than using base R:

```r
my_vowels %>%
    # Relevel the vowels (random order to spread out similar colors)
    mutate(vowel = factor(vowel, levels = c("IY", "EH", "AO", "UW", 
                                            "AW", "IH", "AE", "OW", 
                                            "AH", "OY", "EY", "AA", "UH", "AY"))) %>%
    
    # Summarize by vowel
    group_by(vowel) %>%
    summarize_at(vars(F1.20.:F2.80.), median, na.rm = TRUE) %>%
    
    # Turn it into tall data
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop") %>%
    spread(formant, hz) %>%
    
    # Plot it!
    ggplot(aes(F2, F1, color = vowel, group = vowel)) +
    geom_path(arrow = arrow(ends = "last", type = "closed", length = unit(0.1, "inches"))) +
    scale_x_reverse() + scale_y_reverse() +
    # Put the vowels in a meaningful order in the legend
    scale_color_discrete(breaks = c("IY", "IH", "EY", "EH", "AE", "AA", 
                                    "AO", "OW", "UH", "UW", 
                                    "AH", "AY", "AW", "OY")) + 
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot13.png)

This is still suboptimal. It would be best to just have the name of the vowel on the plot itself. Fortunately, we saw how to do that in the previous workshop. We'll have to create a different dataset and use it in `geom_label`. Instead of one mega string of piped functions, I'll save the main dataset into a new dataframe called `my_vowel_medians_tall`. I'll then use that but just select the onset (the 20% mark) to create `labels_at_onset`, a dataframe specifically for plotting the labels.

```r
my_vowel_medians_tall <- my_vowels %>%
    # Relevel the vowels
    mutate(vowel = factor(vowel, levels = c("IY", "EH", "AO", "UW", 
                                            "AW", "IH", "AE", "OW", 
                                            "AH", "OY", "EY", "AA", "UH", "AY"))) %>%
    
    # Summarize by vowel
    group_by(vowel) %>%
    summarize_at(vars(F1.20.:F2.80.), median, na.rm = TRUE) %>%
    
    # Turn it into tall data
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop") %>%
    spread(formant, hz)
labels_at_onset <- my_vowel_medians_tall %>%
    filter(percent == 20)
```

Now, when we go to plot it, we add the `geom_label` function and use the `labels_at_onset` as the dataframe. The `F1`, `F2`, and color columns still work, so all we need to add is an aesthetic for the label. I also took out `scale_color_discrete` since we're removing the legend, which is done with `guides(color = FALSE)`. 

```r    
ggplot(my_vowel_medians_tall, aes(F2, F1, color = vowel, group = vowel)) +
    geom_path(arrow = arrow(ends = "last", type = "closed", length = unit(0.1, "inches"))) +
    geom_label(data = labels_at_onset, aes(label = vowel)) + 
    scale_x_reverse() + scale_y_reverse() +
    theme_classic() + 
    guides(color = FALSE)
```
![](/images/plots/vowel_plots_2/plot14.png)

So that's the end of this portion of the tutorial. We've seen how to wrangle your data to get it to work for drawing paths. We've seen how to draw all observations or how to summarize them and plot just one line per vowel. Pretty handy.

# Praat-like plots

So the plots we've seen so far were in the F1-F2 vowel space. But sometimes you want your plots to look more like they do in Praat, where the *x*-axis represents time, and the *y*-axis represents frequency in Hertz, and the formants move across from left to right. This is certainly possible to do in R, but, you guessed it, it's going to take some data wrangling again. 

## Plotting all data

Just like we did above, we'll start with plotting all tokens of /aɪ/, which is going to be a bit messy. Then we'll see how to summarize the data and plot just the medians for each vowel.

So last time, we took `my_vowels` and compressed the 10 columns into 1, but that was too far so we used `spread` to spread them out into F1 and F2 columns. Well, as it turns out, to make this Praat-like plot, we want that super tall, compressed version. 

All of this code you've seen before, so it should be relatively clear what is happening. We'll start with the first dataset, `my_vowel`, only keep the tokens of "AY", then we'll smush all ten columns into one, separate out the formant and percent, and then sort it by the ID. We'll save this as a new dataframe, `ay_very_tall`, because it's a "very tall" version of the data.

```r
ay_very_tall <- my_vowels %>%
    filter(vowel == "AY") %>% 
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop") %>%
    arrange(id) %>%
    print()
```

Once we've got this, plotting is pretty straightforward. For the *x*-axis, we want normalized time, which is in the `percent` column. For the *y*-axis, we want frequency in Hz, so we'll use the `hz` column. We want to use `geom_path` again, and for maximal clarity, we'll make the two formants different colors. 

```r
ggplot(ay_very_tall, aes(x = percent, y = hz, group = id, color = formant)) +
    geom_line() + 
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot15.png)

Ah, crud. What happened? As it turns out, we're plotting one line for each token because the `group` variable is based on the `id` column, which has a unique value per token. That's not actually what we want though, is it? When we did the F1-F2 plot, we wanted a single line per token, sure, but in this plot, we actually want *two* lines per token, the F1 and the F2. In order for `ggplot` to draw separate lines, you're going to need a column that has a unique value for every line you want to draw. So we want a different line for each formant for each token. In other words, we need to create what I like to think of as a "formant id" column. (And yes, that means just a little bit more data wrangling.) 

The way I like to do this is to use the `unite` function, which simply concatenates the values in multiple columns for each row. Since each token has its own id already, the `id` column, we can combine this with the formant in the `formant` column. To use `unite`, you first type the name of the new column you want to create (I did `formant_id`), then whatever columns you want to combine. By default, they will concatenate without any intervening material, but I like underscores, so I add `sep = "_"` to put an underscore between them. Also, by default, it'll remove the contributing columns. We don't really need the token id column anymore, but I like to keep the formant column so I can color the lines by formant, so we'll add `remove = FALSE` to `unite`. Tag all this on to the end of the list of functions and you get the following block of code.

```r
ay_very_tall <- my_vowels %>%
    filter(vowel == "AY") %>% 
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop") %>%
    arrange(id) %>%
    unite(formant_id, formant, id, sep = "_", remove = FALSE) %>%
    print()

## # A tibble: 1,490 x 7
##    formant_id    id  vowel  word formant percent     hz
##  *      <chr> <int> <fctr> <chr>   <chr>   <chr>  <dbl>
##  1       F1_1     1     AY timer      F1      20  742.2
##  2       F2_1     1     AY timer      F2      20 1406.0
##  3       F1_1     1     AY timer      F1      35  507.5
##  4       F2_1     1     AY timer      F2      35 1022.6
##  5       F1_1     1     AY timer      F1      50  580.7
##  6       F2_1     1     AY timer      F2      50 1311.6
##  7       F1_1     1     AY timer      F1      65  518.8
##  8       F2_1     1     AY timer      F2      65 1502.9
##  9       F1_1     1     AY timer      F1      80  375.6
## 10       F2_1     1     AY timer      F2      80 1757.3
## # ... with 1,480 more rows
```

Now we can use the same `ggplot2` code, but use the `formant_id` instead of `id`, and we get what we want:

```r
ggplot(ay_very_tall, aes(x = percent, y = hz, group = formant_id, color = formant)) +
    geom_line() + 
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot16.png)

Again, messy data, but you can see the resemblance with how Praat shows the data. The general trend is kinda there though: the vowel starts with a high F1, gets slightly higher, and then comes down to form a higher vowel. Meanwhile, F2 starts slow and moves high, creating a back-to-front trajectory in the vowel space. 

In the next section, we'll summarize the data again by plotting just the median for each vowel and we can see how to plot all the vowels at once.

## Plotting summarized data

Just like last time, we'll have to do some data wrangling to get what we want, but fortunately there's nothing new in this section: it's just applying code that we've seen before. 

```r
my_vowel_medians_very_tall <- my_vowels %>%
    
    # Take the median of each vowel at each timepoint
    group_by(vowel) %>%
    summarize_at(vars(F1.20.:F2.80.), median, na.rm = TRUE) %>%
    
    # Convert it into a "very tall" format
    gather("formant_percent", "hz", starts_with("F")) %>%
    separate(formant_percent, into = c("formant", "percent"), extra = "drop") %>%
    
    # Create the formant id column
    unite(formant_id, formant, vowel, sep = "_", remove = FALSE) %>%
    print()

## # A tibble: 140 x 5
##    formant_id  vowel formant percent    hz
##  *      <chr> <fctr>   <chr>   <chr> <dbl>
##  1      F1_AA     AA      F1      20 540.2
##  2      F1_AE     AE      F1      20 483.1
##  3      F1_AH     AH      F1      20 500.8
##  4      F1_AO     AO      F1      20 544.2
##  5      F1_AW     AW      F1      20 561.1
##  6      F1_AY     AY      F1      20 546.2
##  7      F1_EH     EH      F1      20 467.1
##  8      F1_EY     EY      F1      20 411.1
##  9      F1_IH     IH      F1      20 392.6
## 10      F1_IY     IY      F1      20 363.6
## # ... with 130 more rows
```

And now we plot it. I'll go ahead and manually change the order of the vowels too, like I did before, with `scale_color_discrete`.

```r
ggplot(my_vowel_medians_very_tall, aes(x = percent, y = hz, color = vowel, group = formant_id)) +
    geom_line() + 
    scale_color_discrete(breaks = c("IY", "IH", "EY", "EH", "AE", "AA", 
                                    "AO", "OW", "UH", "UW", 
                                    "AH", "AY", "AW", "OY")) + 
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot17.png)

This looks a lot cleaner than the previous plot. But we're still left with the same problem of just too many darn vowels. For this one what I like to do is to *facet* the plot. Essentially, have each vowel as its own tile, forming a bit of a mosaic. To do that, we use the `facet_wrap` function, and then after a tilde, the name of the column you want to facet the plot by.

```r
ggplot(my_vowel_medians_very_tall, aes(x = percent, y = hz, color = vowel, group = formant_id)) +
    geom_line() + 
    facet_wrap(~vowel) + 
    scale_color_discrete(breaks = c("IY", "IH", "EY", "EH", "AE", "AA", 
                                    "AO", "OW", "UH", "UW", 
                                    "AH", "AY", "AW", "OY")) + 
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot18.png)

Here, color and legend are superfluous and you can remove them if you want, but the color at least makes it look pretty I guess. This is a handy way to quickly compare the formants for all the vowels at once. 

In fact, if you want to create a new column specifically meant for faceting, it's relatively easy to and might make a more meaningful plot. Here, I create a new column called `class`, which tells whether the vowel is tense, lax, or a diphthong. I do this using the `case_when` function, which is essentially a way to create a stack of if-else-if statements. So I say, okay, if the vowel is one of these five, in the new `class` column, put the value as "tense". If it's one of these six vowels, call it "lax", and if it's one of those three call it a "diphthong." The only change in the plot code is in the `facet_wrap`, where I take out `vowel` and put in this new `class` column.

```r
my_vowel_medians_very_tall %>%
    mutate(class = case_when(vowel %in% c("IY", "EY", "AE", "OW", "UW") ~ "tense",
                             vowel %in% c("IH", "EH", "AA", "AO", "UH", "AH") ~ "lax",
                             vowel %in% c("AY", "AW", "OY") ~ "diphthong")) %>%
    ggplot(aes(x = percent, y = hz, color = vowel)) +
    geom_line(aes(group = formant_id)) + 
    facet_wrap(~class) + 
    scale_color_discrete(breaks = c("IY", "IH", "EY", "EH", "AE", "AA", 
                                    "AO", "OW", "UH", "UW", 
                                    "AH", "AY", "AW", "OY")) + 
    theme_classic()
```
![](/images/plots/vowel_plots_2/plot19.png)

The result is a pretty cool plot that lets you easily compare the different classes of vowels. /aɪ/ and /aʊ/ have roughly the same F1 but have very different F2 trajectories. The lax vowels are relatively monophthongal while the tense vowels are more peripheral and dynamic. Also, my /æ/ and /u/ start off with the same F2, which is pretty cool. 

# Conclusion

In this tutorial, I've covered a lot of material. The goal was to show you how to do two different ways to plot vowel trajectories in R. We first saw how to do trajectories in the F1-F2 space, so you can see lines overlaid on a traditional vowel plot. Then we saw how to imitate Praat and show the formants trajectories independently of each other. 

By doing so though, I've introduced a *lot* of data wrangling code, all of it coming from `dplyr` or `tidyr`, which are part of the `tidyverse`. I wasn't able to stop and explain them all in depth, but the list of new functions was `filter`, `mutate`, `select`, `arrange`, `case_when`, `gather`, `spread`, `separate`, `unite`, `group_by`, `summarize`, and `summarize_at`. *These are all very handy functions to know!* I use them every day and I recommend you get comfortable enough with them to be able to use them without even thinking about it. They can really help you as you wrangle other datasets. Again, I can't recommend enough the book [*R for Data Science*](http://r4ds.had.co.nz), which is where I learned all this. 

By the way, I've seen other similar tutorials for how to do trajectory plots in R (and there are likely more). One is [*Vowel formants trajectories and tidy data*](https://stefanocoretta.github.io/post/vowel-formants-trajectories-and-tidy-data/) by Stefano Coretta, which is brief, but covers how to work with data that you may have gotten through a Praat script. Another is [*Working with Vowels Part 2: Plotting vowel data*](http://www.mattwinn.com/tools/HB95_2.html) by Matthew Winn which covers things that this tutorial did not cover like labels and animations. A third is [George Bailey's](http://www-users.york.ac.uk/~gb1055/) excellent [workshop materials](http://www-users.york.ac.uk/~gb1055/sociophonetics_workshop/index.html) for visualizing vowel formant data. There is some overlap with all of these, but hopefully my little tutorial can help you out too.

In the next tutorial, we'll look at how to augment the plots covered in Part 1 and Part 2 by adding labels, and overlaying means. But also how to essentially plot many different datasets all in one, creating complicated but very cool plots. 
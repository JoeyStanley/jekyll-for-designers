---
layout: post
title: Curved Text in ggplot2 with geomtextpath
date: 2021-12-17 22:00:00 -0400
tags: [Data Viz, How-to Guides, Side Projects, Skills]
excerpt: I recently saw a tweet by @timelyportfolio that mentions an R package, geomtextpath, by Allan Cameron. The function overlays text over curved lines, giving you the possibility to add nice labels to the data. Since I plot vowel trajectories a lot and have to label them, I thought I'd try it on some vowel data to see how well it works. 
---

I recently saw a tweet by @timelyportfolio that mentions an R package, [`geomtextpath`](https://github.com/AllanCameron/geomtextpath), by Allan Cameron. The function overlays text over curved lines, giving you the possibility to add nice labels to the data. 

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">this <a href="https://twitter.com/hashtag/rstats?src=hash&amp;ref_src=twsrc%5Etfw">#rstats</a> ggplot2 geom is really nice <a href="https://t.co/URYoZEW8uc">https://t.co/URYoZEW8uc</a>. I have been waiting for something like for a very long time. <a href="https://t.co/HCfxdGQmMt">pic.twitter.com/HCfxdGQmMt</a></p>&mdash; timelyportfolio (@timelyportfolio) <a href="https://twitter.com/timelyportfolio/status/1469683836107866120?ref_src=twsrc%5Etfw">December 11, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Since I plot vowel trajectories a lot and have to label them, I thought I'd try it on some vowel data to see how well it works. 


```r
library(tidyverse)
library(joeysvowels)
library(joeyr)

# remotes::install_github("AllanCameron/geomtextpath")
library(geomtextpath)
```

## Data prep

I'll use some trajectory data I have in my [`joeysvowels`](https://joeystanley.github.io/joeysvowels/) package. I'll mostly work with data from the `coronals` object. This contains a bunch of formant trajectory data from me reading all my vowels flanked by a bunch of combinations of coronal consonants (they were mostly nonce words like /sneɪz/, /nɔdz/, and /dʊz/). 

```r
coronals

## # A tibble: 14,446 × 13
##    vowel_id start   end     t percent    F1    F2    F3    F4 word  pre   vowel
##       <dbl> <dbl> <dbl> <dbl>   <dbl> <dbl> <dbl> <dbl> <dbl> <chr> <chr> <fct>
##  1        1  2.06  2.41  2.06       0  387. 1701. 2629. 3164. snoʊz sn    GOAT 
##  2        1  2.06  2.41  2.07       5  483. 1591. 2454. 3310. snoʊz sn    GOAT 
##  3        1  2.06  2.41  2.09      10  525. 1466. 2526. 3343. snoʊz sn    GOAT 
##  4        1  2.06  2.41  2.13      20  530. 1297  2616. 3330  snoʊz sn    GOAT 
##  5        1  2.06  2.41  2.14      25  497. 1223. 2562. 3280. snoʊz sn    GOAT 
##  6        1  2.06  2.41  2.16      30  461. 1172. 2559. 3252  snoʊz sn    GOAT 
##  7        1  2.06  2.41  2.18      35  414. 1120  2625. 3247. snoʊz sn    GOAT 
##  8        1  2.06  2.41  2.20      40  423  1072. 2655. 3175. snoʊz sn    GOAT 
##  9        1  2.06  2.41  2.22      45  396. 1074  2623. 3248. snoʊz sn    GOAT 
## 10        1  2.06  2.41  2.23      50  368. 1018. 2602. 3168. snoʊz sn    GOAT 
## # … with 14,436 more rows, and 1 more variable: fol <chr>
```

There's a lot of data and I don't need to plot all trajectories of all observations, so I'll boil them down to one trajectory per vowel by taking the median per timepoint per vowel. Also, because these vowels are flanked by coronals, the edges of the vowel trajectories, particularly the second halves, all converged towards the high front portion of the vowel space, so I'll just get a middle portion of the trajectory.

```r
avg_trajs <- coronals %>%
  group_by(vowel, percent) %>%
  summarize(across(c(F1, F2), median)) %>%
  filter(percent >= 10, percent <= 60) %>%
  print()

## # A tibble: 143 × 4
## # Groups:   vowel [13]
##    vowel percent    F1    F2
##    <fct>   <dbl> <dbl> <dbl>
##  1 LOT        10  646. 1279.
##  2 LOT        15  660. 1240.
##  3 LOT        20  664. 1170.
##  4 LOT        25  658. 1175.
##  5 LOT        30  645  1141.
##  6 LOT        35  640. 1136.
##  7 LOT        40  631. 1154.
##  8 LOT        45  634. 1168 
##  9 LOT        50  629. 1167.
## 10 LOT        55  632. 1178.
## # … with 133 more rows
```

## Basic Trajectory Plots

Here's what this data looks like.

```r
my_arrow <- arrow(ends = "last", type = "closed", angle = 25, length = unit(0.3, "cm"))
ggplot(avg_trajs, aes(F2, F1, color = vowel, group = vowel)) +
  geom_path(arrow = my_arrow) + 
  scale_x_reverse() +
  scale_y_reverse() + 
  theme_joey_legend()
```
<img width = "85%" src="/images/plots/geomtextpath/basic.png">


Okay, normally what I'd do here is create a separate dataset that has just the vowel onsets and plot it separately like this. I can remove the legend too at this point since it contributes no new information.

```r
avg_trajs_onsets <- avg_trajs %>%
  filter(percent == min(percent))
ggplot(avg_trajs, aes(F2, F1, color = vowel, group = vowel)) +
  geom_path(arrow = my_arrow) + 
  geom_text(data = avg_trajs_onsets, aes(label = vowel)) + 
  scale_x_reverse() +
  scale_y_reverse() + 
  theme_joey() + 
  theme(legend.position = "none")
```
<img width = "85%" src="/images/plots/geomtextpath/basic_labels.png">


However, with this `geomtextpath` package, I might be able to save myself the work of creating a new dataframe. 

## F1-F2 plots

Okay, now we're ready to try out `geomtextpath`. Here's what it looks like with the function in its most basic form.

```r
ggplot(avg_trajs, aes(F2, F1, color = vowel)) +
  geom_textpath(aes(label = vowel), arrow = my_arrow) + 
  scale_x_reverse() +
  scale_y_reverse() + 
  theme_joey() + 
  theme(legend.position = "none")
```
<img width = "85%" src="/images/plots/geomtextpath/out_of_box.png">

So, a few things to note here. The behavior of the function is most easily seen in <sc>goose</sc> and <sc>choice</sc>. It chooses a spot in the middle of the line and overlays the label. The line itself is cut into two pieces (a behavior that can be overriden with `cut_path = FALSE` if I wanted) with the label centered in the gap. Overall, I'm not a huge fan of the out-of-the-box look, but we can adjust things to make it look better.

The one thing that I'm also not a huge fan of is that the `arrow` argument that I aded to my previous plots doesn't seem to work. So, even though the code for adding an arrow is there, it doesn't seem to do anything. It's not a big deal because I can plot the line with the arrow as I normally would with `geom_path` and then plot the labels as a separate layer with `geom_textpath` and hide the lines associated with that layer with `include_line = FALSE`.

```r
ggplot(avg_trajs, aes(F2, F1, color = vowel)) +
  geom_path(arrow = my_arrow) + 
  geom_textpath(aes(label = vowel), include_line = FALSE) + 
  scale_x_reverse() +
  scale_y_reverse() + 
  theme_joey() + 
  theme(legend.position = "none")
```
<img width = "85%" src="/images/plots/geomtextpath/with_arrows.png">

I actually kind of prefer this method because it makes it easier for me to control the lines and labels individually.

The problem now is that the lines go through the label, which isn't great. We can adjust the vertical position of the label with the `vjust` argument, which will position it slightly above the line. While we're adjusting the position of the label, I'll set `hjust` to `0`, which will move it to the onset of the vowel.

```r
ggplot(avg_trajs, aes(F2, F1, color = vowel)) +
  geom_path(arrow = my_arrow) +
  geom_textpath(aes(label = vowel),
                hjust = 0,
                vjust = -0.5,
                include_line = FALSE) +
  scale_x_reverse() +
  scale_y_reverse() + 
  theme_joey() + 
  theme(legend.position = "none") 
```
<img width = "85%" src="/images/plots/geomtextpath/vjust.png">

For some vowels, this looks pretty well. However, because of the jaggedness of the lines, the label itself is going to be pretty jagged. Lines like <sc>trap</sc>, <sc>mouth</sc>, <sc>price</sc>, <sc>kit</sc>, and <sc>fleece</sc> are illegible. 

<!--There is a `geom_textsmooth` which is the `geomtextpath` equivalent of `stat_smooth`, which might be good, except like `stat_smooth` it only fits a smoothed line from left to right, and most of these trajectories double back on themselves and don't quite work.-->

Fortunately, the argument `keep_straight`, when set to `TRUE`, will force the text to be straight. According to the documentation, it is helpful for "noisy paths", which I think perfectly applies to our this data.

```r
ggplot(avg_trajs, aes(F2, F1, color = vowel)) +
  geom_path(arrow = my_arrow) +
  geom_textpath(aes(label = vowel),
                hjust = 0,
                vjust = -0.5,
                include_line = FALSE,
                keep_straight = TRUE) +
  scale_x_reverse() +
  scale_y_reverse() + 
  theme_joey() + 
  theme(legend.position = "none")
```
<img width = "85%" src="/images/plots/geomtextpath/hjust.png">

This is better, but I'm not a fan of how some of them cross the lines themselves (like <sc>thought</sc>, <sc>fleece</sc>, <sc>choice</sc>). 

A potential solution I came up with is to fit a GAM to the data. Here's some quick and dirty code that fits GAMs to both F1 and F2 for each vowel, extracts predicted measurements, and plots them instead. 

```r
library(mgcv)
library(itsadug)
coronals_gam_preds <- coronals %>%
  # Subset the data
  filter(percent >= 10, percent <= 60) %>%
  select(vowel, percent, F1, F2) %>%
  # Add an id column
  rowid_to_column() %>%
  # Reshape it to a "long" formant
  pivot_longer(cols = c(F1, F2), names_to = "formant", values_to = "hz") %>%
  # Group for the GAMs
  group_by(vowel, formant) %>%
  nest() %>%
  # Fit the GAM and extract predicted measurements
  mutate(mdl = map(data, ~bam(hz ~ percent + s(percent), data = .)),
         preds = map(mdl, ~get_predictions(., cond = list(percent = 10:60, print.summary = FALSE)))) %>%
  # Post-processing
  select(-data, -mdl) %>%
  unnest(preds) %>%
  rename(hz = fit) %>%
  select(-CI) %>%
  pivot_wider(names_from = formant, values_from = hz) %>%
  print()

ggplot(coronals_gam_preds, aes(F2, F1, color = vowel)) +
  geom_path(arrow = my_arrow) +
  geom_textpath(aes(label = vowel),
                vjust = -0.5,
                hjust = 0,
                include_line = FALSE,
                keep_straight = TRUE) +
  scale_x_reverse() +
  scale_y_reverse() + 
  theme_joey() + 
  theme(legend.position = "none")
```
<img width = "85%" src="/images/plots/geomtextpath/gams.png">

The results show a smoother line and indeed smoother labels, but there are still some positions I'm not a fan of, like <sc>kit</sc>, <sc>choice</sc>, and <sc>thought</sc>.

Overall, I'm haven't *quit*e found a plot that I'm satisfied with. Perhaps with a different dataset it could work, but at least for this one, this F1-F2 plot isn't quite doing what I was hoping.



## Spectrogram plots

Another potenial use for `geomtextpath` is to create spectrogram-like plots. Here's a simple one with all the vowels plotted, split up by formant.

```r
avg_trajs %>%
  pivot_longer(cols = c(F1, F2), names_to = "formant", values_to = "hz") %>%
  unite(traj_id, vowel, formant, remove = FALSE) %>%
  ggplot(aes(percent, hz, color = vowel)) +
  geom_textpath(aes(group = traj_id, label = vowel), vjust = -0.1, hjust = 0.3) + 
  scale_y_log10() + 
  facet_wrap(~formant, nrow = 1, scales = "free") + 
  theme_joey() + 
  theme(legend.position = "none")
```
<img width = "85%" src="/images/plots/geomtextpath/spectrogram.png">

It's not bad. With the help of color to help tell the lines apart, I think it does a pretty good job to be honest. Although, I'll mention that I tried this plot out with a couple different aspect ratios, and taller ones work better.



## Ellipses?

`geomtextpath` has a few other functions, including `geom_textdensity`, `geom_textsmooth`, `geom_textcontour`, `geom_textdensity2d`, which correspond to adding labels to density plots (`geom_density`), 2D density plots (`geom_textdensity2d`), smooths (`stat_smooth`), and contour plots (`geom_contour`). What I would love to see is a function that correspoinds to `stat_ellipse`. I often make plots like this where I add dots and ellipses in the same plot and then add labels at vowel averages with a separate data frame:

```r
midpoints_means <- midpoints %>%
  group_by(vowel) %>%
  summarize(across(c(F1, F2), mean))
ggplot(midpoints, aes(F2, F1, color = vowel)) + 
  geom_point(alpha = 0.25) + 
  stat_ellipse(level = 0.67) + 
  geom_text(data = midpoints_means, aes(label = vowel)) + 
  scale_x_reverse() + 
  scale_y_reverse() + 
  theme_joey() + 
  theme(legend.position = "none")
```
<img width = "85%" src="/images/plots/geomtextpath/ellipses.png">

It would be pretty cool to see these ellipses get the labels on them rather than plotting at the centers.

## Conclusion

That's all. I just wanted to try something new. Maybe this might be useful to you as you plot your vowel trajectory data.
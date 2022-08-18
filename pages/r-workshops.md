---
layout: aside
title: R Workshops
redirect_from: 
    - "/r2018"
    - "/r"
---

# R Workshops

In grad school I offered workshops on a variety of topics relating to R between Fall 2017 and Spring 2020. They were held Fridays at 3:30 in the DigiLab at the UGA Main Library. Videos for some of these are available [here](https://digilabuga.github.io/Resources/VideoTutorials.html). Below you will find materials for these workshops. I hope they are thorough enough that you can make use of them as stand-alone documents. 

The data used in these workshops can be found at my [datasets page](/data).

If you are planning on attending a workshop in the near future, please see the [setup instructions](/pages/setup) to help you get R and RStudio installed on your computer.

Note: If you attended these workshops, please consider giving me some feedback with [this anonymous survey](/survey) so I can know how to improve. 

<br/>

## *Intro to R ([Part 1](/downloads/180119-intro-to-r-part1) and [Part 2](/downloads/180126-intro-to-r-part2))*

In this first workshop I cover the basics of R itself. I talk about the differences between R and RStudio and I help folks get both installed and running on their computers. We create a simple "Hello, World!" script using R. Part 2 covers the basics of the R language. It is also be a very simple introduction to some core computer coding concepts like declaring variables and variable types.

*Additional Materials:* You may also be interested in [a PDF of the slides I used in Part 1](/downloads/180119-intro-to-r-part1.pdf) or [a PDF version of Part 2's handout](/downloads/180126-intro-to-r-part2.pdf). An older version that combines elements of both parts can be found here as a [PDF](/downloads/170912-intro-to-r-handout.pdf) or [RMarkdown](/downloads/170913-intro_to_R) file.

<br/>

## *Visualizations with ggplot2* <!-- This section is also duplicated in pages/dataviz -->

ggplot2 is a widely used package that allows for high-quality visualizations. These workshops take you from installation to pretty advanced topics.

### [Part 1: Intro to ggplot2](/downloads/190821-intro_to_ggplot2)

In Part 1 of this workshop I cover the basic syntax and how to make some simple types of plots. 

### [Part 2: Extending your ggplot2 skills](/downloads/190828-ggplot2_intermediate)

Often you'll want to customize your plots in some way. So, in Part 2, we cover how to mess with properties of the plots like the axes, colors, and legends to make the plot work better for you. 

### [Supplement to Part 2](/downloads/190828-ggplot2_supplement)

Apparently I had a lot to say about how to extend your ggplot2 skills, so I ended up creating a supplement with lots of additional detail on how to modify your plots. This handout will vary from time to time as I add to it when I learn new things or remove sections to incorporate them into future workshops.

### [Custom Themes](/downloads/190904-ggplot2_custom_themes)

Based on a popular [blog post](/blog/2016-12-16-custom-themes-in-ggplot2) I wrote, this workshop wraps all customization methods together and shows how to create your own themes.

### [Supplement: A detailed look at `ggplot2::theme`](/downloads/190904-ggplot2_theme)

As I was preparing for the custom themes workshop, I got a little carried away illustrating all the components of the `theme` function. I decided to simplify that portion of the workshop and create this separate handout that just focuses on `theme`. It is not yet finished, but it may be of some help to people (including myself!).


*Additional Materials:* You may also be interested in the 2018 versions of some of these workshops that I gave (Part 1 [Rmarkdown](/downloads/180216-ggplot2-part1) and [PDF](/downloads/180216-ggplot2-part1.pdf) and Part 2 [RMarkdown](/downloads/180223-ggplot2-part2) and [PDF](/downloads/180223-ggplot2-part2.pdf)). An older version from 2017 that combines elements of Parts 1 and 2 can be found here as a [PDF](/downloads/171012-ggplot2_handout.pdf) or [RMarkdown](/downloads/171012-ggplot2) file.

<br/>

## Introduction to the Tidyverse ([Part 1](/downloads/180302-tidyverse_part1) and [Part 2](/downloads/180323-tidyverse_II))

The tidyverse is a suite of packages that includes `dplyr` and `tidyr` which help you wrangle your data. In this two-part workshop, we learn some of the common functions in the tidyverse and compare them to base R, showing that there are multiple ways to accomplish the same task in R. Part 2 in particular looks at how to reshape and transform your data, merging it with other dataset, and other super useful and powerful tools.

*Additional Materials:* You may also be interested in PDFs for [Part 1](/downloads/180302-tidyverse_part1.pdf) and [Part 2](/downloads/180323-tidyverse_II.pdf). An older version that combines elements of both parts can be found here as a [PDF](/downloads/171110-tidyverse_handout.pdf) or [RMarkdown](/downloads/171110-tidyverse) file.

<br/>

## [An Intro to RMarkdown](/downloads/180309-rmarkdown)

R Markdown is a way to create different types of documents using R (pdfs, word files, html files). In this one-day crash course, I show how to make R Markdown files and the kinds of things they would be useful for.

*Additional Materials:* If you'd rather use a PDF of the workshop materials, there's one available [here](/downloads/180309-rmarkdown.pdf). 

<br/>

## [Introduction to Shiny](https://joeystanley.shinyapps.io/intro_to_shiny/)

Shiny is an R package that allows you to make your own interactive web pages. An entire semester could be devoted to Shiny and there's a bit of a learning curve, especially if you haven't used HTML before. This two-day workshop covers just the essentials.

Note that the materials for this workshop include interactive shiny elements, which means I can't host them on my own website. So instead, they're hosted on Shiny's free server space. But this comes with a major drawback: they're only available for 25 user-hours a month. So, if the link above does not work for you, try again in a week or so. Sorry for the inconvenience.

<!--
*Special topics: Regression and mixed-effects modeling* (April 6): In more and more fields, quantitative analysis is the norm. I can't begin to cover everything about fitting statistical models to your data, but I'll cover some introductory concepts to hopefully guide you in the right direction for further study.

*Special topics: Network analysis* (April 13): Network analysis is a fascinating field on its own, and learning to create and analyze visualizations of network data can be helpful for some studies. This workshop will cover some basic visualizations and statistical analysis of network data.  

*Special topics: Working with text* (April 20): Most topics in this series have covered numbers and how to work with them. In this final presentation, I introduce the `stringr` package (part of the Tidyverse suite), and how you can use it to your advantage when working with text in R. 

*Visualization III: Advanced topics in ggplot2* (TBD): In this workshop we go beyond the simple customization techniques and move on to modifying many other aspects of the plot. Time permitting, I'll show how to create your own themes so that they match your powerpoint themes to create a more appealing presentation.
-->


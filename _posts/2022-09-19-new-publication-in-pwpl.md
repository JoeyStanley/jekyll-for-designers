---
layout: post
title: "New publication in the Penn Working Papers in Linguistics"
date: 2022-09-19 22:11:00 -0600
tags: [Methods, Research, Publications, Vowel Overlap]
---

I've just been informed that a manuscript I submitted to the *Penn Working Papers in Linguistics* has been published! It's called "Order of Operations in Sociophonetic Analysis" and is available [here](https://repository.upenn.edu/pwpl/vol28/iss2/17/). In a nutshell, I discuss the various processing steps that are typical of a sociophonetic analysis an I show that changing the order that you run them can have a non-negligible impact on the overall results. I end the paper with a recommended order that we can use.

## Overview

I originally started this paper because I noticed, by accident, that I was getting different results after moving some code around. I was in the middle of an analysis (of the very dataset I use in the paper) and had already gotten some results, but I had to change a few things in the processing and ended up moving a chunk of code up a little bit so that it happened before another chunks. Lo and behold, the results changed. Nothing about the underlying data changed, and nothing about the functions I ran changed---other than their order.

So this sent me down a bit of a rabbit hole. I identified seven processing steps that I did in my analysis pipeline. I then wrote some code that arranged those seven steps into all possible orderings, all 5,040 of them. I then took the exact same input dataset, and processed it 5,040 times, once for each permutations. With each resulting spreadsheet, I then calculated things like how shifted or overlapped people's vowels were. Again, to be clear, the underlying data was identical, and the functions I ran were identical. The only thing that changed was the order that I ran them. 

Well, as you can read in the paper, the results were a little concerning. First, I got many unique values for each person. Not 5,040 unique values, because some pipeline result in identical outputs. But sometimes hundreds of unique values. Some of these were admittedly very similar, but others were quite drastic. In one case, I point out that if I use some pipelines, a particular individual may be interpreted as being a linguistic innovator since she had advanced measures of some of the shifts. But in other pipelines, she would be seen as linguistically conservative since those same measures returned less-advanced measures. Taking the entire group of 53 speakers collectively, some pipelines showed that about half had shifted vowels while other pipelines showed that nearly all of them did. So, just by changing the order that I run my code can have an impact on how my dataset is interpreted!

## Takeaways

So, in addition to pointing out the problem, I do offer a potential solution by prescribing an order of operations. Here's what that is:

1. Reclassify your data into allophones

    This is important because subsequent steps really need to run by *allophone* rather than by *phoneme*. I'd even say that we should separate the data into allophones even for vowels we're not interested in (like allophones of /u/ even if we're only interested in front vowels) because it has implications for what data gets excluded which could affect the centering when you normalize.

1. Remove "bad" data

    This is when you remove outliers. Again, I argue that it should be done by allophone. I also recommend that stopwords and unstressed vowels be treated as their own separate category as well. But, I'm no married to that idea.

1. Normalize

    Regardless of what normalization procedure you use, I think it is at this point in the pipeline that the procedure should happen. The normalization should include all good data (even if it's not pertinent to the study) and not include any bad data.

1. Remove "good" but otherwise uninteresting data.

    Finally, here is where you should subset your data to focus on just the stuff you're interested. So, get rid of vowels or allophones you don't want, stopwords, unstressed vowels, etc. Importantly though, it is only at this point in the pipeline that you should toss vowel trajectories! If you never extracted them in the first place, or just ignored those columns in FAVE's output, then you're excluding trajectory data as step 1 and that has an impact on your results! 

Note that this is a different order than what I suggest in my NWAV49 talk. For now, these steps are based more on logic and theory (and I appreciate the input from Rich Ross and Thomas Kettig for helping me out with that)! I have a manuscript at the moment that dives more into these steps and provides some more quantitative justification for them. Stay tuned!

## My Soapbox

Finally, towards the end, I do get a little preachy. First, I recommend that Order of Operations be explained in detail in methods sections so that we can evaluate them better. In fact, it was only after I submitted this paper that I found [Brand, Hay, Clark, Watson, & Sóskuthy's (2021) paper](https://linkinghub.elsevier.com/retrieve/pii/S0095447021000711) that does exactly what I want to see! That's exactly the kind of transparency that we need. Of course, at this point, it's hard to interpret what effect one order has upon the data as opposed to another order, though, again, stay tuned for my manuscript which does exactly that!

I then say that quantitative linguists should be more mindful of how the data is being processed. I basically subtweet some methods that I'm not particularly fond of (Lobanov transformations, ANAE "benchmarks") without calling them out specifically. I also subtweet a reviewer who said to me once that I shouldn't use New Technique X and should instead use Old Technique Y for comparability with previous studies, even though recent research has shown that Old Technique Y is flawed and New Technique X is better. Anyway, so if those paragraphs sound cryptic, now you know some background. 

Finally, I encourage quantitatively-minded linguists to continue writing methods papers. Whether it be developing a new technique or comparing existing techniques against each other, all that stuff is good for the field.

Anyway, I took the opportunity in this non-peer-reviewed methods paper to stand on my soapbox a little bit and talk about some ov my views on the current state of quantitative linguistics. And I stand by what I wrote in the paper. 

## Conclusion

So that's it! This was a fun one to write because it literally has nothing to do with language or how language works: it's purely methods. In fact, it's not even about one specific method. It's about how we should run our R code! Very niche. But I hope it does some small part in improving quantitative linguistics papers. 

PS: In my notes I refer to "Order of Operations" as "OoO". I'm 1000% okay with having *OoO* be a new abbreviation in people's code and in methods sections now. Who wants to be the first? :)
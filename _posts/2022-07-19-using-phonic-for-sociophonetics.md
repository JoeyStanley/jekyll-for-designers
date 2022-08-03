---
layout: post
title: Using Phonic for Collecting Sociophonetic Data
date: 2022-07-19 15:19:00 -0400
tags: [How-to Guides, Research, Skills, Utah, West]
---

A few months ago, I was looking for ways to collect audio within a Qualtrics survey and I stumbled upon [this answer](https://community.qualtrics.com/XMcommunity/discussion/4517/i-wonder-how-can-i-record-interviewees-audio-response-in-qualtrics) which lead me to [Phonic.ai](https://www.phonic.ai). It looked good, so I decided to try it out for a new project. I'm happy to report that I'm overall very satisfied with how it all turned out. In this post, I'll explain how to use Phonic, tell about some of my experiences with using it, and give you some tips.

Note that this page may update if I think of more things to add. And, in an ideal world, I'd post screenshots and do more of a walkthrough for you, but I just don't quite have the time for that right now. So, I hope this text-based post provides the information you need to get started using Phonic. 

## Project Background

<span class="sidenote-left">I'm deliberately being vague here because I don't want to spoil anonymity in upcoming abstracts and manuscript.</span>
To give you a sense of what I used Phonic for, I should provide a little bit of background on my project. I wanted to collect some audio remotely from perhaps 100 people. Since this is largely exploratory data, all I had people do was read some wordlists and respond to some open-ended questions. I had 200 words that I had them read, split up into four sets of 50. I also had about 5 open-ended questions (with an additional 3 or 4 more, depending on their demographic background). 

I ran the survey for about four months, actively recruiting on Reddit and other places the whole time. In the end, I ended up with data from a whopping 324 people, which is far more than I ever expected. I'm very excited about the project! 

Some of you may know that I've been looking for a good way to collect audio remotely for a while. About five years ago I used Amazon Mechanical Turk to do so after reading about how the folks at Dartmouth used it to collect data across New England. It was---a bit of a nightmare for me. So I am very happy that Phonic is *much* more manageable. 

## What is Phonic?

According to its [homepage](https://www.phonic.ai), Phonic is a "research platform designed for voice and video." Basically, it's a survey-building site that allows you to collect audio and video data. It also has a pretty nice interface that lets you interact with your data in useful ways, which I'll describe later.

To be clear, you do have to pay to use Phonic. <span class="sidenote-left">I suppose Qualtrics is a paid software too, but I've been able to get a free account through every insitution I've been at, so it feels free to me.</span> The more you pay, the more people you can collect data from. When I started using Phonic in March 2022, they had different subscription tiers. The one that looked more appropriate to me was the academic tier, which was about $37 a month for 200 participants. That was a little more than I needed, but I definitely needed more than the 25-a-month available in the lower tier. Recently, they rolled out a different system where you buy tokens as needed. In theory, it comes out to be more economical. 

Phonic is also a survey site in and of itself. What you can integrate with Qualtrics is, as far as I can tell, only a small portion of its full capabilities. However, I am more familiar with Qualtrics and I know that things like survey flow work well in it, so to me it was worth it to use Phonic embedded within a Qualtrics survey rather than go full native Phonic. This of course adds some increased complexity when it comes to processing your data because you'll need to merge two datasets---one from Phonic and one from Qualtrics---but to me it was worth it.

## Using Phonic within Qualtrics

I won't walk you through how to integrate Phonic into Qualtrics questions because they've already got tutorials for that. I used the instructions on [this page](https://docs.phonic.ai/docs/integrate-with-qualtrics) but it looks like they've recently made a video tutorial as well. I haven't watched it, but I presume it has all the information you need. 

The gist is this. First, you need to create a skeleton survey in Phonic, with one question for each instance of audio collection you want to do in Qualtrics. I just made a really simple survey using the most basic audio collection type and put "Wordlist 1" or something simple as the question text. Each survey you make is given a unique ID, and each question within that survey is also given a unique ID. 

You then go over to your Qualtrics survey and create it like normal. Then, wherever you want to add audio collection to your survey, you open up the survey question in Qualtrics in the HTML view and add some invisible code at the bottom that links that question to a specific question using the survey ID and question ID generated in Phonic. You also need to put some HTML code in your survey header. Again, this is all explained in the documentation on Phonic's website. 

Once it's all set up, I've found that it works like a charm. When I tested it out, I noticed that the audio appeared in Phonic very soon after it was submitted in Qualtrics. You should know though that there are a few combinations of browers ond hardware that aren't going to work, which they point out [here](https://survey.phonic.ai/help). For example, it won't work on an iPhone in any browser except Safari, and it won't work directly within Facebook Messenger. 

## My one bad experience and how they fixed it

I will say that I did have some troubles at the start of my data collection. I noticed that people were completing the survey, but the audio didn't go through. I was pretty upset about it too because maybe a dozen people's recordings didn't go through at all. For another dozen or so, only some of the recordings went through. Fortunately, I wasn't paying people, so no harm done I suppose, but it sure stinks losing data.

I wrote to Phonic about the issue, showing them clear evidence that these people had indeed completed the survey but that the audio wans't going through. It took them a few weeks, but they did eventually fix the issue. They even kept me up to date on the status of their progress too. I haven't had any problems since then. So, hats off to a good customer service experience and to them fixing an issue that would have deterred me from using Phonic in the future. 

## How the Phonic interface works

Okay, so with all that in mind, lets say you've got a survey going and you're starting to collect some data. What do you get on your end? You actually get, quite a lot!

For one, it automatically transcribes your data!<span class="sidenote">I only collected spoken American English, so I have no idea how it works on other variaties or other languages.</span> Better yet, it did a pretty good job! Now, to be clear most of the folks I collected data from spoke something close to Standard American English, so take that for what it's worth. These transcriptions were *super* helpful when scanning through the data too. It's so much faster to look through a rough transcription and get the gist of what they're saying than it is to listen to the recording all the way through.

So, with these transcriptions readily available, the Phonic interface lets you quickly navigate through your data. You can look at them by-person and see all the answers each person gave. You can also look at individual questions and see all the answers to each question. 

For each response then, you get the transcription and the waveform together. When you play the audio, the word that is being said in the transcription gets highlighted, so it's easy to follow along. Furthermore, when you click on a word, it'll take you straight to that part of the audio---a very cool feature. You can also click on the waveform and it'll start playing from there. 

The Phonic interface is *very* handy. However, you lose access to all that as soon as your subscription runs out. So, take advantage of it while you can. I wish I could recreate the interface in Praat or Shiny or something, but it wouldn't quite be the same. 

## Managing downloaded data

So, interacting with your data online is nice, but as a sociophonetician, I needed to download the audio so I can do acoustic analysis on it. I also need to link it to the metadata I collect in Qualtrics. This is where you really have to flex your data management skills because it's not super straightforward to get all your data in one place. 

### Downloading audio

First off how is the audio quality? Not bad. Yes, you do get varying sound qualities because each person is recording from their own devices, and I encourage you to look at the literature on how that affects acoustic analysis. But, you do have the option to record and download in WAV format, which is handy.<span class="sidenote">Edit: Sam Hellmuth pointed out to me on Twitter that the WAV files are in fact compressed and only go up to 8000Hz! Thanks for letting me know, Sam!</span> (The other option is mp3.) I suspect though that Phonic runs it through a denoising algorithm, but it's hard to tell what Phonic does and what is the consequence of people's computer/phone microphones.

There are two ways to download audio, and I'll admit that both are not great. The first is to manually go through and download each audio file. While slow and tedious,<span class="sidenote">Keeping in mind that I had eight or so audio clips from each person, it took me about 90 seconds to download all the audio, rename them, and put them in their right folders, for one person. I had 350 people to download, so it took me several days to download it all this way.</span> it has the benefit of you knowing exactly what it is you're downloading. The part that takes the longest is renaming each file since they all get downloaded as "response.wav." I ended up writing a script in R to set up the file structure and generate a bunch of dummy files where the audio would end up. I then copied the name of the dummy file and pasted it as the name of the audio. It's a little tedious to set up, but it worked well for me. 

The second way to download audio is to do it in bulk. I did it this way at first, but after a while it stopped working. I'm not sure if it was because I was sitting on a lot of data, or because of some error on Phonic's end. Regardless, when it did work, it sent me an email with a link to a zip file. The problem is the audio files were all named with some unique ID and it was impossible to link that to the participant and question without referring to the metadata spreadsheet. So from there, I had to then set up an R script where I linked the code to the person and question and then renamed the files that way. Not a super straightforward task if you're not code-savvy. But my goodness was it much faster---when it worked.

### Downloading metadata

All the other data you collect as part of your survey can be downloaded in two spreadsheets. If you've worked with Qualtrics before, you'll be familiar with how you download the data. The spreadsheet contains survey metadata as well as responses from any of the othe questions you ask in Qualtics itself. In my case, I had a few demographic questions before the audio-recording portion started, so that information will of course be in the Qualtrics spreadsheet. 

For the Phonic data, you get similar kind of spreadsheet. Like Qualtrics, it exports the spreadsheet as one row per participant, which means it has many many columns. For each question you get the transcription (see next section) and the unique ID associated with that audio file (see previous section). You also get a bunch of sentiment analysis data. I don't use it and I'm not sure how they generate those numbers, but it's there in case you need it. 

The key piece of information you get in both spreadsheets is the column called "response_id." This is crucial because it's the only piece of information that links the Qualtrics and Phonic datasets. (So, when you go to download the Phonic data, be sure to check the box that says to include it!) Using R or whatever other software you use, you'll need that column to link the data together database-style.

### Downloading transcriptions

When I first saw that Phonic transcribes things automatically, I was very excited because it could save me a *lot* of time. Correcting rough transcriptions is usually much faster than creating fresh transcriptions from scratch. Fortunately, the transcriptions are indeed available as part of the metadata file. 

To help with correcting the transcriptions and generating good TextGrids, I wrote an R script that compiles all the transcriptions into a more streamlined spreadsheet. I have that open when I work with the files in Praat. I then just find transcription that corresponds to the audio file I'm working on, paste it into Praat, and then correct when needed and add utterance-level boundaries to break it up (which generally makes MFA work better in the next step of the pipeline). For things like wordlists (or potentially a reading passage if you use one of those), it's probably better to pull up the original script and go from there since that'll be closer to what people actually said than the automatic transcriptions. 

Unfortunately, word-level ones are not available. In theory, Phonic has that data because it has to know when to higlight each word when you're playing the audio. But I wrote to them and they said that word-level detail is unavailable. Bummer. 

## Tips

So, with all that in mind, here are a few miscellaneous tips that didn't fit anywhere else. 

**Download all your data before your plan runs out!** As soon as your subscrioption is up, you're blocked from .your survey. You won't be able to download your data or even access the handy Phonic interface. Depending on how your payment plan works, if you end up collecting more data than you have paid for, people will still be able to complete your survey and upload recordings, but you won't have access to the audio. 

I'm not sure if this is relevant anymore now that they've migrated to a token-based payment plan, but I had to write to them and **specifically request the academic tier**. I could sign up for the basic or more advanced tiers throught he website itself, but they wanted to ensure that I had a .edu email address before giving me the reduced rate for academics. Anyway, so if you run into a weird bug on the website where the academic tier is not available or whatever, write to them because that's what I had to do.

**One token is one person**. So, it doesn't matter whether your survey has one question or twenty, whether the audio files are short or long, or whether someone completes the survey or not. If they start to record something, that eats up one of your tokens. So, if you can integrate many questions into a single survey, do so because it won't cost you any more money. 

Relatedly, if you're testing your survey, each time you you through your own Qualtrics survey and record some test audio, **that also counts as one token**. I find that kind of annoying because it's really nice to be able to do some robust testing before releasing the survey into the wild. So, plan accordingly. I didn't need that many tests (maybe two or three) before being satisified with the results. 

Phonic has other features that I didn't end up using that much. But, one that I discovered later than I should have is that you can **upload any audio you want and it'll transcribe it for you**. It can even handle multiple speakers! It counts as one token, so if you're paying by the token, this might not be as good of a deal. But if you're like me and paid for a certain number of tokens per month without rollover, it could be more useful. I didn't ever use all 200 of my monthly tokens, so at the end of my billing cycle, I lost a bunch. Meanwhile, I'm sitting on hundreds of recordings from other projects that need to be transcribed. If I had seen the feature sooner, on the last day of my billing cycle I would have uploaded as many of these recordings as I could. It would have been nice to have a couple dozen of those done.

You need to be careful when editing your Qualtrics questions once you've put Phonic codes at the bottom of them. So, for example, if you find a typo and want to change the text of a question, unless you edit it in the HTML viewer, **you'll immediately lose the hidden Phonic HTML code at the bottom**. I learned the hard way that I accidentally did this when someone mentioned in one of their comments that the previous question didn't have a record button. 

Speaking of editing, **it's okay to add more questions to the Phonic survey itself**. I did this twice without any loss of data. I did this because I decided to replace one of the questions in my survey with another one partway through data collection. I just added a new question in the Phonic survey, changed the code in the Qualtrics question, and I was ready to go. Of course, what'll happen is that anyone who took the first version of the survey will have that question as blank, and anyone who takes the survey after the change will have the first question blank. But that's fine. 

Relatedly, **you can even link different Qualtrics surveys to the same Phonic survey**. I also did this because I wanted to create a pared down version of my main survey to send out to a general audience, sort of as a control group. So, I made a new Qualtrics survey, included only what I needed to, but then included the same Phonic IDs in the hidden HTML codes. The result is that I have all my data in one place on the Phonic website, regardless of what survey it came from.

## Conclusion

So, Phonic is cool and I'm a fan. Hopefully this collecting of rambling thoughts has informed you a little bit to help you decide if you want to use it or not. Please feel free to reach out to me about how I set up my survey or how I handled the data afterwards. 
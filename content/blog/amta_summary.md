---
title: 'An unhelpful summary of AMTA 2022'
date: 2022-11-06
Tags: [English]
Categories: [article]
---

Last September I attended virtually the 15th biennial conference of the Association for Machine Translation in the Americas, A.K.A. [AMTA 2022](https://amtaweb.org/amta-2022-proceedings-for-the-main-conference-and-workshops/).
There I presented an adaptation of my master's thesis in collaboration with my tutor (shameless self-promotion [here](https://aclanthology.org/2022.amta-research.13/)).
Even though at the moment I am not enrolled in any course, he helped me with several revisions and provided the budget for attendance, so I wanted to thank him before I move on.

Since the event had a somewhat comfortable schedule for someone in Western Europe who finishes work early enough, and all talks where uploaded to the even page as soon as they finished I tried to attend and write down notes for as many presentations as possible.
Lucky for you, that may be reading this anywhere from 2 months after the conference to infinity, you get to read my summary of this (likely) already outdated event! Some talks will be missing since I either did not find them interesting enough to write down anything or I forgot/skipped them. It ain't much, but it's honest work.

##### G1: Machine Translation as a Prototype for Advanced AI Deployment in Government 
_Speaker: Kathryn Baker (US Dept of Defense)_

As the title suggests, it is about taking the deployment of MT models as an example for future AI oversight in the US. It shows instances where training data may breach constitutional rights, not be usable by the government itself, traceability issues, etc.

##### UP1: PEMT human evaluation at 100x scale with risk-driven sampling
_Kirill Soloviev (ContentQuo)_

Evaluation of MT quality is usually done manually using a random sample (i.e. 10%), due to the fact that edit distance evaluation is not good and purely manual evaluation is expensive. But we can isolate the samples that we want to look up by defining a criteria that identifies the “most risky” translation post-edits by the translators. The presenter gives some ideas on suitable quality risk rules and gives an example of an overarching workflow.

##### G2: A Proposed User Study on MT-Enabled Scanning
_Marianna J Martindale (University of Maryland)*;  Marine Carpuat (University of Maryland)_

When analyzing a large volume of foreign-language text, intelligence analysts triage the overall data using MT and then pass down the specific information that they want to examine more carefully to translators. However, MT errors may make the analyst select the wrong texts, which makes everybody lose time. Which types of MT maybe better when it comes to this sort of triaging? Comprehensibility/readability vs interpretability. The presenter shows these different MT possibilities being combined in the workflow.

##### G3: You've translated it, now what?
_Michael Maxwell (ARLIS, University of Maryland)_

"Our project inferred font size from hOCR bounding boxes, and using that and other cues (e.g. the fact that titles tend to be short) determined which text constituted section titles; from this, a document outline can be created. We also experimented with algorithms for detecting bold text. Our best algorithm has a much improved recall and precision, although the exact numbers are font-dependent."

The authors use techniques such as erosion to delete all text except for the bold words in order to detect bold text in an OCR task focused on digital documents. 
To define the layout of the documents they use this tool that wasn't design by them but I found interesting, https://github.com/Layout-Parser/layout-parser. It looks nice and better suited for general-purpose text mining in a PDF than other general OCR toolkits such as tesseract, I'll give it a go someday because PDF parsing keeps being a funny unsolvable problem several notches above nuclear fusion and paper straws.

##### UP3: Post-editing of machine-translated patents: High tech with high stakes
_Aaron Hebenstreit (Self-employed)_
"In  this talk, real-world Chinese-to-English patent translations will be used in side-by-side comparisons of raw MT output and courtroom-ready products. The types of issues that can make or break a post-edited translation will be illustrated, with discussion of the principles underlying the error types. Certain nuances that challenge both humans and machines must be revealed in order to create a translation product that withstands the scrutiny of the   attorneys, scientists, and inventors who might procure it." 

In some ways automatic translation is better than HT when translating patents because of technical but frequent words in a domain that are learnt by the MT system but not known by the translators. The speaker basically went over examples of different cases where MT and HT interact. 

##### UP4: State of the Machine Translation 2022
_Konstantin Savenkov (Intento, Inc.)*; Michel Lopez (e2f)_

Quality analysis on 31 commercial MT engines.

##### R4: Prefix Embeddings for In-context Machine Translation
_Suzanna Sia (Johns Hopkins University)*; Kevin Duh (Johns Hopkins University)_

The authors use prefix embeddings and lightweight fixed-parameter method to condition a large language model. Especially good for large models in few-shot cases. 

##### G9: NVTC’s Transliteration Plug-in: What’s in a Name?
_Jen Doyon and Ekaterina Harke, National Virtual Translation Center_

Using automatic transliteration as support for MT systems for inteligence purposes. Problems and artifacts with different languages, mainly Arabic and (North) Korean, hahaha god damn really, is it because [REDACTED].  

##### UP10: A Multimodal Simultaneous Interpretation Prototype: Who Said What
Multimodal SI system that presents users “who said what”. The system takes audio-visual approaches to recognize the speaker of each sentence, and then annotates its translation with the textual tag and face icon of the speaker, so that users can quickly understand the scenario.

##### UP23: Feeding NMT a Healthy Diet – The Impact of Quality, Quantity, or the Right Type of Nutrients
_Abdallah Nasir (Tarjama); Sara Alisis (Tarjama); Ruba W Jaikat (Tarjama)*; Rebecca Jonsson (Tarjama); Sara Qardan (Tarjama); Eyas Shawahneh (Tarjama); Nour Al-Khdour (Tarjama)_

For small or mid-range NMT projects, it may be better to create a smaller, varied and good quality dataset rather than adquiring a large dataset with overall low quality. This also presents some changes to the workflow to follow, with heavy emphasis in the creation of the data.

##### R23: Measuring the Effects of Human and Machine Translation on Website Engagement
_Geza Kovacs (Lilt Inc)*; John DeNero (Lilt Inc)_

The authors answer the following 3 questions: 
- Does localizing a webpage with professional translators improve engagement, vs showing MT or English?  --> Showing a translation improves engagement notably in almost all cases. Showing a professional translation increases some specific engagement metrics such as clicking non-download links and scrolling when compared with MT.
- Should we show translations by default or English by default with a language switch option?  --> If showing English by default, only 2% of users switch to their preferred language. If showing human translation, 0.2% change to English, and the number is the same for MT. But in general, engagement metrics decrease overall if English is shown by default, even if a human translation is available upon switching.
- If we show users English, do users end up using their browser's built-in machine translation?  --> 5-10% of users shown English end up using their built-in translator. The use of these translators is higher in countries with lower English proficiency (duh).

##### UP25: Machine Translate: Open resources and community for machine translation

_Cecilia OL Yalangozian (Machine Translate)*; Vilém Zouhar (Charles University); Adam Bittlingmayer (ModelFront)_

{{<image float="left" width="14em" frame="false" src="../img/blog/amta_summary_up25.png" >}}

Presentation of a non-profit called Machine Translation, hosted at [machinetranslate.org](https://machinetranslate.org/), a platform trying to provide open resources in the field and build a community for machine translation. I remembered to write it down here because I made a screenshot of the Spiderman meme from the slides. Sounded nice enough, seems like they are focused on providing a page with commonly asked questions about machine translation. Give them a visit or something.

**_As an aside, I mistyped the name of the webpage and ended up at [machinetranslation.org](http://www.machinetranslation.org/). Looks like a page in Japanese about how people write hotel reviews. I honestly would like someone to tell me what the flying fuck is going on here, especially with that domain name. Maybe hosted by a university to use as an exercise in a machine translation course...? I'll come back with a new report if I manage to figure it out._**





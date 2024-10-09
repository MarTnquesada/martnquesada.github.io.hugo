---
title: 'Automatic rule-based text matching by approximation to a vector space'
date: 2023-12-30
Tags: [English]
Categories: [article]
draft: true
---

Using rules to systematically find the sections of a text that you are interested is something that kind of works.
From using regex to isolate webpage links in scraped text to relying on part-of-speech tagging to identify the most common verbs in Principia Discordia, it just sort of kinda works.
It does not work-work because there is always the possibility of an edge case. 
No matter how nice the framework around your patterns is. I quite like SpaCy, with which I can combine named entity recognition with POS tagging, regex and the like to make very nice, proper patterns.
But even if you can be very granular in your definitions and point at increasingly more specific things (like "A geopolitical tag followed by a date"), you are never quite sure of which edge cases you might be skipping.

This is because this whole ordeal is an NP-Hard problem as long as you care about the future. 
There is no good way to enunciate a pattern because even if you take into consideration all the text ever written for that language (sort of inconvenient if you ask me), someone might write a new sentence tomorrow and introduce a new exception to your rule.
And therefore it is not verifiable nor solvable in a given time P.

If you then say well, I don't think that anyone will write anything new (looking at [this](https://twitter.com/mattyoukhana_/status/1726017550260076889) I doubt it, but you do you) and I happen to have access to all the text, ever.
Congrats, then your problem is only NP-Complete, yay! It is now NP-Complete because, although you can verify whether your pattern works with every instance of the problem or not, you don't have any good way to congeal those instances into a nice pattern. 
There is no deterministic algorithm to do this because, what is even the problem? To model the shortest path is to build an automata that represents the language. 
So the shortest path the equivalent to hiking on a sunny day hoping to get to Alpha Centauri (we are getting there anytime now, I swear).

At this point you might be seeing where this is going, and you perhaps will say "Ahh but see, LLMs are sort of algorithms and can help with that!".
And it is true, but anything can approximate anything. We agree that, no matter how good the GPT4/Gemini/Claude-ish thing that you use is, you can never be fully sure that the pattern is infallible.
Good! Then we have accepted that this is at best an NP-Complete problem, and we are ready to employ the help of some crappy model.


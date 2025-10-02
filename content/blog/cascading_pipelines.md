---
title: 'Cascading pipelines with DSPy are kind of good'
date: 2025-10-07
Tags: [English]
Categories: [article]
draft: true
---

The global machine learning race has given us many great things: stupidly high GPU prices, a great pretext to fire workers while companies pretend that we are not in a global recession (AI is making us so productive! Also please do NOT look at my last quarter), and some truly good models. Although most likely not AGI, as seemingly every major AI company is converging on creating the best video slop website that money can buy, which is totally what would be reasonable to do when you are close to obtaining computer god, what do you mean the bubble might burst? But that is besides the point, because models have gotten truly good. At the time of writing this, the GPT-5 family, although slooooow, is fairly nice. As is Sonnet 4.5, which also boasts great quality. But it is kind of pricey, unlike DeepSeek-V3.2-Exp, which is similar to V3.1-Terminus, but much cheaper.


Across the board model quality keeps climbing _ever so slowly_, and although both the aforementioned DeepSeek family and GPT-5 seem to be getting also cheaper, they are still not exactly free. The biggest concern I would have is what happens when/if the investment money runs out and these companies are still not breaking even. I have seen [JetBrains claim that offering a pricing model with high limit rates that is on par with the current market would make them be at a loss](https://www.reddit.com/r/Jetbrains/comments/1nik5er/communication_is_hard_and_we_could_have_done/). Products like Cursor do make money, but I have concerns for their longevity when a Nvim plugin connected to Codex gets me maybe too close to what I actually use Cursor for. 


So frontier models are good, but I do not know if they are going to get significantly cheaper in the near future. In my main use case, text processing pipelines, things can get expensive quickly (and slow, still looking at you `gpt-5-mini`) as the volume increases. Recently I have been enjoying one possible solution to this: cascading pipelines, along with LLM calls that rely on DSPy (my beloved).


Cascading pipeline

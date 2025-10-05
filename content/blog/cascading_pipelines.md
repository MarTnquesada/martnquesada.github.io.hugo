---
title: 'Cascading pipelines with DSPy are kind of good'
date: 2025-10-05
Tags: [English]
Categories: [article]
draft: true
---

The global machine learning race has given us many great things: stupidly high GPU prices, a great pretext to fire workers while companies pretend that we are not in a global recession (AI is making us so productive! Also please do NOT look at my last quarter), and some truly good models. Although we are most likely not getting AGI, as seemingly every major AI company is converging on creating the best video slop website that money can buy, which is totally what would be reasonable to do when you are close to obtaining computer god, what do you mean the bubble might burst? But that is besides the point, because models have gotten truly good. At the time of writing this, the GPT-5 family, although slooooow, is fairly nice. As is Sonnet 4.5, which also boasts great quality. But it is kind of pricey, unlike DeepSeek-V3.2-Exp, which is similar to V3.1-Terminus, but much cheaper.


Across the board model quality keeps climbing _ever so slowly_, and although both the aforementioned DeepSeek family and GPT-5 seem to be getting also cheaper, they are still not exactly free. The biggest concern I would have is what happens when/if the investment money runs out and these companies are still not breaking even. I have seen [JetBrains claim that offering a pricing model with high limit rates that is on par with the current market would make them be at a loss](https://www.reddit.com/r/Jetbrains/comments/1nik5er/communication_is_hard_and_we_could_have_done/). Products like Cursor do make money, but after all it is just a text editor doing something that can be replicated by others unless they end have having exclusive access to certain models. As it is now I have concerns the longevity of their dominance when a Neovim plugin connected to Codex gets me maybe too close to what I actually use Cursor for. 


So frontier models are good, but I do not know if they are going to get significantly cheaper in the near future. In my main use case, text processing pipelines, things can get expensive quickly (and slow, still looking at you `gpt-5-mini`) as the volume increases. Recently I have been enjoying one of the many possible solutions to this: cascading pipelines, along with LLM calls that rely on DSPy (my beloved).


"Cascading pipeline" can mean a few things, but here I am referring to a multi-stage inference pipeline where earlier stages use cheaper and faster models that are optimized for recall, while later ones are costlier models that aim to detect positive examples and/or extract structured data from them. These are pipelines were some sort of classification needs to take place, but the final output of the system might include other information or a generation that partially depends on the positive samples. 


You can find this sort of cascaded approach in many, many places, just with slightly different flavours, names and Happy Meal toys. It is sometimes used in training, and they have been popular for computer vision tasks. When the pattern is used for inference, it is often times more of an engineering optimization more rather than something you would find in a paper. Since nowadays we are more prone to publish about anything, you can actually read about some implementations of this approach alongside LLMs. In (Murong Yue et al., 2024)[^fn1], the authors build a cascade of LLMs that handle user questions. The consistency of the answers given by the weaker LLM (i.e. calculated through voting) is used as an indication of the difficulty of the question being processed, and it is then used to decide whether the more expensive LLM should take care of it instead. In (LingJiao Chen et al., 2023)[^fn2], another cascade of LLMs is proposed, with DistilBERT tailored to regression as the scoring function. There is also a TikTok paper (Zixuan Wang, Jinghao Shi et al., 2025)[^fn3] that uses a cascade approach for video moderation with multimodal LLMs. The pipeline includes an initial lightweight "router" model which prioritizes recall, implemented as an embedding retrieval system that works by assessing the proximity of the samples to a bank of high-risk representative videos. The videos that are flagged as potentially high-risk go onto the second, more expensive step, which is an MLLM-based ranker that uses LLaVA as its architecture, Mistral 7B as LLM and ViT-Large as vision encoder.  


{{<image float="center" width="32em" frame="false" src="../img/blog/bee.jpg" caption="Please enjoy a bee that I encountered last June. Maybe take a rest of reading and have a biscuit.">}}


Whether it is mostly as a filtering step or to answer queries with cheaper models whenever possible, cascading is nice for pipelines that ingest data asynchronously and are concerned mostly about classification or generation tasks that need to take into consideration every single datapoint. Use cases that require immediate user interaction and do not care as much about all datapoints are often better served by some amalgamation of semantic similarity search plus generation (I am scared of saying RAG outloud). But even in the latter case, some cascading tricks are often applicable.


Let's build an example of a cascading pipeline so that we can talk about specifics, the cool stuff. Let's say that 


{{<image float="right" width="26em" frame="false" src="../img/blog/tree.jpg" caption="Have another break, this time looking at this pine tree. Or perhaps closing your eyes for a little bit. So many paragraphs...">}}




[^fn1]: Large Language Model Cascades with Mixture of Thought Representations for Cost-Efficient Reasoning - Murong Yue, Jie Zhao, Min Zhang, Liang Du, Ziyu Yao (2024)
[^fn2]: FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance - Lingjiao Chen, Matei Zaharia, James Zou (2023)
[^fn3]: Filter-And-Refine: A MLLM Based Cascade System for Industrial-Scale Video Content Moderation - Zixuan Wang, Jinghao Shi, Hanzhong Liang, Xiang Shen, Vera Wen, Zhiqian Chen, Yifan Wu, Zhixin Zhang, Hongyu Xiong (2025)

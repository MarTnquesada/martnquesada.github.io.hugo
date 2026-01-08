---
title: 'Cascading pipelines with DSPy are kind of good'
date: 2026-01-08
Tags: [English]
Categories: [article]
draft: false
---

The global machine learning race has given us many great things: stupidly high GPU and RAM prices, a great pretext to fire workers while companies pretend that we are not in a global recession (AI is making us so productive! Also please do NOT look at my last quarter), and some truly good models. Although we are most likely not getting AGI, as seemingly every major AI company is converging on creating the best video slop website that money can buy, which is totally what would be reasonable to do when you are close to obtaining computer god. But that is besides the point, because models have gotten truly good. At the time of writing this, the GPT-5 family, although slooooow, is fairly nice. As is Sonnet 4.5, which also boasts great quality. But it is kind of pricey, unlike DeepSeek-V3.2-Exp, which is similar to V3.1-Terminus, but much cheaper.
_EDIT: In the time between writing this paragraph and finishing the post, new Gemini, DeepSeek, and Claude models have come out. So. Yeah._


Across the board model quality keeps climbing _ever so slowly_, and although both the aforementioned DeepSeek family and GPT-5 seem to be getting also cheaper, they are still not exactly free. The biggest concern I would have is what happens when/if the investment money runs out and these companies are still not breaking even. I have seen [JetBrains claim that offering a pricing model with high limit rates that is on par with the current market would make them be at a loss](https://www.reddit.com/r/Jetbrains/comments/1nik5er/communication_is_hard_and_we_could_have_done/). Products like Cursor do make money, but after all it is just a text editor doing something that can be replicated by others (provided that they are able to create a decent editor and fine-tune stuff) unless they end have having exclusive access to certain models.  


So frontier models are good, but I do not know if they are going to get significantly cheaper in the near future. In my main use case, text processing pipelines, things can get expensive quickly (and slow, still looking at you `gpt-5-mini`) as the volume increases. So today I want to write about one of the many possible solutions to this: cascading pipelines, along with LLM calls that rely on DSPy (my beloved).


"Cascading pipeline" can mean a few things, but here I am referring to a multi-stage inference pipeline where earlier stages use cheaper and faster models that are optimized for recall, while later ones are costlier models that aim to detect positive examples and/or extract structured data from them. These are pipelines were some sort of classification needs to take place, but the final output of the system might include other information or a generation that partially depends on the positive samples. 


You can find this sort of cascaded approach in many, many places, just with slightly different flavours, names and Happy Meal toys. It is sometimes used in training, and they have been popular for computer vision tasks. When the pattern is used for inference, it is often times more of an engineering optimization more rather than something you would find in a paper. But since nowadays we are more prone to publish about anything, you can actually read about some implementations of this approach alongside LLMs. In (Murong Yue et al., 2024)[^fn1], the authors build a cascade of LLMs that handle user questions. The consistency of the answers given by the weaker LLM (i.e. calculated through voting) is used as an indication of the difficulty of the question being processed, and it is then used to decide whether the more expensive LLM should take care of it instead. In (LingJiao Chen et al., 2023)[^fn2], another cascade of LLMs is proposed, with DistilBERT tailored to regression as the scoring function. There is also a TikTok paper (Zixuan Wang, Jinghao Shi et al., 2025)[^fn3] that uses a cascade approach for video moderation with multimodal LLMs. The pipeline includes an initial lightweight "router" model which prioritizes recall, implemented as an embedding retrieval system that works by assessing the proximity of the samples to a bank of high-risk representative videos. The videos that are flagged as potentially high-risk go onto the second, more expensive step, which is an MLLM-based ranker that uses LLaVA as its architecture, Mistral 7B as LLM and ViT-Large as vision encoder.  

{{<image float="center" width="32em" frame="false" src="../img/blog/bee.jpg" caption="Please enjoy a bee that I encountered last June. Maybe take a break from reading and have a biscuit.">}}

Whether it is mostly as a filtering step or to answer queries with cheaper models whenever possible, cascading is nice for batch processing tasks that are concerned mostly about classification, or generation tasks that need to take into consideration every single datapoint. Use cases that require immediate user interaction and do not care as much about all datapoints are often better served by some amalgamation of semantic similarity search plus generation (I am scared of saying RAG outloud). But even in the latter case, some cascading tricks are often applicable.


Let's build an example of a cascading pipeline so that we can talk about specifics, the cool stuff. Say that we are a telecomunications company and we want to build a system that aims to find and catalog client complaints about issues specific to their equipment, i.e. their router or optic fiber terminal box. This means that the input data to our pipeline will consist of online chats that the clients had with our support agents. This is a somewhat decent example because it is a nuanced classification task where inter-annotator agreement might be lower than expected. As a result, we want to rely at least partially on top-of-the-line LLM. But because we also want to not run out of money, we go the route of a cascading pipeline.


The first step that I have found consistently useful is to find any clear-cut cases of negative samples and remove them by means of ad-hoc rules. For instance, in our particular example it is reasonable to expect that any calls with complaints about their cell phone signal can be discarded. This is especially important for cases where there is a group of samples particularly prone to false positives, like the clients and the assistants always mentioning the router in conversations where the client is terminating their subscription, since they are required to send it back through mail. Of course, this is only possible if such samples have some sort of clearly-defined feature that allows us to isolate them with a few ad-hoc rules. Let's say that we have a list of expressions that appear in the vast majority of calls where the client is returning their router. We can use something like [SpaCy](https://spacy.io/) to generate both simple and relatively complex patterns. 

Although I have frequently found this initial step fairly impactful, it is very easy to overdo it and fall into a slippery slope of continuosly adding new rules as you see instances of misclassified samples during development. The number of rules should be kept to a minimum and be only comprised of extremely high-confidence cases (i.e. in a random subset of 2000 samples, 99.1% matches for this pattern belonged to a conversation about the client finishing their subscription). Alternatively, it could also be implemented as a nearest class mean / prototype-based classifier, which is also a-okay.


After adding any number of heuristics, the next stages of the cascading pipeline can begin to incorporate filters that, while maximizing recall, aim to aggressively reduce the total volume of data, at the cost of being less precise. This can include any probabilistic model that is reasonably fast and cheap for your particular use case:
- A support vector classifier (SVC) with parameters tuned to maximize recall (or a recall-adjacent metric like F2) in cross-validation. Please do not give into the temptation of getting some prediction probabilities (so that you can add an additional threshold) through Platt scaling, pairwise coupling for multiple classes or by ingesting a large amount of unidentified mushrooms and visualizing the probabilities by exiting your body and looking at the back of your head. They are all equally questionable methods.
- An [XGBoost](https://github.com/dmlc/xgboost) or [CatBoost](https://github.com/catboost/catboost) classifier with hyperparameters that maximize recall and/on a hand-picked probability threshold under 0.5. Honestly, I have never gotten boosted trees to play nice with pure text embeddings, so I would recommend this above the other methods listed here only if you have some categorial data alongside your text. In which case CatBoost will probably be pretty good. Otherwise the whole thing will be about as good as an SVC.
- A Transformer classifier fine-tuned on your task ([ModernBERT](https://huggingface.co/answerdotai/ModernBERT-base) would be my go-to right now). This is heavier and demands a larger training set, but does the best with text-only stuff and you can still do a few things to maximize recall: early stopping based on recall/F2, sweeping the decision threshold on validation and then choosing the checkpoint and threshold that achieves the best recall/F2 on said validation set, oversampling rare positive samples, etc.

{{<image float="right" width="26em" frame="false" src="../img/blog/tree.jpg" caption="Have another break, this time looking at this pine tree. Or perhaps closing your eyes for a little bit. So many paragraphs...">}}

These are all cool options that will be sure to get the crowd excited 🏟. And you might think, this is when [DSPy](https://dspy.ai/) comes in. We can pipe our nice, filtered stream of samples into a more expensive stage stage where an LLM takes care of the final decision + perhaps some information extraction. And you would be correct, I quite like this pattern.

__But__


DSPy shines brightest at something different. It is still great at optimizing instructions for a modest Haiku-3 or more capable GPT-5, as well as at abstracting your task into pure code and some training data. However, instruction optimizers like GEPA give you the juiciest improvements when used alongside smaller models. DSPy's cookbook generally assumes that you will rely on a "dumber" student model for the actual task, and a better model as a teacher that will mutate the prompts. But even within simpler models, GEPA seems to really blur the differences between Qwen3-8B (Lakshya A Agrawal, Shangyin Tan et al., 2025)[^fn4] or Mistral small 3.1 and GPT-5 Nano (Lingbo Li, Anuradha Mathrani and Teo Susnjak, 2025)[^fn5]. So we can go really light on the model (and smaller models are getting better at a better pace than frontier ones) while still getting fairly good results. So _let's use that very-small LLM prompted with DSPy as a filtering step_.


There are many good quantized models, I personally really like [Kimi-Linear-48B-A3B-Instruct](https://huggingface.co/models?other=base_model:quantized:moonshotai/Kimi-Linear-48B-A3B-Instruct) for something like this. But I am currently on holidays and the only GPU in my house is attached to a computer from 2015. So how about we instead use a quantized [Qwen3-8B](https://huggingface.co/models?other=base_model:quantized:Qwen/Qwen3-8B).  


To serve this in an actual production system I would go for an instance running Ray Serve with vLLM backend, or a Kubernetes cluster with vLLM pods, or Kubernetes + Ray with vLLM backend (using KubeRay), which I find very easy to use if you already know Ray. In local, if you have anything that will run cuDNN you are probably fine just by spinning up a vLLM service that runs quantization ([this naive one for example](https://huggingface.co/Qwen/Qwen3-8B-FP8) with:

```bash
vllm serve Qwen/Qwen3-8B-FP8 --port 8080
```

But I have a magnificent Apple Silicon machine and I need to rely on Metal, so I can't do that. Instead, I will use llama.cpp with a GGUF-format quantized model:  

```bash
# Download your GGUF quantization of choice
uvx hf download \ 
    Qwen/Qwen3-8B-GGUF Qwen3-8B-Q4_K_M.gguf \
    --local-dir ./models

llama-server \
    -m ./models/qwen3-8b-q4_k_m.gguf \
    --port 8080 \
    --host 0.0.0.0 \

```

Both of the previous options will serve the model using the OpenAI inference standard (exposing `http://localhost:8000/v1/chat/completions` and all of that), so you can directly wire them to DSPy using the `/openai` prefix:


```python
import dspy


lm = dspy.LM(
    # The model name does not matter unless you are using vLLM 
    # with multiple models or use --served-model-name 
    "openai/Qwen/Qwen3-8B-FP8",     
    api_base="http://localhost:8080/v1",
    api_key="whatever-this-is-not-needed-in-local"
)
dspy.configure(lm=lm)

```

Good, now I can assume that we all have the task model running __somewhere__. We can design a basic training in DSPy using the GEPA optimizer. For that, we need an additional reflection model that takes in feedback and writes new prompts, for which I'd recommend using a more capable LLM served by any of the usual suspects. 

For the GEPA metric itself, we should keep in mind the primary function of this step: maximizing recall while still being able to discard a significant % of the incoming samples. While this cannot be done in a formal manner as with the probability threshold over XGBoost or early stopping based on F2 on a Transformer model, we can still coerce DSPy into generating a prompt that favours high recall by adjusting the rewards and penalties that determine the score in each step of our metric. I first stumbled across this approach in this [snippet from John Damask](https://gist.github.com/jbdamask/26967196122505fa9f42eeba40b1dfd4), and it can be done in a few different ways, but I still find his approach elegant enough. Just set the true positive, true negative, false positive and false negative scores in such a way that they tend to maximize recall, like so:

```python
def my_metric:
    tp_reward=1.0, fn_penalty=1.0, fp_penalty=0.25, tn_reward=0.10
):
    ...
    if true_positive:
            score = tp_reward
            feedback = (
                f"✅ Correct: '{sample_raw}'. "
                "Keep in mind the following cues" 
                "for positive samples: ..."            )
    elif true_negative: 
        ...
    ...
    score = float(max(0.0, min(1.0, score)))
    return dspy.Prediction(score=score, feedback=feedback)

```

This can be adapted to most tasks, with the feedback part of the metric being the most domain-dependent thing to adjust.

There are a couple more things that you can also do to maximize recall during the GEPA training:
- Oversample for the positive class in the training set. 
- Add an additional field in your positive samples that provides an explanation for the label. Even better if the field is also present for negative samples, but having it for the recall-optimized class is often enough.
- Adjust your penalties and rewards (i.e. increase the false positive penalty).

And there you have it, a DSPy-powered filtering step for your cascading pipeline. This strategy is still likely not something that you want to use for a process that goes through > 1M samples an hour, but it can be combined with other filters (that, you know, cascade) like the ones mentioned earlier. And you can get fairly cute optimizing your tiny LLM inference by testing ever-smaller models, reducing context size, increasing batch size if you have memory to spare (although this impacts mostly prompt prefill and thus scales with your number of calls, not tokens), etc. 

_The end_



[^fn1]: Large Language Model Cascades with Mixture of Thought Representations for Cost-Efficient Reasoning - Murong Yue, Jie Zhao, Min Zhang, Liang Du, Ziyu Yao (2024)
[^fn2]: FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance - Lingjiao Chen, Matei Zaharia, James Zou (2023)
[^fn3]: Filter-And-Refine: A MLLM Based Cascade System for Industrial-Scale Video Content Moderation - Zixuan Wang, Jinghao Shi, Hanzhong Liang, Xiang Shen, Vera Wen, Zhiqian Chen, Yifan Wu, Zhixin Zhang, Hongyu Xiong (2025)
[^fn4]: GEPA: Reflective Prompt Evolution Can outperform Reinforcement Learning - Lakshya A Agrawal, Shangyin Tan, Dilara Soylu, Noah Ziems, Rishi Khare, Krista Opsahl-Ong, Arnav Singhvi, Herumb Shandilya, Michael J Ryan, Meng Jiang, Christopher Potts, Koushik Sen, Alexandros G. Dimakis, Ion Stoica, Dan Klein, Matei Zaharia, Omar Khattab (2025)
[^fn5]: Automated Risk-of-Bias Assessment of Randomized Controlled Trials: A First Look at a GEPA-trained Programmatic Prompting Framework - Lingbo Li, Anuradha Mathrani, Teo Susnjak (2025)

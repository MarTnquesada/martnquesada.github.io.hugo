---
title: 'An unhelpful summary of PyConES 2024'
date: 2024-10-09
Tags: [English]
Categories: [article]
---

{{<image float="right" width="14em" frame="false" src="../img/blog/pycon_credentials.jpg" >}}

Last week I went to Vigo to attend the Spanish PyCon, PyConES. I traveled there fully sponsored by Datamaran, the company that I work for.
I attended as a speaker, since a colleague and I prepared a talk proposal that got accepted (something something ESG with clustering and LLM reformulation).

### Day 1 🌦

The first day had only two turns for workshop tracks. I attended:

##### Overcoming the One Billion Row Challenge with Python
_Jordi Contestí, Kiko Correoso, Ernesto Coloma Rotger_

This workshop showed different tools and approaches that can be used in Python to overcome the somewhat known [One Billion Rows Challenge](https://1brc.dev/).
The data files themselves had about 50M lines, to avoid laptop oopsies, but the emphasis was on discussing and trying out different file formats and data manipulation frameworks in the attendants' hardware.

They played with regular CSV, Apache Parquet and Feather. 
From the data manipulation side they show examples with `numpy`, `pandas`, `polars`, `pyarrow`, `duckdb`, `dask` and `modin`+`dask`.

Kind of interesting in the sense that I did not know about some of the formats/libraries, although it seemed hard to come out with any definitive thought about which combination of approaches is the most performant, since most of them are hardware-dependent.
Polars and pyarrow were generally pretty good over most file formats. Parquet had the potential to be also very fast in certain cases.
Kiko mentioned so many times that performance "depends" on data and hardware that the whole room started answering each other's questions with "depends".

They shared all the workshop materials in this [repo](https://github.com/PyDataMallorca/PyConES2024_Superando_el_1brc_con_Python). Beware: explanations in Spanish, variables in English, functions in Spanish. But like. You will probably understand it.


##### GenAI❤️f-string. Developing with Generative AI without Black Boxes
_Alejandro Vidal_

This workshop was about black boxes in _libraries around LLM inference, not the LLM architecture itself_, using langchain as the main example.
The initial case was showing how hard it currently is to have visibility on what is the actual HTTP call being sent to the LLM API.
To the point where the "fastest" way to spoof the HTTP call is to set up a man-in-the-middle proxy (as featured in [this article](https://hamel.dev/blog/posts/prompt/)).

In order to have code that actually lets you see your final prompt and HTTP call without much fuss, Alejandro built a minimal RAG using calls to an LLM API and basic Python functions, avoiding creating any classes.
Throughout coding, he showed examples where langchain unnecessarily obscures certain steps (how tutorials point developers into using prompts from the langchain hub which are versioned independently of the code, having SmartLLMChain as a class for a self-critiquing LLM, ...).

Overall it was nice to listen to a measured critique, with specific examples and advocating for simpler code.


### Day 2 ⛈️

The second day had a bunch of talks, but in my great wisdom I went to these:

##### Building Resilient and Scalable Software with MLOps
_Daniel Pérez Cabo (Alice Biometrics)_

Daniel works on Alice, an authentication service. He gave an overview of their ML pipelines, how they have evolved as of recent and common issues.
Some key takeaways were (that is, whatever I remember, because I did not write down notes for this talk):
- They were using a regular cloud server to serve models, but they migrated to using an inference server (on the cloud too), specifically NVIDIA Triton. When asked about the cost increase, he stated that at least in their case it was not significant, given that they were serving several different models from the server.
- He put emphasis on versioning models along with their associated preprocessing and postprocessing steps.
- It was also mentioned that currently they re-deploy models transparently without changing anything in the app itself, only the model, so no app restart is needed.
- He mentioned that they used a Python package called `petisco` (https://github.com/alice-biometrics/petisco) to manage the lifecycle of applications using hexagonal architecture, as well as abstracting away certain aspects of the overall architecture. It seems like it is an open-source library that they built.

##### The Day that I Started to Develop All of My Webs in Python
_Brais Moure (MoureDev)_

I did not really know Brais, but it seems like he is somewhat of an influencer, or at least he has a decent following on social media.
The talk was about `reflex` (https://reflex.dev/), which is a Python framework to create full webapps that wraps around FastAPI + React + Vue.
He basically wanted to build webs using only Python coming from being a mobile-only developer, since Reflex declares components in Python to build the front of the webpage.


##### Who Needs Data Having Distilabel?
_Gabriel Martín Blázquez (HuggingFace)_

Disclaimer: I am very biased because I have collaborated with the Argilla team before, including Gabriel, and they are all very nice 🤗.

Gabriel presented `distilabel` (https://github.com/argilla-io/distilabel), a framework for synthetic data. 
Although not actually paired with the dataset annotation tool `argilla` (https://github.com/argilla-io/argilla), it was built by part of the original team (now operating under HuggingFace, who bought them).
The talked about the initial 0.x distilabel release, which got a lot of traction and was mentioned in quite a few publications. 
As a result, its development was accelerated, and they fell into what he called "hype-driven development". Features were added one after another without too much consideration about the maintainability and general design of the pipelines.
With the new 1.x releases they have done away with threading, and instead allocate independent processes per each step of the pipeline, making them a lot more scalable.
They have also made other nicer changes, but I don't remember them. I do recall thinking that HDD is not really that bad as long as you follow it up with a nice 1.0 release, since it got Distilabel off the ground.

If Distilabel (or Argilla for that matter) sound interesting, I recommend giving them a look, and the team is very active in the HuggingFace discord.

##### Sweet Introduction to Ruff
_Elena M. Codonyer, Ángel Collado Aspas (Datamaran)_

Other two talented and good-looking colleagues of mine gave a talk about Ruff, a Python linter and formatter that we are now using across the company, and how it was introduced step-by-step to all teams.


##### Less Hype and More Responsibility? Who Decides What in the Use of Data and AI
_Anna Colom (The Data Tank)_

The closing talk of the day (and the only one for that time slot) about present and future impacts of AI for the environment, society and politics. 
Anna was nice, but I feel like you need someone who both has technical machine learning knowledge and is well-read in order to give this kind of talk, not the latter (like in this case).

### Day 3 🌧

The last day, with fewer people, more rain, more food:

##### How Covid is Messing with Time Series Prediction and What to Do to Avoid It
_Mireya, Jorge Raúl Gómez Sánchez (decide)_

This was a talk that I was originally not going to attend (instead I was going to go to another one about async in Python).
But during the speakers dinner I talked with some people and the topic of Covid interacting with data came out as something that is still actually messing with not only time series prediction, but also training data from those years.
So I thought I would check it out.

Most of the examples in the talk orbited around Prophet prediction models, and how to alter the data from time series in order to mitigate the effects of anomalous datapoints from Covid.
They broke down the issue into a couple of cases, based on whether the time series was seasonal, whether the Covid period corresponded with a full seasonal cycle, and whether the tendency of the series went back to "normal" after Covid.
Depending on the case they use different components of the time series to "correct" the covid period. If I had the slides I'd share them, because it was interesting, but I mostly forgot the specifics.

Overall I found it genuinely helpful to deal with Covid data, even if this was pretty much exclusively focused on time series.
Also, discussing actual statistics was a nice break from the ML guessing game.

##### Navigating the ESG landscape with LLMs
_Vincent Rizzo, Martín Quesada Zaragoza (Datamaran)_

A very handsome Frenchman and an equally good-looking Spaniard gave out this insightful talk on doing funny clustering and LLM stuff with ESG disclosure for CSRD.

##### Ensuring Data Quality with Databricks
_Antonio, José Manuel García Giménez_

I honestly have not touched Spark in many years and as a result I don't know much about Databricks, 
so I am not sure of why I went to this one. I recall them talking about the three layers of data that they were working with:
- **Bronze:** All the incoming data.
- **Silver:** Same as Bronze, but without intrinsically bad data (that is, data that can be filtered out independently of business rules, such as datapoints with null fields).
- **Gold:** Only the data to be actually used in the final application, which adds filters based on business rules to the data in the Silver layer.

I do not remember much else, other than some questions about scheduled vs always-on streaming to reduce costs (that is, turning on the layers every once in a while since the streaming process continues from the last checkpoint).

##### The Great Misinformation Tsunami: How Generative AI Can Help Us Win the Battle Against Fake News
_Rubén Míguez, Agustín Cañas (Newtral)_

This one was full of people, so much so that I had to stand next to the entrance door the whole time.
The popularity of the talk was probably due to both the theming and the company behind it, Newtral.
I was unaware of them, but they are apparently both a media production company which does TV programs (mostly journalism/political) and documentaries, as well as a tech company doing fact-checking, sometimes for the aforementioned programs.

The talk itself had a few things that I found interesting about the actual structure of the fact-checking pipeline, including a few snapshots about how it has evolved throughout the years.
Most of the effort seemed to be around reducing the number of texts that humans actually have to fact-check, as well as storing previous fact-checks to match new incoming data to them if possible.
They were also working on a chatbot that could be publicly available, 

Other than that the presentation had a lot of statistics about the relative volume of fake news on social media, some stories, and high-level views of their pipelines.
They also showed a slide of their tech stack that left me confused because it had a hundred things on it, many of which one would think that overlap with each other. With MongoDB-Postgres-MariaDB-RedShift-ElasticSearch-MySQL they sure do have some databases. 

##### Workflows with AI agents and Python
_Manuel Díaz, Borja Esteve Molner, Rafael Mena-Yedra (decide)_

They introduced 3 techniques from 3 different papers that are relevant to the presentation:
- **Chain-of-thought prompting** (Chain of Thought Prompting Elicits Reasoning in Large Language Models, Wei J.,et al. 2022).
- **ReAct** (ReAct: Synergizing Reasoning and Acting in Language Models, Yao S., et al. 2022), which establishes the following LLM agent cycle: act, observe and reason.
- **Toolformer** (Toolformer: Language Models Can Teach Themselves to Use Tools, Schick T., et al. 2023), which introduces the concept of LLMs that decide on their own when and how to use external tools.

The gist of it is that you have a graph containing "LLM agents", which are really nodes containing functions that do specific LLM inference calls.
Within this graph you define the flow of states and the characteristics of the LLM roles (i.e. one of them is a functional analyst so its prompt is funny like that), along with other nodes like those requiring human feedback.
And once the process gets started, it kind of takes care of itself autonomously. They use checkpoints that store the state of the whole graph, so that it can be paused, for instance when waiting for human input.
The graph itself ends up being composed of:
- States: State of shared data, I.e. a db
- Nodes: Functions, usually either LLM calls or human feedback loops.
- Links: Decide which node executes what.

For the presentation they use Langgraph + LangChain, but they mentioned other LLM agent frameworks like LlamaIndex Workflows and CrewAi.

They shared a repo with the examples and the slides [here](https://github.com/rael-my/pycon24_agents).

Nice presentation overall, although just like with most things around LLMs that are not the actual model architectures, LLM agents are a simpler concept than I originally expected.


##### The power of observability in machine learning 
_Sara, Christian Carballo Lozano_

Sara and Christian talked about some issues with model management, namely generalization issues and model drifting.

For the first point, they went over model explainability, putting as an example an image detection model using only the background of images to distinguish huskies from wolves, since wolf photos were always in the snow.
They mentioned libraries like Aix360, Interpretml, Sham and Lime, and then showcased Lime in particular.

Regarding model drifting, they touched on observability and broke it down into:
- **Logging**, for which they use logd (https://github.com/hiidef/logd).
- **Metrics**
- **Tracing**, which is having access to how an inference was created from the first input/data ingestion

Data drift was also considered, for which they currently use the Python library `evidently` (https://github.com/evidentlyai/evidently).

This presentation was nice too, but I never end up seeing explainability being automated so that it goes beyond a sample-by-sample study.
When working with text models, it is very rare that you actually spot the issue in a couple of datapoints, I would rather aggregate the focal tokens and get a sense of how it looks for a large subset of samples.

### Closing words

{{<image float="right" width="14em" frame="false" src="../img/blog/pycon_vigo.jpg" >}}

Overall, it was nice. My last academic years were during Covid so all the conferences that I had attended were online, a far cry from eating octopus while dissing langchain with new people.
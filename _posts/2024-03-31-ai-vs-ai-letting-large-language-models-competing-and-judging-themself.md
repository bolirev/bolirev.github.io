---
title: "AI vs AI: Letting Large Language Models competing and judging themself"
categories:
  - Blog
tags:
  - AI
  - LLM
---

OpenAI has been leading the way in AI development, expanding the horizons of machine capabilities. Their work on Large Language Models (LLMs) has been groundbreaking, allowing machines to produce human-like text, answer queries, translate languages, and even write code. With the advent of the short revolution, new LLMs have emerged, such as Mistral AI, Gemini, and Claude-3, often surpassing their predecessors in various fields. Their performance is evaluated through benchmarks like the Massive Multitasks Language Understanding (MMLU) designed to measure the knowledge and problem-solving abilities of language models across a wide range of subjects, from STEM fields to the humanities and social sciences; TriviaQA dataset consisting of a comprehensive reading comprehension dataset that contains over 650,000 question-answer-evidence triples; and many others. However, these tests may not fully capture an AI’s capabilities, as real-world scenarios are often more complex and interactive than benchmark tests.

Unlike traditional metrics and tests used to evaluate LLMs, I was wondering whether we could assess AI by using AI: allowing AI models to interact directly with each other and evaluate their own responses and those of their peers. Inspired by debates used as a training framework to shape and challenge the minds of future leaders, lawyers, or judges (see e.g. [The importance of debate](https://oxfordsummercourses.com/articles/the-importance-of-debate/) or the [Debating society germany](https://www.schoolsdebate.de/index.php/about-us/our-competitions)), I designed a simple framework for LLMs to challenge each other. This framework consists of four parts: topic generation, debating, judging, and voting. In each part, roles will be assigned to LLMs by using prompt inspired by COSTAR — Context Objective, Style, Tone, Audience, and Response — prompt engineering. The results format will be assessed, and if it didn’t fit the task, it was rescheduled.

The questions I aimed to explore were:

- Do LLMs perform better in debates when they choose their own topics
- Are LLMs more effective in the proposing or opposing team?
- Do they judge a debate impartially; i.e., do they not favor themselves?

#### **Article goal:**

In this article we will see how a debate between AI models can be created The evaluation of debates can be found [here](/blog/debates-as-seen-through-the-prompts-of-llms/).

(Code can be found here: <https://github.com/bolirev/ai-debate>)

#### **Short summary of LLM used**

**OpenAI — ChatGPT-4:** GPT-4, the latest model from OpenAI, showcases several significant improvements over its predecessor, GPT-3.5. Its key strengths include enhanced language understanding and generation, enabling it to comprehend and produce various dialects and emotional nuances, as well as create more coherent and creative content. GPT-4 also demonstrates superior reasoning and problem-solving capabilities, tackling complex mathematical and scientific problems with ease. Its multimodal capabilities set it apart, as it can analyze and comment on images and graphics. With an increased scale and capacity, GPT-4 caters to long-form content creation, extended conversations, and document analysis. Moreover, its safety and alignment have been improved, making it more reliable in providing factual responses and refusing disallowed content. Lastly, GPT-4’s advanced programming abilities make it a valuable resource for software developers.

Benchmarks: <https://openai.com/research/gpt-4>

**MistralAI:** This model showcases exceptional capabilities, such as efficient processing through a sparse mixture of experts, dynamic expert utilization for nuanced responses, and the ability to handle a context of 32k tokens. MistralAI’s emphasis on open technology leadership, strong financial backing, and commitment to efficiency in AI solutions further solidify its position in the AI market. Additionally, MistralAI offers a range of products tailored to different needs, from cost-effective endpoints like Mistral-tiny to more robust offerings like Mistral-medium.

Benchmarks: <https://docs.mistral.ai/platform/endpoints/>

**Gemini-Pro:** Gemini-Pro showcases several notable improvements and features compared to its predecessors. Its key strengths include enhanced performance and efficiency, achieving comparable quality to larger models while using fewer computational resources. The model introduces a breakthrough in long-context understanding, processing up to 1 million tokens, which is the longest among any large-scale foundation models. Gemini-Pro also boasts multimodal capabilities, supporting both text and image inputs and comprehending 38 languages. With a focus on safety and alignment, Google has conducted extensive ethics and safety testing for responsible deployment. Additionally, Gemini-Pro offers faster inference speed, potentially leading to real-time latency gains, and demonstrates strong abilities in following simple instructions.

Benchmarks: <https://blog.google/technology/ai/google-gemini-ai/#performance>

**Claude-3:** Claude-3, the AI model from Anthropic, demonstrates more natural, human-like language abilities, engaging in coherent, creative, and nuanced conversations. The model outperforms competitors like GPT-4 in IQ tests and excels in mathematics, information retrieval, and other benchmarks. Claude-3 also features multimodal capabilities, enabling it to analyze and comment on images and graphics. nthropic has prioritized safety and alignment, making Claude-3 more reliable and less prone to harmful outputs. Lastly, the model’s versatility allows it to tackle a wide range of tasks, from creative writing to academic-style analysis.

Benchmarks: <https://www.anthropic.com/news/claude-3-family>

### **An Ai-debate**

#### The steps:

1.  Generate topics: LLMs will be asked to generate 10 topics
2.  Debate: two LLMs will be debating
3.  Judgement: Each debating team will be judge for each discourse with one grade per category
4.  Public-voting: based on the discourse and the judgements, LLMs will be asked to choose the best judgement.

**Pre-requisit**

I first created an API key for each of the platform (except for Gemini which is using gcloud authentification for the Vertex AI api). I placed my keys in a local environment file, saved as:

    OPENAI_API_KEY = '1234'
    MISTRAL_API_KEY = '5678'
    ANTHROPIC_API_KEY = '9101112'

Since I needed to get different AIs, I wrote an abstract AI Chatter defining the core functionalities of the LLMs. This base class is inherited from the APIs specific implementation for OpenAI, MistralAI, Gemini and Claude-3 respectively.

    from ai_debater.models.openai_chatter import OpenAIChatter
    from ai_debater.models.gemini_chatter import GeminiChatter
    from ai_debater.models.mistralai_chatter import MistralAIChatter
    from ai_debater.models.abstractai_chatter import BaseAiChatter
    from ai_debater.models.claude3ai_chatter import Claude3AiChatter

    def generate_model(models2generate: List[str]) -> Tuple[List[BaseAiChatter], pd.DataFrame]:
        models = list()
        if 'GeminiChatter' in models2generate:
            models.append(GeminiChatter())
        if 'MistralAIChatter' in models2generate:
            models.append(MistralAIChatter(api_key = os.environ.get('MISTRAL_API_KEY')))
        if 'OpenAIChatter' in models2generate:
            models.append(OpenAIChatter(api_key=os.environ.get('OPENAI_API_KEY')))
        if 'Claude3AiChatter' in models2generate:
            models.append(Claude3AiChatter(api_key=os.environ.get('ANTHROPIC_API_KEY')))
        if not models:
            raise NameError("No model has been created")
        model_infos = pd.DataFrame({m.model_id():m.metainfo for m in models}).transpose()
        model_infos.index.name = 'model_id'
        model_infos.columns.name = 'property'
        model_infos = model_infos.reset_index()
        return models, model_infosp

Since I not only wanted to play with the interface, but later want to analyse the results, I opted for storing the results in a sqlite file.

    # Create your connection.
    from ai_debater.io_database import IODataBase
    result_manager = IODataBase('results/dataset_v2.db')

#### **Step 1: Topics generation**

The source of a debate is a controversial topics; indeed when everyone agrees nothing needs to be discussed nor debated. I could have come up with different topics by myself, or used popular topics used in debates. However as I wanted to evaluate the performance of AI on their own generated view of potential challenges, I first let each AI generate 10 topics.

A topic may be summaries in one sentence. When designing the prompt I quickly realised that the topic may not have a given side or may be ambiguous. I therefore decided to not only let the LLM generate the topic but as well a rational of this topic. This generated slightly more context for the debating AIs.

For the topic generation: Each LLMs mentioned above has been used.

    from ai_debater.prompt_engineering import TopicCreatorContext
    role = TopicCreatorContext()

    existing_topics = result_manager.load_topics()
    models, model_infos = generate_model(['GeminiChatter','MistralAIChatter','OpenAIChatter','Claude3AiChatter'])
    if existing_topics is not None:
        model_infos = model_infos.loc[model_infos.model_entity.isin(existing_topics.model_entity)==False]
        models=[] if model_infos.empty else [models[a] for a in model_infos.index]

    topics = {}
    for model in tqdm(models):
        model.initialise(role)
        topics[model.model_id()] = model.answer_until_valid([])

    # SavingOpenAIChatter
    if models:
        topics = pd.concat(topics)
        topics.index.names = ['model_id', topics.index.names[-1]]
        topics = topics.reset_index()
        topics['topic_id'] = topics['model_id'] + '-' + topics['ith_topic'].astype(str)
        topics.to_sql("topics", result_manager.connection, if_exists='append')
        model_infos.to_sql("model_infos", result_manager.connection, if_exists='append')
    existing_topics = result_manager.load_topics()
    existing_topics.head()

#### **Step 2: Debate Setup**

A debate involves two teams: a proposing team and an opposing team. It typically runs for a specified duration, during which team members take turns presenting and refuting arguments from the other team. Given that long debates might be challenging for LLMs due to potential loss of context over multiple chat iterations, I chose to limit debates to 4 turns.

To account for possible differences in LLM performance based on team role, I decided to have each pair of models participate as both the proposing and opposing team for a given topic. With four AI models and 30 topics, this results in 360 possible debates. To keep the scope manageable, I limited the debates to two scenarios:

1.  OpenAI vs Mistral AI
2.  Mistral AI vs Claude-3 AI

The LLMs will debate on their own topic: so that we can later look at potential topic biases, and a neutral topic: the one generated by Gemini.

**Data Modeling**

To retain essential information for later analysis, we want to store:

- The topic, identified by the topic-id
- The proposing model, identified by the model-id
- The opposing model, identified by the model-id
- The argument, argument position, and the model speaking

The data schema consists of two tables:

**fact_discourse:**

- A unique key: argument_id
- Argument,
- ith_argument,
- model_speaking: a foreign key corresponding to the model_id stored in model_infos
- discourse_id: a foreign key corresponding found in dim_discourse

**dim_discourse:**

- A unique key: discourse_id
- model_proposing, foreign key in model_infos
- model_opposing, foreign key in model_infos
- topic_id, foreign key in topics

**OpenAI vs Mistral AI Debate Setup**

For the first debate, I set up a competition between OpenAI and Mistral AI, following the guidelines and data modeling described above.

We first load the role Debater following the COSTAR prompt design.

    from ai_debater.debater_tools import run_debate
    from ai_debater.prompt_engineering import DebaterContext
    role = DebaterContext()

    model_in_competitions = ['MistralAIChatter','OpenAIChatter']
    models, model_infos = generate_model(model_in_competitions)
    need2save = True
    for model in tqdm(models):
        model.initialise(role)
    existing_competitions = result_manager.load_competitions()
    existing_topics = result_manager.load_topics()
    topics_creators = [
        'GeminiChatter|gemini-pro', # The neutral LLM
        'MistralAIChatter|mistral-large-latest',
        'OpenAIChatter|gpt-4'
        ]
    selected_topics = existing_topics.loc[existing_topics.model_entity.isin(topics_creators)]

    competing_models = permutations(models, 2)
    for topic_index, (prop, oppo) in tqdm(product(selected_topics.index, competing_models)):
        topic = existing_topics.loc[topic_index]
        current_topic_id = topic.topic_id
        competition_done = False
        if existing_competitions is not None:
            competition_done = existing_competitions.topic_id == current_topic_id
            competition_done&= existing_competitions.model_proposing_entity == prop.model_entity
            competition_done&= existing_competitions.model_opposing_entity == oppo.model_entity
            competition_done = competition_done.any()
        if competition_done:
            continue
        if need2save:
            model_infos.to_sql("model_infos", result_manager.connection, if_exists='append')
            need2save = False
        run_debate(topic, prop, oppo, result_manager.connection)

**Mistral AI vs Claude3 AI**

(Same as above but with Claude-3 instead of

    role = DebaterContext()
    models, model_infos = generate_model(['MistralAIChatter','Claude3AiChatter'])
    need2save = True
    for model in tqdm(models):
        model.initialise(role)
    existing_competitions = result_manager.load_competitions()
    existing_topics = result_manager.load_topics()
    topics_creators = [
        'GeminiChatter|gemini-pro', # The neutral LLM
        'MistralAIChatter|mistral-large-latest',
        'Claude3AiChatter|claude-3-opus-20240229'
        ]
    selected_topics = existing_topics.loc[existing_topics.model_entity.isin(topics_creators)]
    selected_topics = selected_topics.iloc[::-1]

    competing_models = permutations(models, 2)
    for topic_index, (prop, oppo) in tqdm(product(selected_topics.index, competing_models)):
        topic = existing_topics.loc[topic_index]
        current_topic_id = topic.topic_id
        competition_done = False
        if existing_competitions is not None:
            competition_done = existing_competitions.topic_id == current_topic_id
            competition_done&= existing_competitions.model_proposing_entity == prop.model_entity
            competition_done&= existing_competitions.model_opposing_entity == oppo.model_entity
            competition_done = competition_done.any()
        if competition_done:
            continue
        if need2save:
            model_infos.to_sql("model_infos", result_manager.connection, if_exists='append')
            need2save = False
        run_debate(topic, prop, oppo, result_manager.connection)

#### **Step 3: Evaluation by the Jury**

Once the debate has been concluded, it’s necessary to assess the quality of the teams’ responses and their roles in the debate. Debates can be evaluated in five categories,

- Reasoning and Evidence,
- Listening and Response,
- Organisation and Prioritisation,
- Expression and Delivery, and
- Teamwork and Roles.

Each judge will be responsible for grading the participants in each category and providing a rationale for their grade. This rationale is crucial for comparing different judgments on the same debate (see public voting below).

**Data Modeling for Judgments:**

To analyze the judgments later, we need to capture the following information:

- The debate being judged: discourse ID
- The model responsible for judging: model ID
- The resulting judgments: Category, score, team, and rationale

We will store this information in the following tables:

**fact_judgments:**

- Score,
- categories,
- team ID, and rational
- An ID to identify the judgment: judgment_id

**dim_judgments:**

- Judgment_id: a unique key
- Discourse_id,
- model_id_judging

**Judgements**

We first need to prepare the input for the judgement. This consist of formatting a given debate according to the expected input given by the COSTAR defined role. This preprocessing was wrapped in the function \`discourse2input\`.

    from ai_debater.prompt_interface import discourse2input
    from ai_debater.prompt_engineering import JudgesContext
    role = JudgesContext()

    models, model_infos = generate_model(['GeminiChatter','MistralAIChatter','OpenAIChatter', 'Claude3AiChatter']) # '
    need2save = True
    for model in tqdm(models):
        model.initialise(role)

    enriched_judgements = result_manager.load_enriched_judgements()
    existing_competitions = result_manager.load_competitions()

    with tqdm(total=len(models)*existing_competitions.shape[0]) as pbar:
        for _, competition in tqdm(existing_competitions.iterrows(), "Judging competitions", total=existing_competitions.shape[0]):
            discourse_id = competition.discourse_id
            discourse = result_manager.load_discourse(discourse_id)
            message = discourse2input(discourse)
            for judge in models:
                model_id_judging = judge.model_id()
                judgement_id = model_id_judging+':'+discourse_id
                if enriched_judgements is not None:
                    condition = enriched_judgements.discourse_id == discourse_id
                    condition&= enriched_judgements.judge_model_entity == judge.model_entity
                    condition = condition.any()
                    if condition:
                        pbar.update(1)
                        continue
                result = judge.answer_until_valid([message])
                if need2save:
                    model_infos.to_sql("model_infos", result_manager.connection, if_exists='append')
                    need2save = False
                fact_judgements = result.copy().reset_index()
                fact_judgements['judgement_id'] = judgement_id

                dim_judgements = pd.Series({'judgement_id': judgement_id,
                                            'discourse_id': discourse_id,
                                            'model_id_judging': model_id_judging})

                dim_judgements.to_frame().transpose().to_sql("dim_judgements", result_manager.connection, if_exists='append')
                fact_judgements.to_sql("fact_judgements", result_manager.connection, if_exists='append')
                pbar.update(1)

#### **Step 4: Public-voting**

In a open debate, the public sometimes rate the audience. Often done with a clap-o-meter or counting raised hands, at the core we have the audience rating the participant. They may have heard the verdict of the jury prior to making their decision, and are therefore agreeing or disagreeing with a given judge rating. The agreement with a judgement is interesting in this AI debating scenarios as an AI will first judge and them based on its own judgement as well the judgement of others will need to decide which judgement is the most appropriate, i.e. changing its mind or keeping its line of reasoning. To avoid for an obvious link, by using the model name, the judge identity and the public identity are both unknown to the model.

**Data Modeling**

To retain essential information for later analysis, we want to store:

- The dicourse-id,
- The judegement ids: here we have one per judges, so four
- The id of the model acting as the public: public_id

The data schema consists of two tables:

**fact_public:**

- judgement_id: The chosen judgement
- A unique key: public_voting_id,

**dim_public:**

- Unique key: public_voting_id
- discourse_id,
- public_model_id,
- judgement_ids: an Array of judgements

#### Voting

Similarly has above we define a role (PublicContext) and a function to preprocess the judgement and discourse.

This was not sufficient for MistralAI and OpenAI API. Their responses were not following the expected output given by the COSTAR. I therefore added two messages: one acknowledging the judgement and discourse, and the other providing the expected output.

    from ai_debater.prompt_interface import judgement_and_discourse2input
    from ai_debater.prompt_engineering import PublicContext

    role = PublicContext()

    judgements = result_manager.load_judgements()
    judged_discourses = judgements.discourse_id.unique()
    public_votes = result_manager.load_enriched_public()

    models, model_infos = generate_model(['GeminiChatter','MistralAIChatter','OpenAIChatter', 'Claude3AiChatter'])
    need2save = True
    for model in tqdm(models):
        model.initialise(role)
    for discourse_id in tqdm(judged_discourses):
        message, judgement_ids = judgement_and_discourse2input(discourse_id=discourse_id, result_manager=result_manager)
        messages = [message]
        # I needed to add this, because MistralAI and OpenAI failed to respond with the correct output.
        messages.append({'role':'system', 'content': 'thank you for providing the information. In which format should I answer?'})
        messages.append({'role':'user', 'content': 'Please give the judgement id as: <Judgement_ID></Judgement_ID>'})
        for model in models:
            public_model_id = model.model_id()
            public_voting_id = discourse_id+'|'+public_model_id
            if public_votes is not None:
                condition = public_votes.discourse_id == discourse_id
                condition&= public_votes.public_model_entity == model.model_entity
                condition = condition.any()
                if condition:
                    pbar.update(1)
                    continue

            res = model.answer_until_valid(messages)

            if need2save:
                model_infos.to_sql("model_infos", result_manager.connection, if_exists='append')
                need2save = False
            fact_public = res.to_frame().transpose()
            fact_public['public_voting_id'] = public_voting_id
            dim_public = pd.Series({'public_voting_id': public_voting_id,
                                    'discourse_id': discourse_id,
                                    'public_model_id': public_model_id,
                                    'judgement_ids': judgement_ids})

            dim_public.to_frame().transpose().to_sql("dim_public", result_manager.connection, if_exists='append')
            fact_public.to_sql("fact_public", result_manager.connection, if_exists='append')

### **Data modelling summary**

![](/assets/images/posts/ai-vs-ai-letting-large-language-models-competing-and-judging-themself/01-1892e7086ad9.png)

### Conclusion

Four large language models has been engaging in a debate from topic generation, to public voting, via dual debate and judgements. The role of the LLMs were given by a prompt inspired by the COSTAR prompt engineering. The LLMs engage in the task and manage to respond according to the expectation, except for the later step were some of the context were lost, potentially due to the length of the discourse and judgements.

You can find the code here: <https://github.com/bolirev/ai-debate>

### **Further reading:**

#### **Pricing**

- <https://docs.mistral.ai/platform/pricing/>
- <https://openai.com/pricing>
- <https://cloud.google.com/vertex-ai/generative-ai/pricing>
- <https://www.anthropic.com/api> (Scroll a bit)

#### **Interesting links**

**Gemini vs OpenAI:**

- <https://medium.com/@csakash03/googles-gemini-vs-openai-s-chatgpt-4d23288f7769>
- <https://www.thepromptindex.com/evaluating-googles-new-gemini-language-model-how-does-it-stack-up-against-gpt-3-and-gpt-4.html>

**Claude 3v vs Chat GPT:**

- <https://www.analyticsvidhya.com/blog/2024/03/claude-3-sonnet-vs-chatgpt-3-5/>

**How-Tos — Vertex AI:**

- [https://cloud.google.com/python/docs/reference/aiplatform/latest/vertexai.generative_models.GenerativeModel#vertexai_generative_models_GenerativeModel_start_cha](https://cloud.google.com/python/docs/reference/aiplatform/latest/vertexai.generative_models.GenerativeModel#vertexai_generative_models_GenerativeModel_start_chat)t
- [https://cloud.google.com/python/docs/reference/aiplatform/latest/vertexai.generative_models.ChatSessio](https://cloud.google.com/python/docs/reference/aiplatform/latest/vertexai.generative_models.ChatSession)n
- Setting vertexai up: <https://cloud.google.com/vertex-ai/docs/start/cloud-environment>
- <https://blog.google/technology/ai/google-gemini-ai/#performance>

**How-Tos — Claude-3**

- <https://docs.anthropic.com/claude/docs/intro-to-claude>

**How-Tos — OpenAi:**

- <https://platform.openai.com/docs/introduction>

**How-Tos — MistralAI:**

- <https://docs.mistral.ai/>

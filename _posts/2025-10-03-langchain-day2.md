---
layout: post
title: "Langchain Day2"
date: 2025-10-03
categories: [langchain, ai]
tags: [langchain, ai]
---

[Github link](https://github.com/jaehun221/Langchain_Study)

## Model I/O

### FewShotPromptTemplate
```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts.few_shot import FewShotPromptTemplate
from langchain.callbacks import StreamingStdOutCallbackHandler
from langchain.prompts import PromptTemplate

# 예시를 주고 그것을 기반으로 답변을 할 수있게 함

chat = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.1,
    streaming=True,
    callbacks=[
        StreamingStdOutCallbackHandler()
    ],
)

examples = [
    {
        "question": "What do you know about France?",
        "answer": """
        Here is what I know:
        Capital: Paris
        Language: French
        Food: Wine and Cheese
        Currency: Euro
        """,
    },
    {
        "question": "What do you know about Italy?",
        "answer": """
        I know this:
        Capital: Rome
        Language: Italian
        Food: Pizza and Pasta
        Currency: Euro
        """,
    },
    {
        "question": "What do you know about Greece?",
        "answer": """
        I know this:
        Capital: Athens
        Language: Greek
        Food: Souvlaki and Feta Cheese
        Currency: Euro
        """,
    },
]
```
위와 같은 형식으로 example에 예시를 몇가지 넣어두었다.

```python
example_prompt = PromptTemplate.from_template("Human: {question}\nAI:{answer}")


prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    suffix="Human: What do you know about {country}?",
    input_variables=["country"],
)

chain = prompt | chat

chain.invoke({"country": "Spain"})
```
`FewShotPromptTemplate`: 위 코드와 같이 예시를 주고 그것을 기반으로 답변을 할수 있도록 하는 방식이다.
<br/>

### LengthBaseExampleSelector

```python
from langchain_openai import ChatOpenAI
from langchain.prompts.few_shot import FewShotPromptTemplate
from langchain.callbacks import StreamingStdOutCallbackHandler
from langchain.prompts import PromptTemplate
from langchain.prompts.example_selector import LengthBasedExampleSelector

# example = ...

example_prompt = PromptTemplate.from_template("Human: {question}\nAI: {answer}")

example_selector = LengthBasedExampleSelector(
    examples=examples,
    example_prompt=example_prompt,
    max_length=80
)

prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    suffix="Human: What do you know about {country}?",
    input_variables=["country"]
)

chain = prompt | chat

chain.invoke({"country": "Spain"})
```

아까와 같이 example 변수에 예시를 주었을 때 LengthBasedExampleSelector를 사용해 `max_length`를 지정해줄 수 있다.<br/>
max_length가 80이라고 할 때 총 80글자가 넘지 않도록 예시를 선택한다<br/>
ex) length = 30인 example이 3개 있을 때 80이 넘지 않도록 2개의 예시만 사용




### Serialization

```python
from langchain_openai import ChatOpenAI
from langchain.callbacks import StreamingStdOutCallbackHandler
from langchain.prompts import load_prompt

chat = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.1,
    streaming=True,
    callbacks=[
        StreamingStdOutCallbackHandler(),
    ]
)

prompt = load_prompt("./prompt.json")

prompt.format(country="Germmany")

chain = prompt | chat

chain.invoke({
    "country": "Germmany"
})
```

위와 같은 방식으로 json 파일이나 yaml파일을 읽을 수 있다.<br/>
위 코드에서 사용한 prompt.json파일 내용은 아래와 같다.<br/>
```json
{
    "_type": "prompt",
    "template": "What is the capital of {country}",
    "input_variables": ["country"]
}
```

### Composition
```python
from langchain.chat_models import ChatOpenAI
from langchain.callbacks import StreamingStdOutCallbackHandler
from langchain.prompts import PromptTemplate
from langchain.prompts.pipeline import PipelinePromptTemplate

chat = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.1,
    streaming=True,
    callbacks=[
        StreamingStdOutCallbackHandler(),
    ]
)

intro = PromptTemplate.from_template(
    """
    You are a role playing assistant.
    And you are impersonating a {character}
    """
)

example = PromptTemplate.from_template(
    """
    this is an example of how you talk:

    Human: {example_question}
    You: {example_answer}
    """
)

start = PromptTemplate.from_template(
    """
    Start now!

    Human: {question}
    You:
    """
)

final = PromptTemplate.from_template(
    """
    {intro}

    {example}

    {start}
    """
)

prompts = [
    ("intro", intro),
    ("example", example),
    ("start", start)
]

full_prompt = PipelinePromptTemplate(
    final_prompt=final,
    pipeline_prompts=prompts
)

chain = full_prompt | chat

chain.invoke(
    {
        "character": "Pirate",
        "example_question": "What is your location?",
        "example_answer": "Arrrrg! That is a secret!! Arg arg!!",
        "question": "What is your fav food?",
    }
)
```

langchain composition은 위와같이 여러개의 prompt template을 독립된 컴포넌트로 정의한 뒤<br/>
`PipelinePromptTemplate`로 연결해 준다.<br/>
이후 완성된 프롬프트를 `|`를 통해 chat과 연결하고 invoke를 통해 질문한다.

day2 end
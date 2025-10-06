---
layout: post
title: "Langchain Day1"
date: 2025-09-26
categories: [langchain, ai]
tags: [langchain, ai]
---

[Github link](https://github.com/jaehun221/Langchain_Study)

### 폴더구조<br/>
LANGCHAIN_STUDY<br/>
├── .venv/<br/>
├── days/<br/>
├── .env<br/>
├── .gitignore<br/>
├── .python-version<br/>
├── cli_gpt.py<br/>
├── LICENSE<br/>
├── main.py<br/>
├── pyproject.toml<br/>
├── README.md<br/>
└── uv.lock


### 환경 설정<br/>
[uv(python pakage 관리) 설치](https://docs.astral.sh/uv/getting-started/installation/)

- `.env`파일을 만들고 `OPENAI_API_KEY="sk-p..."` 형식으로 발급받은 api key를 작성한다.
- `uv init`
- `uv venv`
- `uv add --group dev ipykernel`
- `uv add langchain langchain-core langchain-community langchain-openai`

## 주요 메서드 및 문법

```python
from langchain_openai import ChatOpenAI, OpenAI

chat = ChatOpenAI(
    model="gpt-3.5-turbo" # 사용할 모델 명시
)

print(chat.invoke("How many planets are there?"))
```
`ChatOpenAI`: OpenAI 기반 챗 모델을 Langchain에서 사용하기 위한 클래스이다.
`invoke`: LangChain에서 chain이나 Runnable 객체를 한 번 실행해서 결과를 반환하는 메서드로<br/>
질문을 던지고 바로 답을 받는 Langchain에서 가장 기본적인 메서드이다.


### Message Schema

```python
from langchain_openai import ChatOpenAI
from langchain.schema import HumanMessage, AIMessage, SystemMessage

chat = ChatOpenAI(
    temperature=0.1
)

messages = [
    SystemMessage(
        content="You are a geography expert. And you only reply in {language}"
    ),
    AIMessage(
        content="안녕, 난 {name}!"
    ),
    HumanMessage(
        content="What is the distance between {country_a} and {country_b}. Also, what is your name?"
    )
]
```

`SystemMessage`: model의 동작 방식이나 역할들을 지정할 수 있다. 예시로 "당신은 지리 전문가입니다." 등등
`AIMessage`: AI가 응답하는 메시지를 의미한다.
`HumanMessage`: 사용자의 질문이 들어간다.

messages를 `chat.invoke(messages)`로 전달하면 AI가 답변을 생성해 준다.<br/>
이러한 메시지로 구조화된 대화를 만들 수 있다.


### Message Template

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate, ChatPromptTemplate

chat = ChatOpenAI(
    temperature=0.1
)
# PromptTemplate
template = PromptTemplate.from_template(
    "What is the distance between {country_a} and {country_b}"
)

prompt = template.format(country_a="korea", country_b="Japan")

chat.invoke(prompt)
```

`PromptTemplate`: 하나의 질문 template를 만들어 재사용 할 수 있다. `{placeholders}`와 같이 변수를 넣고<br/> 
.format을 사용하여 값을 넣어줄 수 있다.


```python
# ChatPromptTemplate
template = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a geography expert. And you only reply in {langauge}"),
        ("ai", "Ciao, mi chiamo {name}!"),
        (
            "human",
            "What is the distance between {country_a} and {country_b}. Also, what is your name?"
        )
    ]
)

prompt = template.format_messages(
    langauge="Korean",
    name="Jaehun Lee",
    country_a="Spain",
    country_b="Italia"
)

chat.invoke(prompt)
```

`ChatPromptTemplate`: 여러 메시지로 chat형 대화 template을 만들 수 있다. system, ai, human을 구분해<br/>
실제 대화 흐름처럼 구성할 수 있다. .format_messages()를 사용해 {placeholders}에 값을 넣어 최종 프롬프트를 완성한다.

`chat.invoke(prompt)`로 template을 챗봇한테 넘겨줘 답변을 생성할 수 있다.


### Output Parser
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatMessagePromptTemplate
from langchain.schema import BaseOutputParser

chat = ChatOpenAI(
    temperature=0.1
)

# a, b, c, d

class CommaOutputParser(BaseOutputParser):
    def parse(self, text):
        items = text.strip().split(",")
        return list(map(str.strip, items))


template = ChatPromptTemplate(
    [
        ("system",
         "You are a list generating machine. Everything you asked will be answerd with a comma separated list of max {max_items} in lowercase. DO NOT reply anything else."),
         ("human", "{question}")
    ]   
)

chain = template | chat | CommaOutputParser()

chain.invoke(
    {
        "max_items" : 5,
        "question" : "What atre the pokemons?"
    }
)
```
**출력 결과**<br/>
['pikachu', 'charmander', 'bulbasaur', 'squirtle', 'jigglypuff']

`BaseOutputParser`: 모델의 출력을 원하는 형태로 가공하기 위해 상속받는 클래스이다. 여기서는 CommaOutputParser로
**,** 로 구분된 리스트 형태로 출력하게 구현했다.



### Chaining Chains
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain.callbacks import StreamingStdOutCallbackHandler

chat = ChatOpenAI(
    temperature=0.1,
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()]
)

chef_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "",
            
        ),
        ("human", " I want to cook {cuisine} food. ")
    ]
)

chef_chain = chef_prompt | chat


veg_chef_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a vegetarian chef specialized on making traditional recipies vegetarian. You find alternative ingredients and explain their preparation. You don't radically modify the recipe. If there is no alternative for a food just say you don't know how to replace it."),
        ("human", "{recipe}")
    ]
)

veg_chain = veg_chef_prompt | chat

final_chain = chef_chain | veg_chain

final_chain.invoke(
    {
        "cuisine": "Italian"
    }
)
```

`streaming=True, callbacks=[StreamingStdOutCallbackHandler()]` : 모델이 각 token이 생성될때 마다 콜백 함수를 호출해<br/>
실시간으로 답변을 출력해준다.

template와 model을 `|`로 묶어 하나의 처리 단위인 chain을 생성할 수 있다.<br/> 
또한 위 코드처럼 `chef_chain | veg_chain`과 같이 model의 출력 **결과**를 **입력**으로 넘겨주는 방식으로 파이프라인을 만들 수 있다.


day1 end
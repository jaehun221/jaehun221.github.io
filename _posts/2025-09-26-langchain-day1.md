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

`invoke`: LangChain에서 chain이나 Runnable 객체를 한 번 실행해서 결과를 반환하는 메서드로<br/>
질문을 던지고 바로 답을 받는 Langchain에서 가장 기본적인 메서드이다.


#### Message Schema

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


<작성중...>
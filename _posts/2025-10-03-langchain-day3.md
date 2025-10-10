---
layout: post
title: "Langchain Day3"
date: 2025-10-03
categories: [langchain, ai]
tags: [langchain, ai]
---

[Github link](https://github.com/jaehun221/Langchain_Study)

## Memory

### Caching
```python
from langchain_openai import ChatOpenAI
from langchain.globals import set_llm_cache, set_debug
from langchain.cache import InMemoryCache, SQLiteCache

# set_llm_cache(InMemoryCache())
set_llm_cache(SQLiteCache("cache.db"))
set_debug(False)


chat = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.1
)

chat.predict("What is the capital of Korea.")
```
`set_llm_cache(SQLiteCache("cache.db"))`: 입/출력에 관한 정보들을 저장해 두고 같은 질문이<br/>
들어오면 이전 답변을 바로 반환해준다.<br/>
처음 `chat.predict("What is the capital of Korea.")`라는 질문을 했을 때 2.1s가 소요되지만<br/>
같은 질문을 다시 하면 0.0s로 답변을 바로 출력해주는 것을 볼 수 있다.


### ConversationBufferMemory
```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(return_messages=True)

memory.save_context({"input" : "Hi"}, {"output" : "How are you?"})

memory.load_memory_variables({})
```

`ConversationBufferMemory`: 대화 내용을 순서대로 버퍼에 저장하는 클래스이다.<br/>
`return_messages=True`로 설정할 시 문자열이 아닌 HumanMessage, AIMessage형식으로 반환해준다.<br/>

`memory.load_memory_variables({})`<br/>출력결과
```
{'history': [HumanMessage(content='Hi', additional_kwargs={}, response_metadata={}),
  AIMessage(content='How are you?', additional_kwargs={}, response_metadata={})]}
```


### ConversationBufferWindowMemory
```python
from langchain.memory import ConversationBufferWindowMemory

memory = ConversationBufferWindowMemory(
    return_messages=True,
    k=5 # 기억할 대화 수
)


def add_message(input, output):
    memory.save_context({"input" : input}, {"output" : output})

add_message("1", "1")
```

`ConversationBufferWindowMemory`: ConversationBufferMemory과 거의 비슷하지만 최근 k개의<br/>메시지만 기억하도록 할 수 있다.<br/> k=5일 때 5를 초과하면 가장 오래된 대화부터 삭제되는 것을 확인할 수 있음.


### ConversationSummaryMemory
```python
from langchain.memory import ConversationSummaryMemory
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(temperature=0.1)

memory = ConversationSummaryMemory(llm=llm)

def add_message(input, output):
    memory.save_context({"input" : input}, {"output" : output})

def get_history():
    return memory.load_memory_variables({})

add_message("Hi I'm Jaehun, I live in South Korea", "Wow that is so cool!")
```
`ConversationSummaryMemory`: 대화 내용을 그냥 저장하지 않고 요약해서 저장한다.<br/>
따라서 메모리 사용량과 토큰 소비량을 줄일 수 있고 LLM이 중요한 맥락만 기억하도록 효율적으로 관리할  수 있다.


### Memory on LLM Chain
```python
# LLm = Off the shlf : General Perpose chain (일반적인 목적을 가지는 체인)

from langchain.memory import ConversationSummaryBufferMemory
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.1
)


memory = ConversationSummaryBufferMemory(
    llm=llm,
    max_token_limit=80,
    memory_key="chat_history",
    return_messages=True
)

template = """

    you are a helpful AI taling a human.

    {chat_history}
    Human: {question}
    Your:

"""

chain = LLMChain(
    llm=llm,
    memory=memory,
    prompt=PromptTemplate.from_template(template=template)
)

chain.predict(question="my name is jaehun.")
```

SummaryBufferMemory와 max_token_limit을 사용해 80토큰을 넘기지 않도록 요약해서 대화기록을 저장하도록 했다.<br/>
`memory_key`: 저장된 대화 기록을 prompt에 넣을 때 사용할 키 이름이다.<br/>
`memory_key="chat_history` 일 때 template 안에 `{chat_history}` 형식으로 넣는다.<br/>
이후 LLMChain을 사용해서 model, prompt, memory를 연결한다.
<br/>
<br/>

위에서 chain.predict로 이름을 알려준 것을 확인할 수 있다.
`chain.predict(question="what is my name?")`
출력
```'Your name is Jaehun! How can I help you today?'```

미리 알려줬던 이름대로 잘 답변을 해준다!

day3 end
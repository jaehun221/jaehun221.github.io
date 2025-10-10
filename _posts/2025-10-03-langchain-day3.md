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

<작성중...>
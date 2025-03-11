---
layout:       post
title:        "AI教我做事之RAG开发-2 开发一个Restful接口"
author:       "Ramendeus"
header-style: text
catalog:      true
tags:
    - AI Agent
    - RAG
    - Grok
    - AI教我做事
---

## AI教我做事之RAG开发-2 开发一个Restful接口


这次的课程就高级了。

我们将之前的RAG内容内聚成一个服务，希望外部以一个API接口的形式调用。

因此，我们需要内置一个web服务器，python下迅速搞定

![AI教我做事之RAG开发-2 开发一个Restful接口](https://www.shxcj.com/wp-content/uploads/2025/03/14bb1b8d-26fc-40f9-9058-0655408ad26c.png)

## 1 全部代码如下

```
#from langchain.document_loaders import PyPDFLoader
from langchain_community.document_loaders import PyPDFLoader

from langchain.text_splitter import RecursiveCharacterTextSplitter

#from langchain_community.llms import Ollama
from langchain_ollama import OllamaLLM
#from langchain.embeddings import HuggingFaceBgeEmbeddings
from langchain_huggingface import HuggingFaceEmbeddings
#from langchain.vectorstores import FAISS
from langchain_community.vectorstores import FAISS

#from langchain.llms import HuggingFacePipeline
from langchain_community.llms import HuggingFacePipeline

from langchain.chains import RetrievalQA
from transformers import pipeline
from langchain.prompts import PromptTemplate

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from transformers import pipeline
import os

# 初始化FastAPI应用
app = FastAPI(title="RAG API", description="A simple RAG API for retrieving answers from documents")

# 定义请求体模型
class QueryRequest(BaseModel):
    query: str

# 全局变量存储RAG链（避免重复初始化）
rag_chain = None
vector_store = None

# 1. 加载和处理文档
def load_and_split_documents(file_path):
    if not os.path.exists(file_path):
        raise ValueError(f"文件 {file_path} 不存在")
    loader = PyPDFLoader(file_path)
    documents = loader.load()
    text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
    docs = text_splitter.split_documents(documents)
    return docs

# 2. 创建向量存储
def create_vector_store(docs):
    embeddings = HuggingFaceEmbeddings(model_name="sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2")
    vector_store = FAISS.from_documents(docs, embeddings)
    return vector_store

# 3. 初始化语言模型
def initialize_llm():
    myLLM = OllamaLLM(model="llama2")
    return myLLM

# 4. 创建RAG链
def create_rag_chain(vector_store, llm):
    prompt_template = """根据以下上下文，直接用中文回答问题，不要添加多余内容。

上下文: {context}

问题: {question}

回答: """
    PROMPT = PromptTemplate(template=prompt_template, input_variables=["context", "question"])
    qa_chain = RetrievalQA.from_chain_type(
        llm=llm,
        chain_type="stuff",
        retriever=vector_store.as_retriever(search_kwargs={"k": 3}),
        return_source_documents=False,
        chain_type_kwargs={"prompt": PROMPT}
    )
    return qa_chain

# 5. 初始化服务
def initialize_service(file_path="example.pdf"):
    global rag_chain, vector_store
    print("加载和分割文档...")
    docs = load_and_split_documents(file_path)
    print("创建向量存储...")
    vector_store = create_vector_store(docs)
    print("初始化语言模型...")
    llm = initialize_llm()
    print("创建RAG链...")
    rag_chain = create_rag_chain(vector_store, llm)
    print("服务初始化完成")

# 6. API端点：查询接口
@app.post("/query", response_model=dict)
async def query_document(request: QueryRequest):
    if rag_chain is None:
        raise HTTPException(status_code=503, detail="服务尚未初始化，请等待或联系管理员")
    try:
        result = rag_chain({"query": request.query})
        return {"answer": result["result"].strip()}
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"查询失败: {str(e)}")

# 7. 启动时初始化
@app.lifespan("startup")
async def startup_event():
    initialize_service("D:\\RAG-TestImage-3.pdf")  # 替换为你的PDF路径

# 运行服务
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=9897)
```

使用方法，启动该程序后。

用curl

curl -X POST “http://localhost:8000/query” -H “Content-Type: application/json” -d ‘{“query”: “文档的主要主题是什么？”}’

响应

{“answer”: “xxxxxxx “}

## 2 部署与第三方调用

1.  **本地测试**
    1.  运行uvicorn app:app –host 0.0.0.0 –port 8000即可在本地访问。
    2.  第三方可通过局域网IP（如http://192.168.x.x:8000/query）调用。
    

## 3 示例调用（第三方视角）

+   **Python客户端**：

```
import requests
url = "http://localhost:8000/query"
data = {"query": "文档的主要主题是什么？"}
response = requests.post(url, json=data)
print(response.json())  # 输出: {"answer": "人工智能"}
```

+   **安全性建议**：
    +   添加API密钥验证：


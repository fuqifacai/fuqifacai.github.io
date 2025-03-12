---
layout:       post
title:        "AI教我做事之RAG开发-10 如何用LlamaIndex写代码本文主要的功能"
author:       "Ramendeus"
header-style: text
catalog:      true
tags:
    - AI Agent
    - RAG
    - Grok
    - AI教我做事
    - LlamaIndex
---

## AI教我做事之RAG开发-10 如何用LlamaIndex写代码本文主要的功能：


## 本文主要的功能：

1.  完整读取本地某个目录下的文件作为数据源
2.  用Ollama本地模型和HF的嵌入模型
3.  进行问答功能，测试RAG的检索命中情况。测试良好。

注意点：

1.  有些引用自行安装。甚至代码没有被直接引用到

## 完整代码

```
# 环境安装（先执行）
# pip install llama-index-core llama-index-llms-ollama llama-index-readers-file llama-index-llms-openai llama-index-embeddings-openai pypdf

import os
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader, Settings
from llama_index.core.node_parser import SimpleNodeParser
from pathlib import Path
from llama_index.core import VectorStoreIndex
from llama_index.llms.huggingface import HuggingFaceLLM
from llama_index.embeddings.huggingface import HuggingFaceEmbedding
from llama_index.core import Settings
from llama_index.core.node_parser import SimpleNodeParser
from llama_index.llms.ollama import Ollama

from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_ollama import OllamaLLM
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_community.vectorstores import FAISS
from langchain_community.llms import HuggingFacePipeline

from langchain.chains import RetrievalQA
from transformers import pipeline
from langchain.prompts import PromptTemplate

# 配置参数
documents_dir = Path('testdata')   # 存放文档的目录，这个位置需要你修改成自己的
cache_dir = "./cache"  # 缓存目录
os.environ["OPENAI_API_KEY"] = "sk-your-key"  # 替换为你的OpenAI API密钥,目前我们没有用OPENAI的

# 1. 文档加载与处理
def load_documents():
    # 支持PDF、Word、TXT等多种格式
    reader = SimpleDirectoryReader(
        input_dir=documents_dir,
        recursive=True  # 递归读取子目录
    )
    documents = reader.load_data()
    print(f"已加载 {len(documents)} 个文档")
    return documents

# 2. 文本分块处理
def chunk_documents(documents):
    # 配置分块参数
    node_parser = SimpleNodeParser.from_defaults(
        chunk_size=1024,  # 块大小
        chunk_overlap=200,  # 块重叠
    )
    nodes = node_parser.get_nodes_from_documents(documents)
    print(f"生成 {len(nodes)} 个文本块")
    return nodes

# 3. 初始化模型
def setup_models():
    # 配置嵌入模型
    Settings.embed_model = HuggingFaceEmbedding(model_name="sentence-transformers/all-MiniLM-L6-v2")
    
    # 配置LLM
    # Settings.llm = HuggingFaceLLM(model_name="gpt2")
    Settings.llm = Ollama(
        model="qwen2",  # 改为你本地的模型名称
        base_url="http://localhost:11434",  # Ollama默认服务地址
        temperature=0.3,
        model_kwargs={"num_gpu": 1},  # 启用GPU加速
        request_timeout=180  # 本地模型需要更长的响应时间
    )
    # 这个又依赖langchain下的llms了
    # Settings.llm  = OllamaLLM(model="qwen2")

# 4. 构建索引
def build_index(nodes):
    index = VectorStoreIndex(nodes)
    index.storage_context.persist(persist_dir=cache_dir)
    print("索引构建完成")
    return index

# 5. 查询处理
def query_engine(index):
    query_engine = index.as_query_engine(
        similarity_top_k=3,  # 检索前3个相关块
        verbose=True  # 显示详细过程
    )
    return query_engine

# 完整流程
def main():
    # 初始化
    setup_models()
    
    # 文档处理
    documents = load_documents()
    nodes = chunk_documents(documents)
    
    # 构建索引
    index = build_index(nodes)
    
    # 创建查询引擎
    engine = query_engine(index)
    
    # 执行查询
    while True:
        query = input("\n请输入问题（输入q退出）: ")
        if query.lower() == "q":
            break
        response = engine.query(query)
        print("\n答案：")
        print(response.response)
        print("\n来源：")
        for node in response.source_nodes:
            print(f"- 文档: {node.metadata['file_path']} (相似度: {node.score:.3f})")

if __name__ == "__main__":
    main()
```

## 代码改进说明：

1.  **模型配置优化**：
    1.  使用`BAAI/bge-small-en-v1.5`嵌入模型（性能接近OpenAI）
    2.  设置`request_timeout=180`适应本地模型较慢的响应
    3.  调整`temperature=0.3`平衡创造性和准确性
2.  **参数调优**：
    1.  `chunk_size=512`：适配本地模型的上下文窗口限制
    2.  `similarity_top_k=4`：增加检索上下文数量
    3.  `response_mode="compact"`：优化prompt结构
3.  **错误处理**：
    1.  添加了基本的异常捕获
    2.  显示生成错误信息


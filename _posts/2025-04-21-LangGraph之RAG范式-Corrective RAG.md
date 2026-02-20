---
layout:       post
title:        "LangGraph之RAG范式-Corrective RAG"
author:       "Ramendeus"
header-style: text
catalog:      true
tags:
    - WebRTC
    - Google
    - Web Streaming
    - Remote Meeting
    - Server side rendering
    - 云渲染
    - 远程桌面
    - Remote Desktop
    - Audio & Video Framework
---

### 1\. **什么是 Corrective RAG (CRAG)？**

**Corrective RAG (CRAG)** 是一种改进的 RAG 技术，旨在解决传统 RAG 的局限性。传统 RAG 通过从外部知识库检索相关文档来增强语言模型的生成能力，但如果检索到的文档不准确或不相关，可能会导致生成的内容出现误导或错误。CRAG 通过引入 **自我反思（self-reflection）** 和 **自我评估（self-grading）** 机制，在检索和生成过程中对文档的质量进行评估和修正，从而提高生成内容的准确性和相关性。

**CRAG 的核心思想**：

+   **评估检索文档**：检查检索到的文档是否与用户查询相关。
+   **修正机制**：如果文档不相关或质量不足，触发补充检索（如 Web 搜索）或重写查询。
+   **跳过知识细化（可选）**：在 LangGraph 的实现中，为了简化流程，通常跳过了文档的知识细化步骤（将文档拆分为“知识片段”并逐一评估）。

**优势**：

+   减少因检索到无关文档导致的生成错误（幻觉问题）。
+   提高生成内容的精确性和上下文相关性。
+   通过动态补充外部数据（如 Web 搜索），增强模型对最新或缺失信息的处理能力。

![LangGraph之RAG范式-Corrective RAG](https://www.shxcj.com/wp-content/uploads/2025/04/6081a7a35b4d1c84e869aac5935f172-819x1024.jpg)

* * *

### 2\. **LangGraph 中 CRAG 的实现逻辑**

在 LangGraph 的 examples/rag/langgraph\_crag.ipynb 示例中，CRAG 被实现为一个基于 **状态图（StateGraph）** 的工作流。

![LangGraph之RAG范式-Corrective RAG](https://www.shxcj.com/wp-content/uploads/2025/04/395e35a6-3299-4e70-9bc4-b17ccacffe51.png)

CRAG 的工作流包括以下关键步骤：

1.  **检索文档（Retrieve）**：从向量数据库中检索与用户查询相关的文档。
2.  **评估文档相关性（Grade Documents）**：对检索到的文档进行相关性评分，判断是否满足生成要求。
3.  **决定是否生成（Decide to Generate）**：根据文档评分决定是直接生成答案、触发 Web 搜索，还是重写查询。
4.  **重写查询（Transform** **Query****）**：如果文档不相关，优化查询以便进行更有效的检索。
5.  **Web 搜索（Web Search）**：当检索到的文档不足以回答问题时，使用外部搜索补充信息。
6.  **生成答案（Generate）**：基于最终的文档集合生成答案。

* * *

### 3\. **LangGraph CRAG 示例代码解析**

以下结合代码片段说明其工作原理。

#### 3.1 **定义图状态（GraphState）**

LangGraph 使用一个状态字典来跟踪工作流中的数据。CRAG 的状态包括：

+   question：用户输入的查询。
+   generation：最终生成的答案。
+   web\_search：是否需要进行 Web 搜索。
+   documents：检索到的文档列表。

```
from typing import List
from typing_extensions import TypedDict
class GraphState(TypedDict):
    question: str
    generation: str
    web_search: str
    documents: List[str]
```

**解释**：这个状态类定义了工作流中每个节点共享和更新的数据结构，确保信息在节点间传递。

#### 3.2 **创建索引**

示例中使用了 **Chroma** 向量数据库来存储文档嵌入，并通过 **OpenAIEmbeddings** 生成文档的向量表示。

在我们的Demo中我们用HuggingFaceEmbeddings代替了**OpenAIEmbeddings**

需要引用**from** langchain\_huggingface **import** HuggingFaceEmbeddings。

同时要注意，下述代码中提供的几个URL可能失效或者无法访问，请修改成自己的。 **解释**：

+   从指定 URL 加载博客文章并拆分为小块（每块 250 个 token）。
+   使用 OpenAI 的嵌入模型将文档转化为向量，存储在 Chroma 数据库中。
+   创建一个检索器，用于根据查询查找相关文档。

```
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import WebBaseLoader
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings

urls = [
    "https://lilianweng.github.io/posts/2023-06-23-agent/",
    "https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/",
    "https://lilianweng.github.io/posts/2023-10-25-adv-attack-llm/",
]

docs = [WebBaseLoader(url).load() for url in urls]
docs_list = [item for sublist in docs for item in sublist]

text_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    chunk_size=250, chunk_overlap=0
)
doc_splits = text_splitter.split_documents(docs_list)

# Add to vectorDB
vectorstore = Chroma.from_documents(
    documents=doc_splits,
    collection_name="rag-chroma",
    embedding=OpenAIEmbeddings(),
)
retriever = vectorstore.as_retriever()
```

#### 3.3 **定义****工作流****节点**

CRAG 工作流包含以下节点，每个节点处理一个特定任务：

1.  **检索节点（retrieve）**： 从向量数据库中检索与查询相关的文档。

```
def retrieve(state):
    """
    Retrieve documents

    Args:
        state (dict): The current graph state

    Returns:
        state (dict): New key added to state, documents, that contains retrieved documents
    """
    print("---RETRIEVE---")
    question = state["question"]

    # Retrieval
    documents = retriever.get_relevant_documents(question)
    return {"documents": documents, "question": question}
```

2.  **文档评分节点（gradeDocuments）**： 使用 LLM 评估检索到的文档是否与查询相关，通常返回“相关”或“不相关”的二元评分。

```
def grade_documents(state):
    """
    Determines whether the retrieved documents are relevant to the question.

    Args:
        state (dict): The current graph state

    Returns:
        state (dict): Updates documents key with only filtered relevant documents
    """

    print("---CHECK DOCUMENT RELEVANCE TO QUESTION---")
    question = state["question"]
    documents = state["documents"]

    # Score each doc
    filtered_docs = []
    web_search = "No"
    for d in documents:
        score = retrieval_grader.invoke(
            {"question": question, "document": d.page_content}
        )
        grade = score.binary_score
        if grade == "yes":
            print("---GRADE: DOCUMENT RELEVANT---")
            filtered_docs.append(d)
        else:
            print("---GRADE: DOCUMENT NOT RELEVANT---")
            web_search = "Yes"
            continue
    return {"documents": filtered_docs, "question": question, "web_search": web_search}
```

3.  **生成节点（generate）**： 使用相关文档生成最终答案。

```
def generate(state):
    """
    Generate answer

    Args:
        state (dict): The current graph state

    Returns:
        state (dict): New key added to state, generation, that contains LLM generation
    """
    print("---GENERATE---")
    question = state["question"]
    documents = state["documents"]

    # RAG generation
    generation = rag_chain.invoke({"context": documents, "question": question})
    return {"documents": documents, "question": question, "generation": generation}
```

4.  **查询重写节点（transformQuery）**： 如果文档不相关，重写查询以提高检索效果。

```
def transform_query(state):
    """
    Transform the query to produce a better question.

    Args:
        state (dict): The current graph state

    Returns:
        state (dict): Updates question key with a re-phrased question
    """

    print("---TRANSFORM QUERY---")
    question = state["question"]
    documents = state["documents"]

    # Re-write question
    better_question = question_rewriter.invoke({"question": question})
    return {"documents": documents, "question": better_question}
```

5.  **Web 搜索节点（webSearch）**： 进行外部搜索，补充文档。

这里使用了https://tavily.com/，自行去上面注册登录后创建一个API Key，填入demo代码中的环境变量即可 **from** langchain\_community.tools.tavily\_search **import** TavilySearchResults

web\_search\_tool = TavilySearchResults(k=3)

```
def web_search(state):
    """
    Web search based on the re-phrased question.

    Args:
        state (dict): The current graph state

    Returns:
        state (dict): Updates documents key with appended web results
    """

    print("---WEB SEARCH---")
    question = state["question"]
    documents = state["documents"]

    # Web search
    docs = web_search_tool.invoke({"query": question})
    web_results = "\n".join([d["content"] for d in docs])
    web_results = Document(page_content=web_results)
    documents.append(web_results)

    return {"documents": documents, "question": question}
```

6.  决定生成

```
def decide_to_generate(state):
    """
    Determines whether to generate an answer, or re-generate a question.

    Args:
        state (dict): The current graph state

    Returns:
        str: Binary decision for next node to call
    """

    print("---ASSESS GRADED DOCUMENTS---")
    state["question"]
    web_search = state["web_search"]
    state["documents"]

    if web_search == "Yes":
        # All documents have been filtered check_relevance
        # We will re-generate a new query
        print(
            "---DECISION: ALL DOCUMENTS ARE NOT RELEVANT TO QUESTION, TRANSFORM QUERY---"
        )
        return "transform_query"
    else:
        # We have relevant documents, so generate answer
        print("---DECISION: GENERATE---")
        return "generate"
```

#### 3.4 **构建****图结构**

工作流通过节点和边连接，形成一个有向图：

+   **节点**：retrieve, gradeDocuments, generate, transformQuery, webSearch。
+   **边**：
    +   从 START 到 retrieve。
    +   从 retrieve 到 gradeDocuments。
    +   根据 gradeDocuments 的结果，条件跳转到 generate 或 transformQuery。
    +   从 transformQuery 到 webSearch，再到 generate。
    +   从 generate 到 END。

```
from langgraph.graph import END, StateGraph, START

workflow = StateGraph(GraphState)

# Define the nodes
workflow.add_node("retrieve", retrieve)  # retrieve
workflow.add_node("grade_documents", grade_documents)  # grade documents
workflow.add_node("generate", generate)  # generatae
workflow.add_node("transform_query", transform_query)  # transform_query
workflow.add_node("web_search_node", web_search)  # web search

# Build graph
workflow.add_edge(START, "retrieve")
workflow.add_edge("retrieve", "grade_documents")
workflow.add_conditional_edges(
    "grade_documents",
    decide_to_generate,
    {
        "transform_query": "transform_query",
        "generate": "generate",
    },
)
workflow.add_edge("transform_query", "web_search_node")
workflow.add_edge("web_search_node", "generate")
workflow.add_edge("generate", END)

# Compile
app = workflow.compile()
```

**解释**：

+   decide\_to\_generate 是一个条件函数，根据文档评分决定下一步。
+   如果至少有一个文档相关，直接生成答案；否则，重写查询并进行 Web 搜索。

#### 3.5 **运行****工作流**

运行 CRAG 工作流，输入一个查询并获取生成结果：

```
from pprint import pprint

# Run
inputs = {"question": "What are the types of agent memory?"}
for output in app.stream(inputs):
    for key, value in output.items():
        # Node
        pprint(f"Node '{key}':")
        # Optional: print full state at each node
        # pprint.pprint(value["keys"], indent=2, width=80, depth=None)
    pprint("\n---\n")

# Final generation
pprint(value["generation"])
```

**输出示例**：

![LangGraph之RAG范式-Corrective RAG](https://www.shxcj.com/wp-content/uploads/2025/04/ae36d8f2-89d2-47ec-86c4-b8388c1a2c2d.png)

* * *

### 4\. **CRAG 的关键特点**

+   **动态修正**：通过文档评分和补充检索（如 Web 搜索），动态修正不相关的文档。
+   **模块化设计**：LangGraph 的图结构使每个步骤（节点）易于修改或扩展。
+   **状态管理**：GraphState 确保信息在节点间无缝传递，保持工作流的一致性。

* * *

### 5\. **与传统 RAG 的对比**

<table class="has-fixed-layout"><tbody><tr><td>特性</td><td>传统 RAG</td><td>CRAG</td></tr><tr><td>文档检索</td><td>直接使用检索到的文档</td><td>评估文档相关性，必要时补充检索</td></tr><tr><td>错误处理</td><td>依赖检索质量，易受无关文档影响</td><td>动态修正无关或低质量文档</td></tr><tr><td>外部数据源</td><td>通常不涉及外部搜索</td><td>支持 Web 搜索补充信息</td></tr><tr><td>复杂性</td><td>简单，线性流程</td><td>复杂，包含条件分支和循环</td></tr></tbody></table>

### . **注意事项与优化建议**

+   **性能开销**：CRAG 比传统 RAG 更复杂，涉及多次 LLM 调用和外部 API（如 Tavily），会增加延迟。但个人感觉是必须和能接受的。
+   **文档评分准确性**：评分模型（通常是 LLM）的质量直接影响 CRAG 效果，建议使用高性能模型如 GPT-4o或更好的。
+   **Web 搜索依赖**：Tavily Search 需要 API 密钥，且搜索结果质量可能因查询优化程度而异。

RA/SD 衍生者AI训练营。发布者：稻草人，转载请注明出处：https://www.shxcj.com/archives/9570


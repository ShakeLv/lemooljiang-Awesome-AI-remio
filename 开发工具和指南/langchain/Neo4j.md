# <center> Neo4j图数据库 </center>

## 资源
[Neo4J](https://neo4j.com) 
[github](https://github.com/neo4j/neo4j)
[docker](https://hub.docker.com/_/neo4j)
[教程](https://www.w3cschool.cn/neo4j/)
[教程2](https://neo4j.ac.cn/docs/getting-started/)
[ubuntu](https://go.neo4j.com/download-thanks.html?edition=community&release=2026.05.0&flavour=deb)
[GraphRAG](https://github.com/microsoft/graphrag)
[GraphRAG文档](https://microsoft.github.io/graphrag/get_started/)


## 简介
![neo4j.jpg](https://img.ilark.io/img/202601/1781525728711-yxl52.jpg)

Neo4j 是一款主流的**图数据库（Graph Database）**，以图论为基础，用**节点（Nodes）**和**关系（Relationships）**来存储和查询数据，而非传统数据库的表结构。

`Neo4j` 是一种流行的图形数据库，用于以类似图形的结构表示数据，其中实体是节点，实体之间的关系是边。当与 `LangChain` 集成时，`Neo4j` 就变成了使用结构化图数据执行`RAG` 的强大工具。`Neo4j`的使用主要有两种方式，一种是本地安装，另一种则是可以使用云服务.


## 典型使用场景
1. **社交网络分析**
   - 好友关系、兴趣圈子、影响力传播、六度分隔查询
   
2. **知识图谱与语义网**
   - 构建领域知识库、智能问答、语义搜索、实体关系推理
   
3. **推荐系统**
   - 基于用户行为、商品属性、社交关系的多维关联推荐
   
4. **欺诈检测与风控**
   - 识别异常交易网络、信用卡套现团伙、保险欺诈环
   
5. **身份与访问管理（IAM）**
   - 用户-角色-权限-资源的复杂授权关系查询
   
6. **网络与IT运维**
   - 服务器依赖拓扑、微服务调用链分析、根因定位
   
7. **供应链管理**
   - 供应商-制造商-分销商-客户的全链路追溯
   
8. **生命科学**
   - 基因蛋白质相互作用、药物分子结构、临床路径分析


### 查询语言
Neo4j 使用 **Cypher** 作为声明式查询语言，语法直观，类似描述图模式：

```cypher
// 查找"张三"的二级好友
MATCH (a:Person {name: '张三'})-[:FRIEND]->()-[:FRIEND]->(b:Person)
RETURN b.name

//快速清空数据库：
MATCH (n)
DETACH DELETE n
```

## 写入数据
在基于 LangChain 结合 Neo4j 构建知识图谱或知识问答（GraphRAG）的场景下，文本数据的读写、节点（Node）与关系（Relationship）的处理是整个流程中最核心的步骤。
在最新的 LangChain (langchain-neo4j) 生态中，通常有两种主要范式来处理这一流程：
- 半自动/自动化：使用大模型（LLM）配合图转换器（Graph Transformers） 自动抽取。
- 纯代码/硬编码：使用 Cypher 语句精确控制 读写与图谱结构。

```py
# 借助 LLM 与图转换器（推荐用于快速构建）
# 这种方法适合非结构化文本（如 PDF、Word、长篇小说、新闻），让 LLM 自动去识别文本中的“主谓宾”，将其转化为节点和关系。

# 安装包
pip install langchain -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install langchain-openai langchain-community langchain-experimental
pip install langchain-neo4j

from langchain_core.documents import Document
from dotenv import dotenv_values
from langchain_openai import ChatOpenAI
from langchain_neo4j import Neo4jGraph
from langchain_experimental.graph_transformers import LLMGraphTransformer


env_vars = dotenv_values('.env')
OPENAI_KEY = env_vars['OPENAI_API_KEY'] 
OPENAI_BASE_URL = env_vars['OPENAI_API_BASE'] 


# 打开文件，并赋予读取模式 'r' 
def readTxt():
	with open('./ball.txt', 'r', encoding='gbk') as file:
	    content = file.read()
	    documents = [Document(page_content=content)]
	    #print(123, documents)
	    return documents

# 创建图数据库实例
graph = Neo4jGraph(url='neo4j+s://a2f9xcxx.databases.neo4j.io', 
	  username="a2f9xxxx", 
	  password="gh7EJn9Ik1xxxxx.xx",
	  database="a2f9xxxx"
)  

graph_llm = ChatOpenAI(temperature=0, model_name="gpt-5.4-mini", api_key=OPENAI_KEY, base_url=OPENAI_BASE_URL)

# 图转换器配置
graph_transformer = LLMGraphTransformer(llm=graph_llm)  #可以使用默认设置
"""
#或者是添加节点和关系边
graph_transformer = LLMGraphTransformer(
    llm=graph_llm,
    allowed_nodes=["国家", "分组", "16强赛"],    # 可以自定义节点
    allowed_relationships=["举行", "对抗", "淘汰", "战胜"],  # 可以自定义关系
)
"""

documents = readTxt()
graph_documents = graph_transformer.convert_to_graph_documents(documents)

# 写入图数据
# 该方法会自动在 Neo4j 中建立节点、属性，并连线创建关系
# graph.add_graph_documents(graph_documents)
graph.add_graph_documents(
    graph_documents, 
    baseEntityLabel=True, # 建议开启，会为所有节点额外添加一个 '__Entity__' 标签，方便后续做向量检索
    include_source=True   # 是否将原始文本 Chunk 作为一个节点连在实体上（适合回溯溯源）
)

print(f"Graph documents: {len(graph_documents)}")
print(f"Nodes from 1st graph doc:{graph_documents[0].nodes}")
print(f"Relationships from 1st graph doc:{graph_documents[0].relationships}")
```

![neo4j3.jpg](https://img.ilark.io/img/202601/1781788647769-y9zho.jpg)

![neo4j4.jpg](https://img.ilark.io/img/202601/1781788654612-tnx0v.jpg)

写入后的数据如图所示

## 批量导入结构化数据
编写明确的 Cypher 语句（推荐用于生产环境、结构化数据）。
如果你的原始文本已经是结构化的（如 CSV、JSON），或者你对节点和关系的产生有极其严格的业务逻辑要求，不希望依赖 LLMs 的随机性，则应该使用 Cypher 语句 显式地读取文本并写入。
```py
from langchain_neo4j import Neo4jGraph
import json

# 假设你的文本数据是一组结构化的实体关系描述
json_data = """
[
    {"source": "张三", "source_type": "Person", "target": "阿里巴巴", "target_type": "Organization", "relation": "就职于", "since": "2020"},
    {"source": "阿里巴巴", "source_type": "Organization", "target": "杭州", "target_type": "Location", "relation": "总部位于", "since": "1999"}
]
"""
data_list = json.loads(json_data)

graph = Neo4jGraph(url="bolt://localhost:7687", username="neo4j", password="password")

# 节点与关系的处理与写入（通过 Cypher 模版）
# 在 Neo4j 中，处理节点和关系的核心原则是 去重与合并 (MERGE)。如果使用 CREATE，每次运行都会创建重复的节点；而 MERGE 相当于 “存在则匹配，不存在则创建”。
# 我们可以利用 Neo4jGraph.query 方法，将数据作为参数传进去批量执行：
# 2.1 编写精细化的 Cypher 动态写入模板
# 注意：因为 Neo4j 标签(Label)和关系类型(Type)不能动态参数化，
# 建议在业务逻辑清晰时，针对不同的节点类型写不同的 Cypher，或者使用类似 APOC 的动态扩展。
cypher_query = """
UNWIND $data AS row
// 1. 处理源节点（去重创建）
MERGE (source:Entity {name: row.source})
ON CREATE SET source.type = row.source_type

// 2. 处理目标节点（去重创建）
MERGE (target:Entity {name: row.target})
ON CREATE SET target.type = row.target_type

// 3. 处理关系与动态属性 (这里为了兼容动态关系，使用 APOC 插件，若未装 APOC，关系名需固定)
CALL apoc.create.relationship(source, row.relation, {since: row.since}, target) YIELD rel
RETURN count(*)
"""

# 2.2 执行写入
graph.query(cypher_query, params={"data": data_list})
print("医疗图谱数据导入成功！")


# 补充下，增加数据的cypher语句
graph.query(
"""
MERGE (m:Movie {title:'Top Gun'})
WITH m
UNWIND ['Tom Cruise', 'Val Kilmer', 'Anthony Edwards', 'Meg Ryan'] AS actor
MERGE (a:Actor {name:actor})
MERGE (a)-[:ACTED_IN]->(m)
"""
)
```

Cypher语句的核心设计解析
- UNWIND 展开机制：
数据中的 symptom、check 等字段是数组（如 ["紫绀", "胸痛"]）。Cypher 无法直接把数组连向一个节点。我们使用 WITH row, d UNWIND row.symptom AS sym_name，将数组拍平循环，一条条地为每个症状创建节点并连线。

- MERGE 保证幂等性：
比如“内科”这个科室，不仅“肺泡蛋白质沉积症”有，“感冒”也有。使用 MERGE (dept:Department {name: dept_name}) 可以确保整个数据库里只有一个名叫“内科”的节点，所有相关的疾病都会连向它。

- 关系语义化：
通过 -[:HAS_SYMPTOM]->、-[:NEED_CHECK]-> 明确了图谱中边的含义，这对于后续构建 GraphCypherQAChain 时，大模型理解和生成正确的 Cypher 提问至关重要。


## 查询数据
CypherQAChain 是一个 LangChain 组件，允许您使用自然语言与 Neo4j 图数据库进行交互。它使用 LLM 和图谱模式将用户问题转换为 Cypher 查询，针对图谱执行该查询，并利用返回的上下文信息和原始问题，通过第二个 LLM 生成自然语言回复。
```py
from langchain_core.documents import Document
from dotenv import dotenv_values
from langchain_openai import ChatOpenAI
from langchain_neo4j import Neo4jGraph
from langchain_experimental.graph_transformers import LLMGraphTransformer
from langchain_neo4j import GraphCypherQAChain


env_vars = dotenv_values('.env')
OPENAI_KEY = env_vars['OPENAI_API_KEY'] 
OPENAI_BASE_URL = env_vars['OPENAI_API_BASE'] 


# 创建图数据库实例
graph = Neo4jGraph(url='neo4j+s://a2f9xcxx.databases.neo4j.io', 
	  username="a2f9xxxx", 
	  password="gh7EJn9Ik1xxxxx.xx",
	  database="a2f9xxxx"
)

graph_llm = ChatOpenAI(temperature=0, model_name="gpt-5.4-mini", api_key=OPENAI_KEY, base_url=OPENAI_BASE_URL)

cypher_chain = GraphCypherQAChain.from_llm(
    graph=graph,
    cypher_llm=graph_llm,
    qa_llm=graph_llm,
    validate_cypher=True, 
    verbose=True,
    allow_dangerous_requests=True
)

response = cypher_chain.invoke({"query": "I组有哪些队"})
# response = cypher_chain.invoke({"query": "美国在哪个小组"})
print(response["result"])
"""
> Entering new GraphCypherQAChain chain...
Generated Cypher:
MATCH (g:Group {id: 'I组'})-[:HAS_TEAM]->(c:Country)
RETURN c.id AS team
Full Context:
[{'team': '法国'}, {'team': '塞内加尔'}, {'team': '伊拉克'}, {'team': '挪威'}]

> Finished chain.
I组有法国、塞内加尔、伊拉克和挪威。
"""
```

# GraphRAG

## 资源
[Neo4J](https://neo4j.com) 
[github](https://github.com/neo4j/neo4j)
[docker](https://hub.docker.com/_/neo4j)
[教程](https://www.w3cschool.cn/neo4j/)
[教程2](https://neo4j.ac.cn/docs/getting-started/)
[ubuntu](https://go.neo4j.com/download-thanks.html?edition=community&release=2026.05.0&flavour=deb)
[GraphRAG](https://github.com/microsoft/graphrag)
[GraphRAG文档](https://microsoft.github.io/graphrag/get_started/)


## GraphRAG
GraphRAG（Graph-based Retrieval-Augmented Generation）是一种基于图结构的检索增强生成技术，其核心在于将非结构化数据转化为知识图谱，通过图遍历算法实现更精准的语义检索。相较于传统RAG模型，GraphRAG通过节点-边关系建模，可捕捉实体间的复杂关联，在医疗诊断、金融风控、法律文书分析等场景中展现出显著优势。

传统`RAG`系统在负责合成来自各种来源的信息或理解数据集中的总体主题时却很困难。例如，如果询问传统`RAG` 系统：“根据该研究数据集，全球气温变暖的主要原因是什么？”，它很难提供全面的答案，因为与气候相关的不同信息的分散在整个数据集中，而语义检索的方式没有办法跨整个数据集去做全局检索。这样的需求凸显了对 `RAG` 更加结构化和智能的方法的需求。这就是 GraphRAG（Graph Retrieval-Augmented Generation）派上用场的地方，它的核心作用是**提高大模型的模型通过利用结构化知识图谱提供精确且上下文丰富的答案的能力**。

`GraphRAG`通过利用大模型从原始文本数据中提取知识图谱来满足跨上下文检索的需求。该知识图将信息表示为互连实体和关系的网络，与简单的文本片段相比，提供了更丰富的数据表示。这种结构化表示使 `GraphRAG` 能够擅长回答需要推理和连接不同信息的复杂问题。

无需Neo4j的GraphRAG实现方案
在不依赖Neo4j的情况下，GraphRAG依然可以构建基于知识图谱的检索增强生成系统，核心是替换图数据库存储与查询层，同时保留实体/关系抽取、图构建、检索与生成的流程。
1. 存储层替代方案
    轻量内存图：使用 networkx 或 igraph 在内存中维护节点与边，适合小规模或临时计算。
    分布式图存储：如 JanusGraph（HBase/Cassandra 后端）、ArangoDB（多模型数据库，支持图查询），可替代Neo4j实现大规模图谱存储。
    三元组存储：RDF4J、Apache Jena，支持SPARQL查询，适合语义网标准场景。
2. 检索实现思路
    图遍历：用 networkx 的 shortest_path、neighbors 等方法替代 Cypher 查询，实现多跳推理。
    混合检索：结合向量数据库（如 FAISS、Milvus）进行语义召回，再用图结构过滤结果。
3. 生成阶段优化
    将检索到的子图转为结构化文本输入LLM，例如：
(感冒, 导致, 头痛)
(头痛, 缓解, 布洛芬)
    使用提示词约束生成逻辑顺序，确保因果链条清晰。
4. 适用场景与优势
    离线分析：无需部署Neo4j，降低运维成本。
    本地化部署：适合资源受限或数据敏感环境。
    可扩展性：可根据数据规模灵活切换存储后端。
注意事项
    大规模图谱需考虑索引与分片，避免遍历性能瓶颈。
    内存图方案不适合超大规模数据，需结合持久化存储。

这种方案保留了GraphRAG的核心优势（结构化知识+多跳推理），同时摆脱了对Neo4j的依赖，适合快速原型和特定业务落地。


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


## GraphRAG安装
```
1.创建虚拟环境
virtualenv rag_env
source rag_env/Scripts/activate

2.安装依赖
pip install graphrag -i https://pypi.tuna.tsinghua.edu.cn/simple
pip show graphrag

3.命令测试 graphrag --helpn
```

## 基本使用
```py
1. 初始化
graphrag init
生成.env and settings.yaml, and a directory input
2. 基本配置，.env and settings.yaml
    model_provider: openai
    model: gpt-5.4-mini
    auth_method: api_key # or azure_managed_identity
    api_key: ${GRAPHRAG_API_KEY} # set this in the generated .env file, or remove if managed identity
    api_base: https://aitool.larkneer.com/v1
3. 建立索引。把文件放入input文件夹中
graphrag index
4. 查询
graphrag query "What are the top themes in this story?"
graphrag query "文档里面涉及几家公司" 
graphrag query "美国在哪一组，有哪些对手" 
graphrag query \
"Who is Scrooge and what are his main relationships?" \
--method local
```
---
title: 图数据处理方式：NetworkX 与 Neo4j
date: 2025-12-20
categories: 图数据库
tags: ['Neo4j', 'NetworkX ']
abstract: 图数据处理方式：NetworkX 与 Neo4j 
---

# 图数据处理方式：NetworkX 与 Neo4j 

[toc]

## 1 NetworkX 是什么？  

**NetworkX** 是 Python 的图论与复杂网络分析库。

在 NetworkX 中，图是一个 Python 内存对象，用于表示节点、边及其属性。

```python
# 示例：创建节点和边
import networkx as nx

G = nx.Graph()
G.add_node(1, name="Alice") # 添加节点
G.add_edge(1, 2, weight=3)  # 添加边，带权重
```

### 1.1 特点

- 基于内存（非数据库）
- 使用 Python API 操作，灵活直观，适合算法分析与实验
- 不支持并发和持久化（程序结束即消失）

### 1.2 适用场景

- 图算法研究
- 小 / 中规模数据分析
- 数据探索与快速原型验证

## 2 Neo4j是什么

Neo4j 是一个图数据库（Graph Database），用于存储、查询和管理图结构数据。
与 NetworkX 不同，Neo4j 中的图是持久化存储在磁盘中。

Neo4j 使用**属性图模型（Property Graph Model）**，节点和关系都可以带属性。

```
# 创建节点和关系（Cypher）
CREATE (a:Person {name: "Alice"})
CREATE (b:Person {name: "Bob"})
CREATE (a)-[:FRIEND {since: 2023}]->(b)
```

### 2.1 特点

- 基于磁盘存储，支持持久化
- 支持高并发访问
- 使用 Cypher 查询语言
- 擅长多跳关系查询
- 适合大规模图数据

### 2.2 适用场景

- 知识图谱
- 社交网络
- 推荐系统
- 风控关系分析
- 复杂业务关系建模

## 3 NetworkX 与 Neo4j 的区别与联系

### 3.1 两者对比

| 问题                 | 选择     |
| -------------------- | -------- |
| 是否需要持久化       | Neo4j    |
| 是否需要并发访问     | Neo4j    |
| 是否以算法为主       | NetworkX |
| 是否是生产系统       | Neo4j    |
| 是否小规模实验       | NetworkX |
| 是否需要复杂关系查询 | Neo4j    |

### 3.2 两者联系

在实际项目中，常见做法是 **二者结合**：

```
NetworkX（建图 / 分析 / 计算）
          ↓
Neo4j（存储 / 查询 / 可视化 / 服务）
```

> NetworkX 负责“计算 + 分析”，Neo4j 负责“存储 + 查询 + 服务化”。

#### 3.2.1 使用场景分析

| 阶段              | 使用工具 | 作用                                      |
| ----------------- | -------- | ----------------------------------------- |
| 数据清洗 & 图构建 | NetworkX | 将原始数据整理为图结构，计算节点/边的属性 |
| 图算法分析        | NetworkX | 执行最短路径、中心性、社团检测等算法      |
| 数据存储 & 查询   | Neo4j    | 将处理好的图数据持久化，支持复杂关系查询  |
| 可视化 & 服务     | Neo4j    | 提供图可视化或 API 服务，方便业务使用     |

#### 3.2.2 工作流程示例

1. **数据准备**
   - 原始数据（如 CSV、JSON、数据库）
   - 在 Python 中加载并清洗
2. **建图 & 算法计算（NetworkX）**
   - 构建图：节点、边、属性
   - 计算图算法指标（如 PageRank、最短路径、社区划分）
3. **导入 Neo4j**
   - 将 NetworkX 图导出为 CSV 或通过 Neo4j Python Driver 导入
   - 在 Neo4j 中建立节点、关系和属性
4. **查询 & 可视化（Neo4j）**
   - 使用 Cypher 查询图数据
   - 可视化图结构，支持业务分析
   - 提供 API 服务给前端或应用程序

#### 3.2.3 优势

- **各取所长**：
  - NetworkX：灵活、算法丰富、快速原型
  - Neo4j：持久化、大规模、高并发、查询高效
- **减少冗余计算**：
  - 复杂算法先在 NetworkX 中算好，再存入 Neo4j，查询时直接使用结果
- **便于迭代开发**：
  - 算法调试在 NetworkX，业务查询在 Neo4j

## 4 代码实现：从 NetworkX 导入 Neo4j

目标：已有 **NetworkX 图的 JSON 文件**，将其导入 **Neo4j 图数据库**，构建包含节点与关系的知识图谱。

### 代码1：全量 NetworkX 导入

整体设计思路：采用的是**「全量 NetworkX → Neo4j」导入方案**：

1. **JSON → NetworkX**
   - 先把 JSON 中的关系数据解析成 NetworkX 图
   - 在内存中完成节点去重、属性整理
2. **NetworkX → Neo4j**
   - 先写节点
   - 再写关系
   - 写关系时通过 `MATCH` 精确匹配节点，避免笛卡尔积

```python
import json
import logging
import networkx as nx # 内存图结构
from neo4j import GraphDatabase

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Neo4j 配置
NEO4J_URI = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASSWORD = "winter123"

# JSON 文件路径
INPUT_PATH = "/data/hsw/neo4j-graphrag/military_v3_clean_new.json"

# 构建 NetworkX 图
def load_graph_from_json(input_path: str) -> nx.MultiDiGraph: # 有向，多条同类型边
    graph = nx.MultiDiGraph()
    
    with open(input_path, 'r', encoding='utf-8') as f:
        relationships = json.load(f) # “边为中心”的数据格式
    
    node_mapping = {}  # (label, name) -> node_id
    node_counter = 0
    
    for rel in relationships:
        start_node_data = rel["start_node"]
        end_node_data = rel["end_node"]
        relation = rel["relation"]
        
        # 处理 start node
        start_name = start_node_data["properties"].get("name", "")
        if isinstance(start_name, list):
            start_name = ", ".join(str(item) for item in start_name)
        elif not isinstance(start_name, str):
            start_name = str(start_name)
        
        start_key = (start_node_data["label"], start_name)
        if start_key not in node_mapping:
            node_id = f"{start_node_data['label']}_{node_counter}"
            node_mapping[start_key] = node_id
            node_counter += 1
            
            node_attrs = start_node_data["properties"].copy()
            node_attrs["label"] = start_node_data["label"]
            
            # 添加 level
            level_map = {"attribute":1, "entity":2, "keyword":3, "community":4}
            node_attrs["level"] = level_map.get(start_node_data["label"], 2)
            
            graph.add_node(node_id, **node_attrs)
        
        # 处理 end node
        end_name = end_node_data["properties"].get("name", "")
        if isinstance(end_name, list):
            end_name = ", ".join(str(item) for item in end_name)
        elif not isinstance(end_name, str):
            end_name = str(end_name)
        
        end_key = (end_node_data["label"], end_name)
        if end_key not in node_mapping:
            node_id = f"{end_node_data['label']}_{node_counter}"
            node_mapping[end_key] = node_id
            node_counter += 1
            
            node_attrs = end_node_data["properties"].copy()
            node_attrs["label"] = end_node_data["label"]
            node_attrs["level"] = level_map.get(end_node_data["label"], 2)
            
            graph.add_node(node_id, **node_attrs)
        
        # 添加边
        start_id = node_mapping[start_key]
        end_id = node_mapping[end_key]
        graph.add_edge(start_id, end_id, relation=relation)
    
    return graph

# 2️⃣ 写入 Neo4j（优化后的避免笛卡尔积）
def write_graph_to_neo4j(graph: nx.MultiDiGraph, uri, user, password):
    driver = GraphDatabase.driver(uri, auth=(user, password))
    
    def write_transaction(tx, nodes, edges):
        # 写节点
        for node_id, data in nodes:
            label = data.pop("label", "Entity")
            tx.run(
                f"MERGE (n:{label} {{id: $id}}) SET n += $props",
                id=node_id,
                props=data
            )
        
        # 写边（避免笛卡尔积）
        for u, v, data in edges:
            rel_type = data.get("relation", "RELATED_TO")
            tx.run(
                f"""
                MATCH (a {{id: $u}})
                MATCH (b {{id: $v}})
                MERGE (a)-[r:{rel_type}]->(b)
                """,
                u=u,
                v=v
            )
    
    with driver.session() as session:
        session.write_transaction(write_transaction, list(graph.nodes(data=True)), list(graph.edges(data=True)))
    
    driver.close()
    logger.info("Graph written to Neo4j successfully!")

# 3️⃣ 主流程
if __name__ == "__main__":
    G = load_graph_from_json(INPUT_PATH)
    logger.info(f"Loaded graph with {G.number_of_nodes()} nodes and {G.number_of_edges()} edges")
    write_graph_to_neo4j(G, NEO4J_URI, NEO4J_USER, NEO4J_PASSWORD)

```

特点：

- 先用 NetworkX 读 JSON 构建完整图对象。
- 然后一次性写入 Neo4j，先写节点再写边。
- 避免笛卡尔积写边（用 `MATCH (a {id:$u}) MATCH (b {id:$v}) MERGE ...`）。

优点：

- 简单直观，适合小规模图。
- 可直接利用 NetworkX 做图算法计算。

缺点：

- 全量构建图，内存占用大，JSON 很大时容易爆掉。
- 写入 Neo4j 不支持批量，效率低。
- 不适合百万级节点边的场景。

### 代码2：流式读取边

不再构建完整 NetworkX 图，而是”边读边写”直接落 Neo4j

```python
import re
import time
import ijson   # ijson 流式读取
import logging
from neo4j import GraphDatabase

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# ======================
# Neo4j 配置
# ======================
NEO4J_URI = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASSWORD = "winter123"

# JSON 文件路径
INPUT_PATH = "/data/hsw/neo4j-graphrag/military_v3_clean_new.json"

# 节点 level 映射
LEVEL_MAP = {
    "attribute": 1,
    "entity": 2,
    "keyword": 3,
    "community": 4
}

# ======================
# 工具函数
# ======================
def normalize_name(name):
    """保证 name 是字符串"""
    if isinstance(name, list):
        return ", ".join(str(x) for x in name)
    if name is None:
        return ""
    return str(name)


def normalize_rel_type(rel: str) -> str:
    """
    把任意关系字符串转换为合法 Neo4j REL TYPE
    """
    if not rel:
        return "RELATED_TO"

    # 非字母数字下划线 → _
    rel = re.sub(r"[^\w]", "_", rel)

    # 不能以数字开头
    if rel[0].isdigit():
        rel = "_" + rel

    return rel


# ======================
# 主逻辑：边读边写
# ======================
def stream_graph_to_neo4j(input_path, uri, user, password):
    driver = GraphDatabase.driver(uri, auth=(user, password))

    node_mapping = {}   # (label, name) -> node_id
    node_counter = 0
    edge_counter = 0

    with driver.session() as session, open(input_path, "r", encoding="utf-8") as f:
        objects = ijson.items(f, "item")

        for rel in objects:
            # -------- start node --------
            start_node = rel["start_node"]
            start_label = start_node["label"]
            start_name = normalize_name(start_node["properties"].get("name"))
            start_key = (start_label, start_name)

            if start_key not in node_mapping:
                start_id = f"{start_label}_{node_counter}"
                node_mapping[start_key] = start_id
                node_counter += 1

                props = dict(start_node["properties"])
                props["name"] = start_name
                props["level"] = LEVEL_MAP.get(start_label, 2)

                session.run(
                    f"MERGE (n:{start_label} {{id: $id}}) SET n += $props",
                    id=start_id,
                    props=props
                )
            else:
                start_id = node_mapping[start_key]

            # -------- end node --------
            end_node = rel["end_node"]
            end_label = end_node["label"]
            end_name = normalize_name(end_node["properties"].get("name"))
            end_key = (end_label, end_name)

            if end_key not in node_mapping:
                end_id = f"{end_label}_{node_counter}"
                node_mapping[end_key] = end_id
                node_counter += 1

                props = dict(end_node["properties"])
                props["name"] = end_name
                props["level"] = LEVEL_MAP.get(end_label, 2)

                session.run(
                    f"MERGE (n:{end_label} {{id: $id}}) SET n += $props",
                    id=end_id,
                    props=props
                )
            else:
                end_id = node_mapping[end_key]

            # -------- edge --------
            raw_rel = rel.get("relation", "RELATED_TO")
            rel_type = normalize_rel_type(raw_rel)

            session.run(
                f"""
                MATCH (a {{id: $u}})
                MATCH (b {{id: $v}})
                MERGE (a)-[:{rel_type}]->(b)
                """,
                u=start_id,
                v=end_id
            )

            edge_counter += 1

            if edge_counter % 10000 == 0:
                logger.info(
                    f"Imported {edge_counter} edges, "
                    f"{node_counter} nodes"
                )

    driver.close()
    logger.info(
        f"Finished! Total nodes: {node_counter}, edges: {edge_counter}"
    )


# ======================
# 入口
# ======================
if __name__ == "__main__":
    stream_graph_to_neo4j(
        INPUT_PATH,
        NEO4J_URI,
        NEO4J_USER,
        NEO4J_PASSWORD
    )
```

特点：

- 使用 `ijson` 边读边写，避免一次性加载大文件。
- 每读取一条边就写入 Neo4j。
- 自动处理节点唯一性，通过 `node_mapping` 保存。
- 动态生成合法的关系类型。

优点：

- 内存占用低，可处理大文件。
- 写入 Neo4j 时不会出现笛卡尔积。
- 实时进度打印。

缺点：

- 每条边都单独提交事务，事务开销大，写入速度慢。
- 边多时 Neo4j 写入吞吐低（每秒几十到几百条）。

### 代码3：批量写入优化

**流式 + 批量 UNWIND + 事务**

UNWIND 是 Neo4j 批量写入的核心

```
UNWIND $rows AS row
MERGE (n {id: row.id})
```

在 Python 侧：累积 `node_batch`、累积 `edge_batch`、达到 `BATCH_SIZE` 后，一次 UNWIND 写入 N 条数据

```
流式数据
    
Python 内存批次（1000 ~ 5000）
   ↓
一次 UNWIND 写入 Neo4j
```

**效果**

- Bolt 往返次数：`N → N / batch`
- 写入吞吐提升一个数量级
- Neo4j CPU / IO 利用率更高

```python
import ijson
import re
import time
from neo4j import GraphDatabase
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# ======================
# Neo4j 配置
# ======================
NEO4J_URI = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASSWORD = "winter123"

INPUT_PATH = "/data/hsw/neo4j-graphrag/military_v3_clean_new.json"

LEVEL_MAP = {
    "attribute": 1,
    "entity": 2,
    "keyword": 3,
    "community": 4
}

BATCH_SIZE = 1000   # 🔥 可以调：1000 / 2000 / 5000

# ======================
# 工具函数
# ======================
def normalize_name(name):
    if isinstance(name, list):
        return ", ".join(str(x) for x in name)
    return "" if name is None else str(name)


def normalize_rel_type(rel: str) -> str:
    if not rel:
        return "related_to"
    rel = re.sub(r"[^\w]", "_", rel)
    if rel[0].isdigit():
        rel = "_" + rel
    return rel


# ======================
# 批量写入函数
# ======================
def write_batch(tx, node_batch, edge_batch):
    if node_batch:
        tx.run(
            """
            UNWIND $nodes AS n
            MERGE (x:`%s` {id: n.id})
            SET x += n.props
            """ % "Entity",  # label 已经体现在 props 里
            nodes=node_batch
        )

    if edge_batch:
        tx.run(
            """
            UNWIND $edges AS e
            MATCH (a {id: e.u})
            MATCH (b {id: e.v})
            MERGE (a)-[r:`%s`]->(b)
            """ % "REL",  # 实际关系类型在下面处理
            edges=edge_batch
        )


# ======================
# 主流程
# ======================
def stream_graph_to_neo4j():
    driver = GraphDatabase.driver(NEO4J_URI, auth=(NEO4J_USER, NEO4J_PASSWORD))

    node_mapping = {}
    node_counter = 0
    edge_counter = 0

    node_batch = []
    edge_batch = []

    start_time = time.time()
    batch_start_time = start_time

    with driver.session() as session, open(INPUT_PATH, "r", encoding="utf-8") as f:
        objects = ijson.items(f, "item")

        for rel in objects:
            # ---------- start node ----------
            sn = rel["start_node"]
            sl, sn_name = sn["label"], normalize_name(sn["properties"].get("name"))
            sk = (sl, sn_name)

            if sk not in node_mapping:
                sid = f"{sl}_{node_counter}"
                node_mapping[sk] = sid
                node_counter += 1

                props = dict(sn["properties"])
                props["name"] = sn_name
                props["level"] = LEVEL_MAP.get(sl, 2)

                node_batch.append({
                    "id": sid,
                    "props": props
                })
            else:
                sid = node_mapping[sk]

            # ---------- end node ----------
            en = rel["end_node"]
            el, en_name = en["label"], normalize_name(en["properties"].get("name"))
            ek = (el, en_name)

            if ek not in node_mapping:
                eid = f"{el}_{node_counter}"
                node_mapping[ek] = eid
                node_counter += 1

                props = dict(en["properties"])
                props["name"] = en_name
                props["level"] = LEVEL_MAP.get(el, 2)

                node_batch.append({
                    "id": eid,
                    "props": props
                })
            else:
                eid = node_mapping[ek]

            # ---------- edge ----------
            edge_batch.append({
                "u": sid,
                "v": eid,
                "type": normalize_rel_type(rel.get("relation"))
            })

            edge_counter += 1

            # ---------- 批量提交 ----------
            if edge_counter % BATCH_SIZE == 0:
                now = time.time()

                with session.begin_transaction() as tx:
                    # 节点
                    if node_batch:
                        tx.run(
                            """
                            UNWIND $nodes AS n
                            MERGE (x {id: n.id})
                            SET x += n.props
                            """,
                            nodes=node_batch
                        )

                    # 边（按 type 分组，避免动态 rel type）
                    rel_groups = {}
                    for e in edge_batch:
                        rel_groups.setdefault(e["type"], []).append(e)

                    for rel_type, edges in rel_groups.items():
                        tx.run(
                            f"""
                            UNWIND $edges AS e
                            MATCH (a {{id: e.u}})
                            MATCH (b {{id: e.v}})
                            MERGE (a)-[:{rel_type}]->(b)
                            """,
                            edges=edges
                        )

                    tx.commit()

                batch_cost = now - batch_start_time
                speed = BATCH_SIZE / batch_cost if batch_cost > 0 else 0

                logger.info(
                    f"[batch] edges={edge_counter}, nodes={node_counter}, "
                    f"batch_time={batch_cost:.2f}s, "
                    f"speed={speed:.0f} edges/s, "
                    f"total_time={(now-start_time)/60:.2f} min"
                )

                node_batch.clear()
                edge_batch.clear()
                batch_start_time = now

    driver.close()
    total_time = time.time() - start_time
    logger.info(
        f"FINISHED ✅ nodes={node_counter}, edges={edge_counter}, "
        f"total_time={total_time/60:.2f} min"
    )


# ======================
# 入口
# ======================
if __name__ == "__main__":
    stream_graph_to_neo4j()

```

特点：

- 流式读取 JSON，同时收集节点和边到批量列表。
- 每 `BATCH_SIZE` 条边提交一次事务。
- 按 `type` 分组边，动态生成关系类型。
- 打印批量写入速度（edges/s）。

优点：

- 内存占用仍低，处理大文件。
- 批量事务大幅提升写入速度。
- 支持动态关系类型。

缺点：

- `UNWIND` 写节点时未按 label 分组，可能导致 Neo4j 扫描较多节点。
- 边的动态类型可能导致 Neo4j schema 不统一。

### 代码4：批量按 label/type 分组 + 约束

```python
import ijson
import re
import time
from neo4j import GraphDatabase
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# ======================
# Neo4j 配置
# ======================
NEO4J_URI = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASSWORD = "winter123"

INPUT_PATH = "/data/hsw/neo4j-graphrag/military_v3_clean_new.json"

LEVEL_MAP = {
    "attribute": 1,
    "entity": 2,
    "keyword": 3,
    "community": 4
}

BATCH_SIZE = 5000  # 可调大提高速度

# ======================
# 工具函数
# ======================
def normalize_name(name):
    if isinstance(name, list):
        return ", ".join(str(x) for x in name)
    return "" if name is None else str(name)


def normalize_rel_type(rel: str) -> str:
    if not rel:
        return "related_to"
    rel = re.sub(r"[^\w]", "_", rel)
    if rel[0].isdigit():
        rel = "_" + rel
    return rel

# ======================
# 主流程
# ======================
def stream_graph_to_neo4j():
    driver = GraphDatabase.driver(NEO4J_URI, auth=(NEO4J_USER, NEO4J_PASSWORD))

    # ⚡ 自动兼容 Neo4j 4.x / 5.x 的节点唯一约束
    with driver.session() as session:
        try:
            # Neo4j 5.x
            session.run(
                "CREATE CONSTRAINT node_id_unique IF NOT EXISTS FOR (n) ON (n.id) REQUIRE n.id IS UNIQUE;"
            )
        except Exception:
            try:
                # Neo4j 4.x
                session.run("CREATE CONSTRAINT ON (n) ASSERT n.id IS UNIQUE;")
            except Exception:
                logger.info("约束可能已存在，跳过")

    node_mapping = {}
    node_counter = 0
    edge_counter = 0

    node_batch = []
    edge_batch = []

    start_time = time.time()
    batch_start_time = start_time

    with driver.session() as session, open(INPUT_PATH, "r", encoding="utf-8") as f:
        objects = ijson.items(f, "item")

        for rel in objects:
            # ---------- start node ----------
            sn = rel["start_node"]
            sl, sn_name = sn["label"], normalize_name(sn["properties"].get("name"))
            sk = (sl, sn_name)

            if sk not in node_mapping:
                sid = f"{sl}_{node_counter}"
                node_mapping[sk] = sid
                node_counter += 1

                props = dict(sn["properties"])
                props["name"] = sn_name
                props["level"] = LEVEL_MAP.get(sl, 2)

                node_batch.append({
                    "id": sid,
                    "label": sl,
                    "props": props
                })
            else:
                sid = node_mapping[sk]

            # ---------- end node ----------
            en = rel["end_node"]
            el, en_name = en["label"], normalize_name(en["properties"].get("name"))
            ek = (el, en_name)

            if ek not in node_mapping:
                eid = f"{el}_{node_counter}"
                node_mapping[ek] = eid
                node_counter += 1

                props = dict(en["properties"])
                props["name"] = en_name
                props["level"] = LEVEL_MAP.get(el, 2)

                node_batch.append({
                    "id": eid,
                    "label": el,
                    "props": props
                })
            else:
                eid = node_mapping[ek]

            # ---------- edge ----------
            edge_batch.append({
                "u": sid,
                "v": eid,
                "type": normalize_rel_type(rel.get("relation"))
            })

            edge_counter += 1

            # ---------- 批量提交 ----------
            if edge_counter % BATCH_SIZE == 0:
                now = time.time()

                with session.begin_transaction() as tx:
                    # 写节点（按 label 批量）
                    label_groups = {}
                    for n in node_batch:
                        label_groups.setdefault(n["label"], []).append(n)
                    for label, nodes in label_groups.items():
                        tx.run(
                            f"""
                            UNWIND $nodes AS n
                            MERGE (x:`{label}` {{id: n.id}})
                            SET x += n.props
                            """,
                            nodes=nodes
                        )

                    # 写边（按 type 批量）
                    rel_groups = {}
                    for e in edge_batch:
                        rel_groups.setdefault(e["type"], []).append(e)
                    for rel_type, edges in rel_groups.items():
                        tx.run(
                            f"""
                            UNWIND $edges AS e
                            MATCH (a {{id: e.u}})
                            MATCH (b {{id: e.v}})
                            MERGE (a)-[r:`{rel_type}`]->(b)
                            """,
                            edges=edges
                        )

                    tx.commit()

                batch_cost = now - batch_start_time
                speed = BATCH_SIZE / batch_cost if batch_cost > 0 else 0

                logger.info(
                    f"[batch] edges={edge_counter}, nodes={node_counter}, "
                    f"batch_time={batch_cost:.2f}s, speed={speed:.0f} edges/s, "
                    f"total_time={(now-start_time)/60:.2f} min"
                )

                node_batch.clear()
                edge_batch.clear()
                batch_start_time = now

    # ---------- 写入剩余未满 batch 的节点和边 ----------
    if node_batch or edge_batch:
        with driver.session() as session:
            with session.begin_transaction() as tx:
                label_groups = {}
                for n in node_batch:
                    label_groups.setdefault(n["label"], []).append(n)
                for label, nodes in label_groups.items():
                    tx.run(
                        f"""
                        UNWIND $nodes AS n
                        MERGE (x:`{label}` {{id: n.id}})
                        SET x += n.props
                        """,
                        nodes=nodes
                    )

                rel_groups = {}
                for e in edge_batch:
                    rel_groups.setdefault(e["type"], []).append(e)
                for rel_type, edges in rel_groups.items():
                    tx.run(
                        f"""
                        UNWIND $edges AS e
                        MATCH (a {{id: e.u}})
                        MATCH (b {{id: e.v}})
                        MERGE (a)-[r:`{rel_type}`]->(b)
                        """,
                        edges=edges
                    )
                tx.commit()

    driver.close()
    total_time = time.time() - start_time
    logger.info(
        f"FINISHED ✅ nodes={node_counter}, edges={edge_counter}, total_time={total_time/60:.2f} min"
    )

# ======================
# 入口
# ======================
if __name__ == "__main__":
    stream_graph_to_neo4j()

# INFO:__main__:[batch] edges=5000, nodes=570, batch_time=0.11s, speed=45506 edges/s, total_time=0.00 min
# INFO:__main__:[batch] edges=10000, nodes=2835, batch_time=18.73s, speed=267 edges/s, total_time=0.31 min
# INFO:__main__:[batch] edges=15000, nodes=5135, batch_time=39.17s, speed=128 edges/s, total_time=0.97 min
# INFO:__main__:[batch] edges=20000, nodes=7372, batch_time=67.29s, speed=74 edges/s, total_time=2.09 min
# INFO:__main__:[batch] edges=25000, nodes=9515, batch_time=93.87s, speed=53 edges/s, total_time=3.65 min
# INFO:__main__:[batch] edges=30000, nodes=11787, batch_time=114.98s, speed=43 edges/s, total_time=5.57 min
```

特点：

- 批量提交节点时按 label 分组，边按 type 分组。
- 自动创建节点唯一约束，兼容 Neo4j 4.x/5.x。
- 大批量提交 (`BATCH_SIZE=5000`)。
- 记录每批次时间和速度。

优点：

- 批量写入速度显著提升。
- 避免节点扫描，提高 MERGE 性能。
- 处理大规模图（几十万边以上）可用。
- 动态兼容 Neo4j 版本约束。

缺点：

- 大批量提交时可能占用较多内存。
- 动态关系类型仍存在，Neo4j 中关系种类太多可能影响查询优化。

### 代码5：统一关系 RELATION + type 属性

```python
import ijson
import re
import time
from neo4j import GraphDatabase
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# ======================
# 配置
# ======================
NEO4J_URI = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASSWORD = "winter123"
INPUT_PATH = "/data/hsw/neo4j-graphrag/military_v3_clean_new.json"

LEVEL_MAP = {
    "attribute": 1,
    "entity": 2,
    "keyword": 3,
    "community": 4
}

BATCH_SIZE = 10000  # 初始批量，可调整

# ======================
# 工具函数
# ======================
def normalize_name(name):
    if isinstance(name, list):
        return ", ".join(str(x) for x in name)
    return "" if name is None else str(name)

def normalize_rel_type(rel: str) -> str:
    if not rel:
        return "related_to"
    rel = re.sub(r"[^\w]", "_", rel)
    if rel[0].isdigit():
        rel = "_" + rel
    return rel

# ======================
# 主流程
# ======================
def stream_graph_to_neo4j():
    driver = GraphDatabase.driver(NEO4J_URI, auth=(NEO4J_USER, NEO4J_PASSWORD))

    # ⚡ 创建节点唯一约束（兼容 4.x/5.x）
    with driver.session() as session:
        try:
            session.run(
                "CREATE CONSTRAINT node_id_unique IF NOT EXISTS FOR (n) ON (n.id) REQUIRE n.id IS UNIQUE;"
            )
        except Exception:
            try:
                session.run("CREATE CONSTRAINT ON (n) ASSERT n.id IS UNIQUE;")
            except Exception:
                logger.info("约束可能已存在，跳过")

    node_mapping = {}
    node_counter = 0
    edge_counter = 0

    node_batch = []
    edge_batch = []

    start_time = time.time()
    batch_start_time = start_time

    with driver.session() as session, open(INPUT_PATH, "r", encoding="utf-8") as f:
        objects = ijson.items(f, "item")

        for rel in objects:
            # ---------- start node ----------
            sn = rel["start_node"]
            sl, sn_name = sn["label"], normalize_name(sn["properties"].get("name"))
            sk = (sl, sn_name)

            if sk not in node_mapping:
                sid = f"{sl}_{node_counter}"
                node_mapping[sk] = sid
                node_counter += 1

                props = dict(sn["properties"])
                props["name"] = sn_name
                props["level"] = LEVEL_MAP.get(sl, 2)

                node_batch.append({
                    "id": sid,
                    "label": sl,
                    "props": props
                })
            else:
                sid = node_mapping[sk]

            # ---------- end node ----------
            en = rel["end_node"]
            el, en_name = en["label"], normalize_name(en["properties"].get("name"))
            ek = (el, en_name)

            if ek not in node_mapping:
                eid = f"{el}_{node_counter}"
                node_mapping[ek] = eid
                node_counter += 1

                props = dict(en["properties"])
                props["name"] = en_name
                props["level"] = LEVEL_MAP.get(el, 2)

                node_batch.append({
                    "id": eid,
                    "label": el,
                    "props": props
                })
            else:
                eid = node_mapping[ek]

            # ---------- edge ----------
            edge_batch.append({
                "u": sid,
                "v": eid,
                "type": normalize_rel_type(rel.get("relation"))
            })

            edge_counter += 1

            # ---------- 批量提交 ----------
            if edge_counter % BATCH_SIZE == 0:
                now = time.time()
                with session.begin_transaction() as tx:
                    # 写节点
                    if node_batch:
                        label_groups = {}
                        for n in node_batch:
                            label_groups.setdefault(n["label"], []).append(n)
                        for label, nodes in label_groups.items():
                            tx.run(
                                f"""
                                UNWIND $nodes AS n
                                MERGE (x:`{label}` {{id: n.id}})
                                SET x += n.props
                                """,
                                nodes=nodes
                            )

                    # 写边（统一关系 RELATION，type 属性保存原类型）
                    if edge_batch:
                        tx.run(
                            """
                            UNWIND $edges AS e
                            MATCH (a {id: e.u})
                            MATCH (b {id: e.v})
                            MERGE (a)-[r:RELATION]->(b)
                            SET r.type = e.type
                            """,
                            edges=edge_batch
                        )
                    tx.commit()

                batch_cost = now - batch_start_time
                speed = BATCH_SIZE / batch_cost if batch_cost > 0 else 0

                logger.info(
                    f"[batch] edges={edge_counter}, nodes={node_counter}, "
                    f"batch_time={batch_cost:.2f}s, speed={speed:.0f} edges/s, "
                    f"total_time={(now-start_time)/60:.2f} min"
                )

                node_batch.clear()
                edge_batch.clear()
                batch_start_time = now

    # ---------- 写入剩余未满批次 ----------
    if node_batch or edge_batch:
        with driver.session() as session:
            with session.begin_transaction() as tx:
                if node_batch:
                    label_groups = {}
                    for n in node_batch:
                        label_groups.setdefault(n["label"], []).append(n)
                    for label, nodes in label_groups.items():
                        tx.run(
                            f"""
                            UNWIND $nodes AS n
                            MERGE (x:`{label}` {{id: n.id}})
                            SET x += n.props
                            """,
                            nodes=nodes
                        )
                if edge_batch:
                    tx.run(
                        """
                        UNWIND $edges AS e
                        MATCH (a {id: e.u})
                        MATCH (b {id: e.v})
                        MERGE (a)-[r:RELATION]->(b)
                        SET r.type = e.type
                        """,
                        edges=edge_batch
                    )
                tx.commit()

    driver.close()
    total_time = time.time() - start_time
    logger.info(
        f"FINISHED ✅ nodes={node_counter}, edges={edge_counter}, total_time={total_time/60:.2f} min"
    )

# ======================
# 入口
# ======================
if __name__ == "__main__":
    stream_graph_to_neo4j()

```

特点：

- 批量写入节点按 label 分组。
- 所有边写入统一关系 `RELATION`，原始关系类型存入属性 `type`。
- 保留批量提交逻辑和进度统计。
- 适合关系类型过多或动态生成的场景。

优点：

- Neo4j schema 简化，关系类型统一，查询更稳定。
- 批量提交，写入效率高。
- 可处理超大图。
- 方便后续根据 `type` 属性查询或分析。

缺点：

- 查询时需要使用 `r.type` 而非直接关系类型。
- 如果关系类型本身很重要，查询写法略有变化。

### 代码总结

| 版本      | 适用场景                | 优点                                        | 缺点                                  |
| --------- | ----------------------- | ------------------------------------------- | ------------------------------------- |
| **代码1** | 小图、算法计算          | 简单，直观                                  | 内存大，写入慢，不适合大图            |
| **代码2** | 大图、流式导入          | 内存低，实时                                | 每条边单独事务，慢                    |
| **代码3** | 大图、批量导入          | 批量提高速度                                | 节点未按 label 分组，关系类型多       |
| **代码4** | 大图、批量 + schema优化 | 节点按 label 分组，关系按 type 分组，速度快 | 动态关系多，查询优化略复杂            |
| **代码5** | 超大图 + 简化 schema    | 关系统一，便于查询和分析                    | 查询需用属性 `type`，不直观展示原类型 |

- 如果图规模较小 (<10万节点/边)：**代码1 或 2** 足够。
- 中大型图 (10万~100万边)：**代码3 或 4**，按 label/type 批量写入。
- 超大图 (>100万边) 或关系类型太多：**代码5** 最稳妥，统一关系类型、批量提交。

------

💡 **优化思路**：

1. **增量导入**：每次只处理新边，避免重复导入。
2. **节点唯一约束**：保证 `MERGE` 快速。
3. **边分组写入**：减少 Neo4j 动态关系创建，提升性能。
4. **批量大小调节**：根据机器性能调整 `BATCH_SIZE`。
5. **日志与进度**：打印速度、累计节点边数量，便于监控。
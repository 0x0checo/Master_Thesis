# Sustainability Knowledge Base 研究项目指导手册

## 📋 目录
1. [项目概述](#项目概述)
2. [四个研究问题详解](#四个研究问题详解)
3. [知识库核心概念](#知识库核心概念)
4. [三种Representation对比](#三种representation对比)
5. [API与接口设计](#api与接口设计)
6. [评估指标体系](#评估指标体系)
7. [实施路径与时间规划](#实施路径与时间规划)

---

## 项目概述

### 核心目标
构建一个 **Sustainability Knowledge Base**，用于支持 AI-assisted discussion moderation（讨论调节机器人）。

### 研究框架
```
PDF/文献 → LLM提取 → 结构化表示 → 统一API → 下游RAG应用
                         ↓
              对比不同representation的效果
```

### 四个研究问题
| RQ | 核心问题 | 重点 |
|----|----------|------|
| RQ1 | 如何提取和转换sustainability principles? | 数据准备（非核心） |
| RQ2 | 对比不同representation formats | **核心贡献** |
| RQ3 | 哪种最适合discussion moderation场景? | **核心贡献** |
| RQ4 | 评估标准是什么? | 方法论 |

---

## 四个研究问题详解

### RQ1: 数据提取与转换

> **建议**: 不需要对比多种extraction方法，用LLM extraction即可

#### 修订后的RQ1表述
~~"How can sustainability principles be reliably extracted..."~~

→ "How can sustainability principles from research literature be **transformed into structured representations** suitable for reasoning?"

#### 具体步骤
1. **用LLM提取信息**（单一方法）
   - 使用GPT/Claude + prompt engineering
   - 输出结构化JSON

2. **展示transformation pipeline**
   ```
   PDF/Text → LLM extraction → Raw JSON 
                                  ├─> Schema-based (JSON/SQL)
                                  ├─> Knowledge Graph (Neo4j)  
                                  └─> Ontology (可选)
   ```

3. **验证extraction quality**
   - 人工检查sample（50-100个principles）
   - 报告 Precision, Recall（证明数据质量可靠）

#### 论文篇幅分配
- Extraction: 1章（Methodology中的data preparation）
- Representation comparison: 2-3章（**核心！**）

---

### RQ2: Representation对比（核心）

#### 推荐对比的格式

| Representation | 工具 | 优势 | 劣势 |
|----------------|------|------|------|
| **JSON** | 文本文件 | 简单，容易理解 | 查询慢，难推理 |
| **SQL Database** | SQLite | 查询快，结构清晰 | 关系表示不灵活 |
| **Knowledge Graph** | Neo4j | 复杂推理，关系表达强 | 学习曲线陡 |

#### 推荐组合
- **Option A（最简单）**: JSON vs Knowledge Graph
- **Option B（推荐）**: SQL vs Knowledge Graph
- **Option C（完整）**: JSON + SQL + Knowledge Graph

#### 三种格式的具体表示

**Schema-based (JSON)**
```json
{
  "principle_id": "P001",
  "principle_text": "Avoid systematic increases of...",
  "domain": "energy",
  "topic": "wind_energy",
  "phase": "production",
  "risks": ["bird_mortality", "habitat_disruption"],
  "mitigations": ["site_selection", "turbine_design"],
  "sources": [{"doc": "Smith2023.pdf", "page": 5}]
}
```

**Knowledge Graph (RDF triplets)**
```
(WindEnergy)--[IN_DOMAIN]-->(Energy)
(WindEnergy)--[HAS_RISK]-->(BirdMortality)
(BirdMortality)--[MITIGATED_BY]-->(SiteSelection)
(P001)--[CITED_FROM]-->(Source:Smith2023)
```

---

### RQ3: Moderation场景评估（核心）

#### 模拟场景
假设讨论中提到: *"we're using wind energy for our factory"*

Moderator需要查询:
- 有哪些sustainability风险?
- 需要问什么问题?
- 能否解释reasoning过程?

#### 测试维度
1. **Query速度** - 是否满足real-time需求?
2. **生成质量** - 能否产生有意义的intervention questions?
3. **可解释性** - 能否解释reasoning过程?

#### User Study（如果时间允许）
- 给几个sustainability专家看不同representation生成的questions
- 让他们评价质量

---

### RQ4: 评估方法论

在做RQ1-3的过程中回答，详见[评估指标体系](#评估指标体系)

---

## 知识库核心概念

### 什么是知识库？

```
知识库 = 数据内容 + 存储格式 + 查询接口
```

#### 类比理解
- 📚 **图书馆** = 知识库
- 📖 **书** = 每条sustainability principle
- 🗂️ **分类系统** = 不同的representation方法

### 知识库的"物理形态"

| Representation | 知识库是什么 | 是JSON文件吗? | 给下游的格式 |
|----------------|-------------|--------------|-------------|
| 纯JSON | `kb.json` | ✅ 是 | 同一个文件 |
| SQL Database | `kb.db` (SQLite) | ❌ 不是 | export.json |
| Knowledge Graph | `neo4j_data/` (文件夹) | ❌ 不是 | export.json |

### 统一知识库设计（推荐）

**不要**: 为每个domain创建独立的knowledge base  
**应该**: 一个knowledge base，用metadata区分domains

```json
{
  "principle_id": "P001",
  "principle_text": "...",
  "fssd_principle": "principle_1",
  "domain": "energy",           // energy / transport / chemicals
  "topic": "wind_energy",
  "subtopic": "offshore_wind",
  "lifecycle_phase": "production",  // supply / production / use
  "risks": [...],
  "mitigations": [...],
  "sources": [...]
}
```

#### 优势
1. **Cross-domain reasoning**: 可跨领域推理（如electric transport涉及energy + transport）
2. **共享principles**: 8个FSSD原则是通用的
3. **Easier maintenance**: 维护一个系统
4. **Better comparison**: RQ2对比时用统一数据更公平

---

## 三种Representation对比

### Format A: 纯JSON（最简单）

```python
# knowledge_base_json.py
class JSONKnowledgeBase:
    def __init__(self, file_path):
        with open(file_path) as f:
            self.data = json.load(f)
    
    def query(self, domain=None, topic=None):
        results = self.data
        if domain:
            results = [p for p in results if p['domain'] == domain]
        if topic:
            results = [p for p in results if p['topic'] == topic]
        return results
```

**优点**: 超级简单  
**缺点**: 查询慢（数据多时）、不能做复杂推理

---

### Format B: SQL Database

```sql
CREATE TABLE principles (
    id TEXT PRIMARY KEY,
    text TEXT,
    domain TEXT,
    topic TEXT,
    phase TEXT
);

CREATE TABLE risks (
    principle_id TEXT,
    risk TEXT,
    FOREIGN KEY (principle_id) REFERENCES principles(id)
);

-- 查询示例
SELECT risk FROM principles 
JOIN risks ON principles.id = risks.principle_id
WHERE domain = 'energy';
```

**优点**: 查询快、结构清晰  
**缺点**: 关系表示不够灵活

---

### Format C: Knowledge Graph (Neo4j)

```cypher
// 创建节点和关系
CREATE (p:Principle {id: 'P001', text: '...'})
CREATE (d:Domain {name: 'energy'})
CREATE (t:Topic {name: 'wind_energy'})
CREATE (r:Risk {name: 'bird_mortality'})

CREATE (p)-[:IN_DOMAIN]->(d)
CREATE (p)-[:ABOUT]->(t)
CREATE (p)-[:HAS_RISK]->(r)

// 查询示例
MATCH (d:Domain {name:'energy'})<-[:IN_DOMAIN]-(p:Principle)-[:HAS_RISK]->(r:Risk)
RETURN p.text, r.name
```

**优点**: 擅长复杂关系、推理能力强  
**缺点**: 学习曲线陡

---

## API与接口设计

### 为什么需要API?

```
您的工作: Knowledge Base → 通过API → 下游同事的RAG Bot
```

类比：API就像"电话系统"，规定"怎么问"和"怎么答"

### 三种方案对比

| 方案 | 难度 | 时间 | 适用阶段 |
|------|------|------|----------|
| **方案1: JSON文件** | ⭐ | 1天 | Week 1-6 |
| **方案2: Python模块** | ⭐⭐ | 1-2天 | Week 5-8 |
| **方案3: REST API** | ⭐⭐⭐ | 3-7天 | Week 12+ |

---

### 方案1: JSON文件交换（推荐先用）

```python
# export_kb.py
def export_knowledge_base():
    # 从数据库导出
    data = export_from_sql()  # 或 export_from_neo4j()
    
    with open("knowledge_base.json", "w") as f:
        json.dump(data, f, indent=2)

# 下游同事使用
with open("knowledge_base.json") as f:
    kb = json.load(f)
    results = [p for p in kb if p["topic"] == "wind_energy"]
```

---

### 方案2: Python模块接口

```python
# knowledge_base.py
class KnowledgeBase:
    def __init__(self, representation_type="schema"):
        self.type = representation_type
        # 初始化连接...
    
    def query(self, topic=None, domain=None, phase=None):
        """统一查询接口"""
        if self.type == "schema":
            return self._query_schema(topic, domain, phase)
        elif self.type == "graph":
            return self._query_graph(topic, domain, phase)
    
    def get_risks(self, topic):
        """便捷方法"""
        results = self.query(topic=topic)
        return list(set(r for p in results for r in p["risks"]))

# 下游同事使用
from knowledge_base import KnowledgeBase
kb = KnowledgeBase(representation_type="schema")
risks = kb.get_risks("wind_energy")
```

---

### 方案3: REST API

```python
# api_server.py
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/query', methods=['GET'])
def query():
    topic = request.args.get('topic')
    domain = request.args.get('domain')
    representation = request.args.get('representation', 'schema')
    
    kb = kb_schema if representation == "schema" else kb_graph
    results = kb.query(topic=topic, domain=domain)
    
    return jsonify({"results": results, "count": len(results)})

# 运行: python api_server.py
# 访问: GET /api/query?topic=wind_energy&representation=schema
```

---

### 部署选项

| 选项 | 费用 | 适用场景 |
|------|------|----------|
| **本地电脑** | 免费 | 开发测试、同办公室协作 |
| **公司Server** | 免费 | 需IT协助部署 |
| **Cloud (Render)** | 免费tier | 论文demo、remote访问 |

**建议路径**:
- Week 1-8: 本地开发
- Week 12+: 如需demo，部署到Render（1天，免费）

---

## 评估指标体系

### 1. Traceability（可追溯性）

> 能否从结构化的fact追溯回原始文献?

| 指标 | 计算方法 | 目标 |
|------|----------|------|
| Source Coverage Rate | 有source的facts / 总facts | 100% |
| Source Retrieval Accuracy | 正确的references / 抽样数 | >90% |
| Tracing Granularity | Level 1-3 | Level 3 |

**Granularity Levels**:
- Level 1: 只知道来自哪篇paper
- Level 2: 知道page number
- Level 3: 知道exact sentence/paragraph

**实现要求**:
```json
{
  "source_document": "paper_id",
  "source_location": "page 5, section 2.3",
  "original_text": "exact quote from source"
}
```

---

### 2. Reasoning Capability（推理能力）

> 能否基于facts进行逻辑推理?

#### Reasoning Tasks

| Task Type | 示例 | 难度 |
|-----------|------|------|
| Simple Lookup | "What are the risks of wind energy?" | ⭐ |
| Multi-hop | "If we use wind energy, what principles might be violated?" | ⭐⭐ |
| Contradiction Detection | 检测矛盾的facts | ⭐⭐⭐ |
| Explanation Generation | "Why is chemical X problematic?" | ⭐⭐⭐ |

#### 评估指标

| 指标 | 说明 |
|------|------|
| Reasoning Coverage | 支持多少种task types |
| Inference Correctness | 正确answers / 总queries |
| Reasoning Depth | 平均reasoning steps |
| Execution Time | 每个query耗时(ms) |

---

### 3. Ease of Maintenance（易维护性）

> Subject matter experts能否轻松维护?

| 指标 | 方法 | 
|------|------|
| Human Readability Score | 专家打分 1-5 |
| Edit Complexity | 修改需要的步骤数 |
| Error Rate | 用户任务中的错误次数 |
| Task Completion Time | 完成CRUD操作的时间 |

#### User Study设计
1. 招募3-5个sustainability专家
2. 让他们完成以下任务:
   - Task 1: "Find all facts about wind energy"
   - Task 2: "Correct this wrong fact"
   - Task 3: "Add a new mitigation strategy"
3. 记录: 完成时间、错误次数、主观满意度

---

## 实施路径与时间规划

### 推荐时间表

| 阶段 | 周数 | 任务 |
|------|------|------|
| **Phase 1** | Week 1-2 | 用JSON实现第一个representation |
| **Phase 2** | Week 3-4 | 学习并实现Knowledge Graph |
| **Phase 3** | Week 5-6 | 对比评估两种representation |
| **Phase 4** | Week 7-8 | 升级到Python模块接口 |
| **Phase 5** | Week 9-10 | Moderation场景测试 |
| **Phase 6** | Week 11-12 | User Study（如时间允许） |
| **Phase 7** | Week 12+ | 部署到Cloud（可选） |

### 项目文件结构

```
📁 your_project/
├── 📄 data/
│   ├── raw_papers/              # 原始PDF文件
│   └── extracted/               # LLM提取的原始数据
│       └── extracted_facts.json
│
├── 📊 representations/          # 不同格式的知识库
│   ├── json_based/
│   │   └── kb.json              # JSON知识库
│   ├── schema_based/
│   │   ├── kb.db                # SQLite数据库
│   │   └── export.json
│   └── knowledge_graph/
│       ├── neo4j_data/          # Neo4j数据
│       └── export.json
│
├── 📝 code/
│   ├── extraction.py            # LLM提取pipeline
│   ├── build_json_kb.py         # 构建JSON KB
│   ├── build_schema_kb.py       # 构建SQL KB
│   ├── build_graph_kb.py        # 构建Knowledge Graph
│   ├── knowledge_base.py        # 统一查询接口
│   ├── evaluation.py            # 评估脚本
│   └── api_server.py            # API服务
│
├── 📋 evaluation/
│   ├── test_queries.json        # 测试queries
│   ├── gold_standard.json       # Ground truth
│   └── results/                 # 评估结果
│
└── 📄 README.md
```

---

## 与下游同事协调Checklist

### Week 1-2: 确定Interface

发送邮件确认:
1. 需要什么格式的输出? JSON够吗?
2. 需要查询哪些信息? (By topic/domain/risk?)
3. 需要什么fields? (Risks/Mitigations/Sources?)
4. 响应时间要求?
5. 先给JSON files可以吗?

### Week 3: 提供第一版数据

交付物:
- `knowledge_base_schema.json`
- `knowledge_base_graph.json`
- `README.md`

### Week 8: 获取feedback

收集下游同事的实际queries作为evaluation benchmarks

---

## 论文写作建议

### 关于Extraction (RQ1)
> "We employ LLM-based extraction using GPT-4/Claude with carefully designed prompts. Manual verification of a sample of 100 principles showed precision of X% and recall of Y%, demonstrating sufficient data quality for our comparison study."

### 关于Representation (RQ2)
> "We compare two representation approaches: (1) Schema-based representation using structured JSON/SQL, and (2) Graph-based representation using Neo4j. These represent two fundamentally different paradigms for knowledge organization..."

### 关于API (RQ3)
> "We provide a unified JSON API that abstracts the underlying representation format, enabling seamless integration with downstream RAG systems regardless of whether schema-based or graph-based representations are used internally."

### 关于Deployment
> "The system is designed to be deployment-ready and can be easily deployed to cloud platforms. For demonstration purposes, we deployed the API to Render, enabling real-time access for evaluation."

---

## 核心要点总结

1. **RQ1不是核心**: 用LLM extraction即可，重点在transformation
2. **RQ2-3是核心贡献**: 对比不同representation在moderation场景的表现
3. **统一知识库设计**: 一个KB，用metadata区分domains
4. **API从简单开始**: JSON文件 → Python模块 → REST API
5. **评估三个维度**: Traceability, Reasoning, Maintenance
6. **费用可以为0**: 本地开发 + Render免费tier

---

*文档整理日期: 2026-02-05*

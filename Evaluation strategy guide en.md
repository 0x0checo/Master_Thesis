# Evaluation Strategy Guide: How to Evaluate Without a Complete Bot

## 🎯 First Clarification: What are the 5 Evaluation Objects?

### RQ2's 3 Objects:
1. **Traceability**
2. **Reasoning capability**
3. **Ease of maintenance**

### RQ3's 2 Objects:
4. **Topic-aware intervention**
5. **Explanation**

**You're right! There are 5 evaluation dimensions in total.**

---

## 🤔 Core Question: How to Evaluate Without a Bot?

### Key Insight:
**You don't need a complete bot!** What you need is:
- **Simulate the bot's requirement scenarios**
- **Create benchmark queries**
- **Test how different representations respond to these queries**

---

## 💡 Solution: Layered Evaluation Strategy

### **Layer 1: Intrinsic Evaluation** (No bot needed)
Evaluate the properties of the representation itself

### **Layer 2: Task-based Evaluation** (Simulate bot requirements)
Simulate questions the bot would ask, see which representation performs better

### **Layer 3: Extrinsic Evaluation** (Requires downstream assistance)
Actually integrate with bot, observe real-world effects **(Optional, if time permits)**

---

## 📊 Detailed Evaluation Plan (No Bot Required)

### **RQ2: Evaluation of Three Objects**

#### **1. Traceability (Completely independent, no bot needed)**

**Evaluation Method**:

**Test 1: Source Linking Test**
```
Given: An extracted principle
Task: Can you find the accurate location in the original literature?
```

**Specific Operations**:
1. Randomly select 20 principles from your knowledge base
2. Have 2 people (can be yourself + classmate) try to verify the source
3. Record:
   - Can the source be found? (Yes/No)
   - How long does it take to find? (seconds)
   - Precision of source? (document level / page level / sentence level)

**Comparison**:
```python
# Schema-based representation
{
  "principle": "...",
  "source": {
    "document": "Smith2023.pdf",
    "page": 5,
    "section": "2.3",
    "quote": "exact text..."
  }
}
# Evaluate: Can you click a link to jump directly to the corresponding PDF location?

# Knowledge Graph representation
(Principle:P001)-[:CITED_FROM {page: 5}]->(Document:Smith2023)
# Evaluate: Can you trace back to source through graph traversal?
```

**Metrics**:
- ✅ Source completeness: What % of facts have sources?
- ✅ Source accuracy: Is the source reference accurate?
- ✅ Retrieval time: How long to find the source?

**No bot or downstream colleagues needed!**

---

#### **2. Reasoning Capability (No complete bot needed)**

**Key**: Create a set of **benchmark reasoning queries**

**Evaluation Method**:

**Step 1: Define reasoning task types**

Based on sustainability domain, define reasoning the bot might need:

```
Type 1: Simple Lookup
Q: "What are the risks of wind energy?"
Expected: Direct retrieval

Type 2: Aggregation
Q: "What are all the risks in the energy domain?"
Expected: Aggregate multiple facts

Type 3: Multi-hop Reasoning
Q: "If we use wind energy in production phase, 
    which FSSD principles might be violated?"
Expected: WindEnergy → Production → Principles → Violations

Type 4: Comparison
Q: "Compare sustainability risks of wind vs solar energy"
Expected: Retrieve two topics and compare

Type 5: Causal Reasoning
Q: "Why is chemical X problematic for biodiversity?"
Expected: X → causes Y → violates biodiversity principle

Type 6: Mitigation Retrieval
Q: "What can mitigate bird mortality from wind turbines?"
Expected: Find mitigations corresponding to the risk
```

**Step 2: Implement these queries**

For each representation, implement the same queries:

**Schema-based (SQL)**:
```sql
-- Type 1: Simple Lookup
SELECT risks FROM principles WHERE topic='wind_energy';

-- Type 3: Multi-hop
SELECT p.fssd_principle FROM principles p
JOIN topics t ON p.topic_id = t.id
WHERE t.name='wind_energy' AND p.phase='production';
```

**Knowledge Graph (Cypher)**:
```cypher
// Type 1
MATCH (t:Topic {name:'wind_energy'})-[:HAS_RISK]->(r:Risk)
RETURN r.name

// Type 3  
MATCH (t:Topic {name:'wind_energy'})
      -[:IN_PHASE]->(ph:Phase {name:'production'})
      -[:VIOLATES]->(p:FssdPrinciple)
RETURN p.name
```

**Step 3: Comparative evaluation**

Create 10-15 such queries, 2-3 for each reasoning type:

| Query ID | Type | Schema-based | Knowledge Graph |
|----------|------|--------------|-----------------|
| Q1 | Lookup | ✅ Supported | ✅ Supported |
| Q2 | Aggregation | ✅ Supported | ✅ Supported |
| Q3 | Multi-hop | ⚠️ Complex SQL | ✅ Easy (graph traversal) |
| Q4 | Comparison | ✅ Supported | ✅ Supported |
| Q5 | Causal | ❌ Need custom code | ✅ Native support |

**Metrics**:
- ✅ **Task coverage**: How many reasoning task types can be supported?
- ✅ **Query complexity**: How complex is the implementation? (lines of code, query complexity)
- ✅ **Correctness**: Are results correct? (compare against gold standard)
- ✅ **Execution time**: How fast is each query?

**No complete bot needed! Just define queries and implement them.**

---

#### **3. Ease of Maintenance (No bot needed, but may need a few volunteers)**

**Evaluation Method**:

**Test 1: Readability Test**
1. Show your representation to 3-5 people (classmates/sustainability experts)
2. Ask: "Can you understand what this fact means?"
3. Rate on 5-point scale

**Test 2: Edit Task**
1. Create 3 tasks:
   - Task A: "Find the fact about wind energy bird mortality"
   - Task B: "Correct this wrong mitigation strategy"
   - Task C: "Add a new risk to solar energy"

2. Have participants complete tasks using the interface you provide

3. Record:
   - Task completion time
   - Number of errors
   - Subjective difficulty (1-5 scale)

**Test 3: Schema Evolution**
1. Simulate requirement change: "We need to add a new field 'geographical_context'"
2. Evaluate how much modification each representation requires:
   - Schema-based: Need to alter table, update all records
   - Knowledge Graph: Just add new edge type

**Metrics**:
- ✅ **Readability score** (1-5)
- ✅ **Edit complexity** (steps required)
- ✅ **Task completion time**
- ✅ **Schema flexibility** (how easy to extend)

**Needs a small number of volunteers (3-5), but no bot or downstream colleagues.**

---

### **RQ3: Evaluation of Two Objects**

#### **4. Topic-aware Intervention (Requires simulation, no real bot needed)**

**Key**: Create **simulated discussion scenarios**

**Evaluation Method**:

**Step 1: Create discussion transcripts**

Write several simulated meeting dialogues (5-10 scenarios):

```
Scenario 1: Wind Energy Discussion
---
Speaker A: "We're planning to use wind energy for our new factory."
Speaker B: "That sounds sustainable. Any concerns?"
Speaker A: "I think it's completely clean energy."

→ Bot should intervene: Remind about bird mortality and habitat risks
```

```
Scenario 2: Electric Transport Discussion
---
Speaker A: "Let's switch to electric vehicles for our fleet."
Speaker B: "The battery production uses lithium mining."
Speaker A: "Is that a problem?"

→ Bot should intervene: Explain sustainability risks of lithium mining
```

**Step 2: Test whether each representation can support intervention**

For each scenario, test:

**Manual Query Simulation**:
```python
# Extract keywords from discussion
keywords = extract_keywords("wind energy", "factory")

# Query knowledge base
schema_results = query_schema_db(keywords)
graph_results = query_knowledge_graph(keywords)

# Evaluate: Can it return relevant facts?
```

**Evaluation Dimensions**:

1. **Relevance**: Are returned facts relevant to the discussion?
   - Manual judgment: 3 raters score each result (1-5)
   
2. **Completeness**: Does it include all risks that should be mentioned?
   - Compare against predefined "expected facts"
   
3. **Response Time**: Can it return in real-time? (< 2 seconds)

4. **Context Awareness**: Can it understand the discussion context?
   - Example: "factory" → should focus on "production phase"

**Comparison Table**:

| Scenario | Schema-based | Knowledge Graph |
|----------|--------------|-----------------|
| Relevance | 3.5/5 | 4.2/5 |
| Completeness | 60% | 80% |
| Response Time | 0.5s | 1.2s |
| Context Aware | Partial | Better |

**No real bot needed! Manually simulate the bot's queries.**

---

#### **5. Explanation (No bot needed)**

**Evaluation Method**:

**Test 1: Explanation Generation**

Given an intervention, test whether explanation can be generated:

**Scenario**:
```
Bot says: "Wind energy may have biodiversity impacts."
User asks: "Why? Can you explain?"
```

**Test whether each representation can support explanation**:

**Schema-based**:
```python
def generate_explanation(topic="wind_energy", risk="biodiversity"):
    # Query database
    result = db.query(
        "SELECT principle_text, risk_description, source "
        "FROM principles "
        "WHERE topic=? AND risk_type=?",
        (topic, risk)
    )
    
    # Generate explanation
    explanation = f"According to {result.source}, {result.risk_description}"
    return explanation
```

**Knowledge Graph**:
```python
def generate_explanation(topic="wind_energy", risk="biodiversity"):
    # Graph traversal
    path = graph.query("""
        MATCH path = (t:Topic {name:'wind_energy'})
                     -[*]->(r:Risk {type:'biodiversity'})
        RETURN path
    """)
    
    # Generate explanation from path
    explanation = explain_reasoning_path(path)
    return explanation
```

**Evaluation Dimensions**:

1. **Traceability of Explanation**: Can the reasoning chain be traced?
   - Knowledge Graph: ✅ Can show path
   - Schema-based: ⚠️ Requires additional logic

2. **Explanation Completeness**: Does it include all relevant information?
   - Source, risk, mitigation, reasoning

3. **Human Understandability**: Can people understand this explanation?
   - Have 3-5 people read the explanation and rate it (1-5)

**Metrics**:
- ✅ **Reasoning chain visibility** (Can it be visualized?)
- ✅ **Explanation completeness** (%)
- ✅ **Understandability score** (1-5)

**No bot needed! Manually generate explanations and evaluate.**

---

## 🤝 What Assistance is Needed from Downstream Colleagues?

### **Required Assistance (Minimum)**:

1. **Interface Specification** (Complete in Week 1-2):
   ```
   Need to clarify:
   - What output format does downstream RAG need? (JSON schema)
   - What API endpoints are needed?
   - What are typical query patterns?
   ```

2. **Provide Example Queries** (Week 3-4):
   ```
   Ask RAG colleagues to provide 5-10 queries they would send:
   - "Find risks of wind energy in production"
   - "What mitigations exist for biodiversity loss?"
   - etc.
   
   Use these as your benchmark queries
   ```

3. **Mock Integration Test** (Week 12-15, optional):
   ```
   If time permits, do a simple integration:
   - You provide API
   - They call API to get facts
   - Test if it works
   - Record performance metrics
   ```

### **Nice to Have (Optional)**:

4. **User Study Participation** (if doing user study):
   ```
   Downstream colleagues participate as domain experts:
   - Readability test
   - Query quality evaluation
   - Provide feedback
   ```

5. **End-to-end Demo** (final weeks, if time permits):
   ```
   Actually integrate with their bot for a demo:
   - Prove your knowledge base is useful
   - Can write in thesis "deployed in a real system"
   - More convincing
   ```

---

## 📝 Suggested Evaluation Timeline

### **Parts Independent of Downstream Colleagues** (Can be completed independently):

**Week 1-4**: 
- ✅ Implement extraction pipeline
- ✅ Build Schema-based representation
- ✅ Build Knowledge Graph representation

**Week 5-6**:
- ✅ Traceability evaluation (completely independent)
- ✅ Create benchmark queries

**Week 7-9**:
- ✅ Reasoning capability evaluation
- ✅ Ease of maintenance tests (find 3-5 volunteers)

**Week 10-12**:
- ✅ Topic-aware intervention simulation
- ✅ Explanation generation tests

### **Parts Requiring Downstream Assistance** (Schedule flexibly):

**Week 3** (Early):
- 📞 Confirm API interface specification

**Week 8** (Mid):
- 📞 Get example queries from RAG team

**Week 14-15** (Late, optional):
- 📞 Mock integration test
- 📞 End-to-end demo (if time permits)

---

## 🎯 Summary: Your Evaluation Strategy

### **Core Principle**: 
**Simulation > Real Deployment**

You can evaluate all 5 dimensions **without a complete bot**:

1. **Traceability**: ✅ Completely independent evaluation
2. **Reasoning**: ✅ Simulate with benchmark queries
3. **Maintenance**: ✅ Test with user tasks
4. **Topic-aware intervention**: ✅ Test with simulated discussions
5. **Explanation**: ✅ Manually generate and evaluate

### **Role of Downstream Colleagues**:
- Provide **requirement specifications** and **example queries**
- **Do not need** them to develop a complete bot
- **Optional**: Do integration test at the end to add convincingness

### **How to Write in Thesis**:
```
"We evaluate our representations through simulated bot queries
and user studies, without requiring a fully implemented 
conversational system. This approach allows systematic comparison
while remaining independent of downstream implementation details."
```

---

## 📊 Complete Evaluation Metrics Summary

### RQ2 Metrics

| Dimension | Metric | Method | Independence |
|-----------|--------|--------|--------------|
| **Traceability** | Source completeness (%) | Count facts with sources | ✅ Fully independent |
| | Source accuracy (%) | Manual verification | ✅ Fully independent |
| | Retrieval time (sec) | Time measurement | ✅ Fully independent |
| | Granularity level (1-3) | Assess precision | ✅ Fully independent |
| **Reasoning** | Task coverage | Count supported task types | ✅ Fully independent |
| | Inference correctness (%) | Compare to gold standard | ✅ Fully independent |
| | Query complexity | Lines of code / query length | ✅ Fully independent |
| | Execution time (ms) | Time measurement | ✅ Fully independent |
| **Maintenance** | Readability score (1-5) | Volunteer rating | ⚠️ Need 3-5 volunteers |
| | Edit complexity | Steps required | ⚠️ Need 3-5 volunteers |
| | Task completion time | Time measurement | ⚠️ Need 3-5 volunteers |
| | Schema flexibility | Assess modification effort | ✅ Fully independent |

### RQ3 Metrics

| Dimension | Metric | Method | Independence |
|-----------|--------|--------|--------------|
| **Topic-aware Intervention** | Relevance (1-5) | Rater scoring | ⚠️ Need 3 raters |
| | Completeness (%) | Compare to expected facts | ✅ Fully independent |
| | Response time (sec) | Time measurement | ✅ Fully independent |
| | Context awareness | Qualitative assessment | ⚠️ Need 3 raters |
| **Explanation** | Reasoning chain visibility | Check if path shown | ✅ Fully independent |
| | Explanation completeness (%) | Check components | ✅ Fully independent |
| | Understandability (1-5) | Volunteer rating | ⚠️ Need 3-5 volunteers |

---

## 🔑 Key Takeaways

1. **You don't need a complete bot** - Simulate the bot's needs instead
2. **Most evaluation can be done independently** - Only maintenance and some RQ3 metrics need volunteers
3. **Downstream colleagues provide specs, not implementations** - Get interface requirements and example queries early
4. **Create benchmark queries as your evaluation foundation** - These simulate real bot behavior
5. **Simulated discussion scenarios test intervention capability** - No real conversations needed
6. **Integration test is optional but adds credibility** - Do it at the end if time permits

---

*Document compiled: 2026-02-05*

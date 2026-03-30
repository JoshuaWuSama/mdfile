# Phase 4 - AI 护城河

> 基于 Phase 3，构建高级 AI 能力壁垒：知识图谱引擎、学生学习模型、AI 教学策略、RAG 问答

> **注意**：本阶段部分功能可根据实际情况提前融入 Phase 2/3 开发

---

## 1. 阶段定位

构建系统的 **核心竞争壁垒**，从工具型产品升级为 **AI 学习操作系统**：

- 教材知识图谱引擎（自动构建 + 可视化 + 关系推理）
- BKT 学生知识追踪模型（替代简化版掌握度计算）
- 自适应学习路径推荐
- RAG 教材知识问答（学生提问 → 教材检索 → LLM 回答）
- AI 自动出题（基于知识点 + 难度）
- 艾宾浩斯遗忘曲线复习系统
- 教师 AI 报告 Agent（自动生成教学分析报告）

---

## 2. 系统架构（完整终态）

```
              ┌─────────────────────────────────┐
              │            前端层                │
              │      Vue3 + NaiveUI + Vite      │
              │                                 │
              │ 学生端 / 教师端 / 家长端 / 管理端  │
              └──────────────┬──────────────────┘
                             │
                       Axios (JWT Token)
                             │
         ┌───────────────────┴───────────────────────┐
         │                                           │
  ┌──────▼────────┐                        ┌────────▼────────────┐
  │  业务服务层    │                        │    AI Agent层       │
  │  FastAPI      │                        │    OpenClaw          │
  │               │                        │                     │
  │ 全部 P1-P3    │                        │ 教材解析Agent       │
  │ 业务模块      │                        │ 题目解析Agent       │
  │               │                        │ 学情分析Agent       │
  │ ★ 知识问答    │                        │ 错题归纳Agent       │
  │ ★ 自适应推荐  │                        │ 学习规划Agent       │
  │ ★ 复习提醒    │                        │ ★ 习题生成Agent    │
  │               │                        │ ★ 教师报告Agent    │
  │               │                        │ ★ 知识问答Agent    │
  │               │                        │ ★ 自适应推荐Agent  │
  └──────┬────────┘                        └────────┬────────────┘
         │                                          │
  ┌──────▼──────────────┐              ┌────────────▼─────────────┐
  │     数据层           │              │      AI 能力层           │
  │                     │              │                          │
  │ SQLite              │              │ LLM (DeepSeek)           │
  │ Tortoise-ORM        │              │ ★ RAG 知识检索           │
  │ ★ ChromaDB 向量库   │              │ ★ Embedding 向量化       │
  │                     │              │ ★ 知识图谱推理 (NetworkX)│
  └─────────────────────┘              │ ★ BKT 模型              │
                                       └──────────────────────────┘

★ = Phase 4 新增模块
```

---

## 3. 新增技术栈

在 Phase 3 基础上新增：

### 3.1 后端新增

| 分类 | 技术 | 说明 |
|------|------|------|
| 向量数据库 | ChromaDB | 教材内容向量检索（RAG） |
| Embedding | sentence-transformers / text2vec | 文本向量化 |
| 知识图谱 | NetworkX | 知识点关系图构建与分析 |
| RAG 框架 | LlamaIndex 或 LangChain | 检索增强生成 |
| 图片处理 | Pillow | 教材图片处理 |
| 异步文件 | aiofiles | 大文件异步处理 |

### 3.2 前端新增

| 分类 | 技术 | 说明 |
|------|------|------|
| 知识图谱可视化 | @antv/g6 | 知识点关系图交互式可视化 |
| 对话组件 | 自定义 ChatBox | AI 问答对话界面 |

---

## 4. 新增/变更目录结构

### 4.1 前端新增

```
frontend/src/
├── api/
│   ├── knowledge-graph.ts         # ★ 知识图谱接口
│   ├── chat.ts                    # ★ AI 问答接口
│   ├── adaptive.ts                # ★ 自适应推荐接口
│   └── review.ts                  # ★ 复习系统接口
├── components/
│   ├── KnowledgeGraph.vue         # ★ G6 知识图谱可视化组件
│   ├── ChatBox.vue                # ★ AI 问答对话组件
│   ├── ReviewReminder.vue         # ★ 复习提醒组件
│   └── AdaptivePath.vue           # ★ 学习路径推荐组件
├── composables/
│   ├── useChat.ts                 # ★ AI 问答 hook
│   └── useReview.ts               # ★ 复习计划 hook
├── views/
│   ├── knowledge-graph/
│   │   └── index.vue              # ★ 知识图谱全景页（G6）
│   ├── chat/
│   │   └── index.vue              # ★ AI 知识问答页
│   ├── adaptive/
│   │   └── index.vue              # ★ 自适应学习推荐页
│   ├── review/
│   │   └── index.vue              # ★ 智能复习页
│   └── analysis/
│       └── ai-report.vue          # ★ AI 教学报告页（教师端）
```

### 4.2 后端新增

```
backend/
├── agents/
│   ├── question_gen_agent.py      # ★ 习题生成 Agent
│   ├── report_agent.py            # ★ 教师报告 Agent
│   ├── chat_agent.py              # ★ 知识问答 Agent
│   └── adaptive_agent.py          # ★ 自适应推荐 Agent
├── models/
│   ├── knowledge_edge.py          # ★ 知识点关系边模型
│   ├── review_schedule.py         # ★ 复习计划模型
│   ├── chat_history.py            # ★ 问答历史模型
│   └── adaptive_record.py         # ★ 自适应推荐记录
├── schemas/
│   ├── knowledge_graph.py         # ★
│   ├── chat.py                    # ★
│   ├── review.py                  # ★
│   └── adaptive.py                # ★
├── routes/
│   ├── knowledge_graph.py         # ★
│   ├── chat.py                    # ★
│   ├── review.py                  # ★
│   └── adaptive.py                # ★
├── services/
│   ├── knowledge_graph_service.py # ★ 知识图谱服务
│   ├── chat_service.py            # ★ 问答服务
│   ├── review_service.py          # ★ 复习计划服务
│   ├── adaptive_service.py        # ★ 自适应推荐服务
│   └── rag_service.py             # ★ RAG 检索服务
├── utils/
│   ├── knowledge_graph.py         # ★ NetworkX 知识图谱工具
│   ├── bkt.py                     # ★ BKT 算法实现
│   ├── ebbinghaus.py              # ★ 遗忘曲线算法
│   └── embedding.py               # ★ 文本向量化工具
├── rag/                           # ★ RAG 模块
│   ├── __init__.py
│   ├── indexer.py                 # 教材向量索引构建
│   ├── retriever.py               # 相似内容检索
│   └── generator.py               # 检索 + 生成
├── data/
│   └── chroma_db/                 # ★ ChromaDB 数据目录
```

---

## 5. 数据库变更

### 5.1 新增表

#### 知识点关系边表

```sql
knowledge_edge
├── id              INTEGER PRIMARY KEY
├── source_id       INTEGER FK → knowledge_node.id   -- 前置知识
├── target_id       INTEGER FK → knowledge_node.id   -- 后续知识
├── relation_type   VARCHAR(50)                      -- prerequisite/contains/related
├── weight          FLOAT DEFAULT 1.0                -- 关系强度
├── ai_generated    BOOLEAN DEFAULT TRUE
├── created_at      DATETIME
└── updated_at      DATETIME
```

关系类型说明：

```
prerequisite  — 前置依赖（学 A 才能学 B）
contains      — 包含关系（A 包含 B）
related       — 关联关系（A 和 B 相关）
```

#### 复习计划表

```sql
review_schedule
├── id              INTEGER PRIMARY KEY
├── user_id         INTEGER FK → user.id
├── knowledge_id    INTEGER FK → knowledge_node.id
├── question_id     INTEGER FK → question.id         -- 可空，指定复习题
├── review_stage    INTEGER DEFAULT 1                -- 复习阶段（1~7）
├── next_review_at  DATETIME                         -- 下次复习时间
├── last_reviewed   DATETIME
├── is_completed    BOOLEAN DEFAULT FALSE
├── created_at      DATETIME
└── updated_at      DATETIME
```

艾宾浩斯复习间隔：

```
Stage 1: 1 天后
Stage 2: 2 天后
Stage 3: 4 天后
Stage 4: 7 天后
Stage 5: 15 天后
Stage 6: 30 天后
Stage 7: 完成（标记掌握）
```

#### 问答历史表

```sql
chat_history
├── id              INTEGER PRIMARY KEY
├── user_id         INTEGER FK → user.id
├── session_id      VARCHAR(50)                      -- 对话会话ID
├── role            VARCHAR(10)                      -- user / assistant
├── content         TEXT NOT NULL
├── knowledge_ids   JSON                             -- 关联知识点
├── source_chunks   JSON                             -- RAG 检索的原文片段
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 自适应推荐记录表

```sql
adaptive_record
├── id              INTEGER PRIMARY KEY
├── user_id         INTEGER FK → user.id
├── student_id      INTEGER FK → student.id          -- 可空（个人模式）
├── recommend_type  VARCHAR(30)                      -- question/knowledge/plan
├── recommend_data  JSON                             -- 推荐内容
├── reason          TEXT                             -- 推荐理由
├── accepted        BOOLEAN                          -- 是否采纳
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 教材向量索引表（元数据）

```sql
textbook_chunk
├── id              INTEGER PRIMARY KEY
├── textbook_id     INTEGER FK → textbook.id
├── chapter_id      INTEGER FK → chapter.id
├── chunk_index     INTEGER                          -- 分片序号
├── content         TEXT NOT NULL                    -- 原文内容
├── embedding_id    VARCHAR(100)                     -- ChromaDB 中的 ID
├── created_at      DATETIME
└── updated_at      DATETIME
```

### 5.2 变更表

Phase 1 的 `student_knowledge_state` 增强：

```sql
student_knowledge_state
+ p_learn       FLOAT DEFAULT 0.0      -- BKT: 学习概率
+ p_slip        FLOAT DEFAULT 0.1      -- BKT: 失误概率
+ p_guess       FLOAT DEFAULT 0.25     -- BKT: 猜测概率
+ p_transit     FLOAT DEFAULT 0.1      -- BKT: 转移概率
+ bkt_mastery   FLOAT DEFAULT 0.0      -- BKT 计算的掌握度
```

---

## 6. 核心算法

### 6.1 BKT（贝叶斯知识追踪）

替代 Phase 1 的简化掌握度计算：

```python
class BKT:
    """贝叶斯知识追踪模型"""
    
    def __init__(self, p_learn=0.0, p_slip=0.1, p_guess=0.25, p_transit=0.1):
        self.p_learn = p_learn    # 初始掌握概率
        self.p_slip = p_slip      # 已掌握但答错概率
        self.p_guess = p_guess    # 未掌握但答对概率
        self.p_transit = p_transit # 未掌握→掌握的转移概率
    
    def update(self, p_mastery: float, is_correct: bool) -> float:
        """根据答题结果更新掌握概率"""
        if is_correct:
            # 答对时的后验概率
            p_correct_given_mastery = 1 - self.p_slip
            p_correct_given_not = self.p_guess
            
            numerator = p_mastery * p_correct_given_mastery
            denominator = numerator + (1 - p_mastery) * p_correct_given_not
            p_learned = numerator / denominator if denominator > 0 else p_mastery
        else:
            # 答错时的后验概率
            p_wrong_given_mastery = self.p_slip
            p_wrong_given_not = 1 - self.p_guess
            
            numerator = p_mastery * p_wrong_given_mastery
            denominator = numerator + (1 - p_mastery) * p_wrong_given_not
            p_learned = numerator / denominator if denominator > 0 else p_mastery
        
        # 考虑学习转移
        p_mastery_new = p_learned + (1 - p_learned) * self.p_transit
        
        return min(max(p_mastery_new, 0.0), 1.0)
```

### 6.2 艾宾浩斯遗忘曲线

```python
# 复习间隔（天）
REVIEW_INTERVALS = [1, 2, 4, 7, 15, 30]

def get_next_review_time(stage: int, last_reviewed: datetime) -> datetime:
    """根据复习阶段计算下次复习时间"""
    if stage > len(REVIEW_INTERVALS):
        return None  # 已完成所有复习
    interval = REVIEW_INTERVALS[stage - 1]
    return last_reviewed + timedelta(days=interval)

def calculate_retention(hours_since_last: float) -> float:
    """计算记忆保持率"""
    import math
    # 艾宾浩斯遗忘曲线: R = e^(-t/S)
    # S = 记忆稳定性因子
    S = 24  # 初始稳定性（小时）
    return math.exp(-hours_since_last / S)
```

### 6.3 自适应学习路径

```python
def recommend_next_knowledge(user_id: int, subject_id: int) -> list:
    """基于知识图谱 + 掌握度推荐下一步学习内容"""
    
    # 1. 获取知识图谱
    graph = build_knowledge_graph(subject_id)
    
    # 2. 获取学生掌握度
    mastery = get_student_mastery(user_id, subject_id)
    
    # 3. 找到未掌握 + 前置已掌握的知识点
    candidates = []
    for node in graph.nodes:
        if mastery.get(node, 0) < 0.6:  # 未掌握
            prerequisites = get_prerequisites(graph, node)
            if all(mastery.get(p, 0) >= 0.6 for p in prerequisites):
                candidates.append(node)
    
    # 4. 按优先级排序（掌握度低 + 难度低 优先）
    candidates.sort(key=lambda n: (mastery.get(n, 0), graph.nodes[n]['difficulty']))
    
    return candidates[:5]
```

### 6.4 RAG 知识问答

```
学生提问
  │
  ▼
Embedding 向量化
  │
  ▼
ChromaDB 相似度检索（Top K）
  │
  ▼
拼接检索结果 + 原始问题
  │
  ▼
LLM 生成回答
  │
  ▼
返回答案 + 引用来源
```

---

## 7. AI Agent 设计（新增）

### 7.1 新增 Agent

| Agent | 输入 | 输出 | 触发时机 |
|-------|------|------|----------|
| 习题生成Agent | 知识点 + 难度 + 题型 + 数量 | 自动生成题目 | 教师手动触发 |
| 教师报告Agent | 班级学情数据 + 时间范围 | 教学分析报告（Markdown） | 教师触发 / 定时 |
| 知识问答Agent | 学生问题 + 教材上下文 | 基于教材的回答 | 学生提问时 |
| 自适应推荐Agent | 掌握度模型 + 知识图谱 | 推荐学习内容 + 练习题 | 学生访问推荐页 |

### 7.2 Agent 工作流（完整）

```
教材上传 → 教材解析Agent → 章节拆分 + 知识点提取
                │
                ├→ 知识关系抽取 → 知识图谱构建（NetworkX）
                └→ 文本分片 → 向量化 → ChromaDB 索引

题目录入 → 题目解析Agent → 知识点绑定 + 难度评估

学生做题 → 学情分析Agent → BKT 模型更新掌握度
                │
                ├→ 错题归纳Agent → 错因分析
                ├→ 自适应推荐Agent → 推荐下一步学习
                └→ 复习调度 → 艾宾浩斯计划

学生提问 → 知识问答Agent → RAG 检索 + LLM 回答

教师查看 → 教师报告Agent → 生成教学分析报告

教师出题 → 习题生成Agent → 基于知识点自动出题
```

---

## 8. 新增接口

### 8.1 知识图谱 `/api/v1/knowledge-graph`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/knowledge-graph/{subject_id}` | 获取学科知识图谱（节点+边） | 全部 |
| GET | `/api/v1/knowledge-graph/{subject_id}/mastery?user_id=` | 带掌握度的知识图谱 | student/teacher |
| POST | `/api/v1/knowledge-graph/edges` | 创建知识关系 | teacher/admin |
| PUT | `/api/v1/knowledge-graph/edges/{id}` | 更新知识关系 | teacher/admin |
| DELETE | `/api/v1/knowledge-graph/edges/{id}` | 删除知识关系 | teacher/admin |
| POST | `/api/v1/knowledge-graph/{subject_id}/auto-build` | AI 自动构建关系 | teacher/admin |
| GET | `/api/v1/knowledge-graph/{subject_id}/prerequisites/{node_id}` | 查询前置知识 | 全部 |
| GET | `/api/v1/knowledge-graph/{subject_id}/path?from=&to=` | 知识学习路径 | 全部 |

### 8.2 AI 问答 `/api/v1/chat`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| POST | `/api/v1/chat/ask` | 提问（RAG） | student |
| GET | `/api/v1/chat/history?session_id=` | 对话历史 | student |
| GET | `/api/v1/chat/sessions` | 对话会话列表 | student |
| DELETE | `/api/v1/chat/sessions/{session_id}` | 删除对话 | student |

### 8.3 自适应推荐 `/api/v1/adaptive`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/adaptive/recommend?subject_id=` | 获取学习推荐 | student |
| GET | `/api/v1/adaptive/recommend-questions?knowledge_id=&count=` | 推荐练习题 | student |
| POST | `/api/v1/adaptive/feedback` | 推荐反馈（是否采纳） | student |
| GET | `/api/v1/adaptive/learning-path?subject_id=` | 自适应学习路径 | student |

### 8.4 复习系统 `/api/v1/review`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/review/today` | 今日待复习列表 | student |
| GET | `/api/v1/review/schedule?days=7` | 复习计划（未来N天） | student |
| POST | `/api/v1/review/{id}/complete` | 完成复习 | student |
| POST | `/api/v1/review/{id}/skip` | 跳过复习 | student |
| GET | `/api/v1/review/stats` | 复习统计（完成率、记忆曲线） | student |

### 8.5 AI 出题 `/api/v1/ai/generate-questions`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| POST | `/api/v1/ai/generate-questions` | AI 生成题目 | teacher |

请求体：

```json
{
  "subject_id": 1,
  "knowledge_ids": [1, 2, 3],
  "count": 5,
  "difficulty": 0.6,
  "types": ["single_choice", "fill"]
}
```

### 8.6 教师 AI 报告 `/api/v1/ai/report`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| POST | `/api/v1/ai/report/class/{class_id}` | 生成班级教学报告 | teacher |
| GET | `/api/v1/ai/report/{id}` | 查看报告 | teacher |
| GET | `/api/v1/ai/report/list?class_id=` | 报告列表 | teacher |

---

## 9. 页面设计

### 9.1 新增页面

| 页面 | 路由 | 角色 | 功能 |
|------|------|------|------|
| 知识图谱全景 | `/knowledge-graph` | 全部 | G6 交互式知识图谱，节点颜色按掌握度着色 |
| AI 知识问答 | `/chat` | student | 对话式问答，基于教材 RAG |
| 自适应学习 | `/adaptive` | student | AI 推荐学习内容、推荐练习题 |
| 智能复习 | `/review` | student | 今日复习列表、记忆曲线可视化 |
| AI 出题 | `/question/ai-generate` | teacher | 选知识点/难度/题型 → AI 生成 |
| AI 教学报告 | `/analysis/ai-report` | teacher | AI 生成的班级教学分析报告 |

### 9.2 改造页面

| 页面 | 改造内容 |
|------|----------|
| 学习仪表盘 | 增加：今日复习提醒、AI 推荐卡片 |
| 学情分析 | 掌握度替换为 BKT 模型输出 |
| 教材详情 | 增加知识图谱可视化入口 |
| 错题本 | 增加：关联复习计划、AI 推荐同类题 |
| 练习入口 | 增加：「AI 推荐模式」（自适应出题） |

---

## 10. 可提前融入 Phase 2/3 的功能

以下功能可根据开发进度提前引入：

### 融入 Phase 2

| 功能 | 说明 | 难度 |
|------|------|------|
| BKT 掌握度算法 | 替代简化版算法，提升准确度 | 中 |
| 艾宾浩斯复习提醒 | 错题产生后自动生成复习计划 | 低 |

### 融入 Phase 3

| 功能 | 说明 | 难度 |
|------|------|------|
| 教师报告 Agent | 教师查看班级报告时 AI 生成教学建议 | 中 |
| 知识图谱可视化 | 知识点管理页增加图谱视图 | 中 |
| AI 自动出题 | 教师发布作业时可 AI 生成题目 | 中 |

---

## 11. 环境变量新增

### 后端 `.env.development` 新增

```env
# ChromaDB
CHROMA_DB_PATH=data/chroma_db
CHROMA_COLLECTION_NAME=textbook_chunks

# Embedding
EMBEDDING_MODEL=shibing624/text2vec-base-chinese
EMBEDDING_DEVICE=cpu

# RAG
RAG_CHUNK_SIZE=500
RAG_CHUNK_OVERLAP=50
RAG_TOP_K=5

# BKT 默认参数
BKT_P_SLIP=0.1
BKT_P_GUESS=0.25
BKT_P_TRANSIT=0.1

# 复习系统
REVIEW_ENABLED=true
```

---

## 12. 初始化数据

在 Phase 3 基础上新增：

- 预构建示例教材的知识图谱（节点 + 边）
- 预建立 ChromaDB 向量索引（示例教材）
- 初始化 BKT 参数默认值
- 预设复习计划示例数据

---

## 13. 推荐成熟插件汇总

### 前端

| 插件 | 用途 | 包名 |
|------|------|------|
| @antv/g6 | 知识图谱可视化 | `@antv/g6` |
| echarts + vue-echarts | 雷达图/折线图/记忆曲线 | `echarts` + `vue-echarts` |
| markdown-it + katex | 题目/报告渲染 | `markdown-it` + `katex` |
| vue-pdf-embed | PDF 预览 | `vue-pdf-embed` |
| @vueuse/core | 通用 Composables | `@vueuse/core` |

### 后端

| 插件 | 用途 | 包名 |
|------|------|------|
| chromadb | 向量数据库 | `chromadb` |
| sentence-transformers | 中文文本向量化 | `sentence-transformers` |
| networkx | 知识图谱 | `networkx` |
| PyMuPDF | PDF 解析 | `PyMuPDF` |
| aiofiles | 异步文件 | `aiofiles` |
| Pillow | 图片处理 | `Pillow` |
| apscheduler | 定时复习提醒 | `apscheduler` |

---

## 14. 开发任务清单

| 序号 | 模块 | 工作内容 | 优先级 |
|------|------|----------|--------|
| 1 | BKT 算法 | 实现 BKT 模型，替代简化掌握度 | P0 |
| 2 | 知识图谱存储 | knowledge_edge 表 + NetworkX 构建 | P0 |
| 3 | 知识图谱可视化 | G6 交互式图谱组件 | P0 |
| 4 | RAG 索引 | 教材分片 + Embedding + ChromaDB 索引 | P0 |
| 5 | RAG 问答 | 检索 + LLM 生成 + 对话界面 | P0 |
| 6 | AI 出题 | 习题生成Agent + 教师端页面 | P1 |
| 7 | 遗忘曲线 | 复习计划算法 + 复习列表页面 | P1 |
| 8 | 自适应推荐 | 推荐算法 + 推荐页面 | P1 |
| 9 | 教师报告 | 报告 Agent + 报告展示页面 | P1 |
| 10 | 知识关系自动构建 | AI 自动抽取知识点关系 | P2 |
| 11 | 掌握度迁移 | 将历史数据迁移到 BKT 模型 | P2 |
| 12 | 学习路径推荐 | 基于图谱的最优学习路径 | P2 |

---

## 15. 注意事项

- ChromaDB 数据较大，建议独立存储目录，不纳入 Git
- Embedding 模型首次加载较慢，建议预下载
- BKT 参数可按学科/知识点微调，建议提供管理界面
- RAG 检索结果需在前端展示引用来源（教材章节）
- 知识图谱可视化节点数量大时需启用 G6 的布局优化
- AI 生成的题目需人工审核后才能入库
- 复习提醒可通过定时任务 + 前端仪表盘实现，暂不需要推送

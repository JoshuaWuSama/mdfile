# Phase 1 - 个人学习 OS

> 阶段目标：构建面向个人学习者的 AI 学习助手，单用户模式，聚焦核心学习能力

---

## 1. 阶段定位

本阶段构建一个 **个人学习操作系统**，无需登录鉴权，单用户直接使用。核心能力：

- 教材上传与 AI 解析（PDF → 章节 + 知识点提取）
- 个人题库管理（手动录入 + AI 辅助解析）
- 自主练习做题
- 错题自动归纳 + AI 错因分析
- 个人学情分析（知识掌握度追踪）
- AI 学习计划生成
- 知识点浏览与关联

---

## 2. 系统架构

```
              ┌─────────────────────────┐
              │        前端层            │
              │  Vue3 + NaiveUI + Vite  │
              │                         │
              │    个人学习工作台        │
              └───────────┬─────────────┘
                          │
                    Axios (HTTP)
                          │
         ┌────────────────┴────────────────┐
         │                                 │
  ┌──────▼────────┐              ┌────────▼────────┐
  │  业务服务层    │              │   AI Agent层    │
  │  FastAPI      │              │   OpenClaw      │
  │               │              │                 │
  │ 教材管理       │              │ 教材解析Agent   │
  │ 题库管理       │              │ 题目解析Agent   │
  │ 练习系统       │              │ 学情分析Agent   │
  │ 错题系统       │              │ 错题归纳Agent   │
  │ 学情分析       │              │ 学习规划Agent   │
  │ 学习计划       │              │                 │
  └──────┬────────┘              └────────┬────────┘
         │                                │
  ┌──────▼────────┐              ┌────────▼────────┐
  │   数据层       │              │   AI 能力层     │
  │ SQLite        │              │ LLM (DeepSeek)  │
  │ Tortoise-ORM  │              │ httpx 异步调用   │
  └───────────────┘              └─────────────────┘
```

---

## 3. 本阶段技术栈

### 3.1 前端

| 分类 | 技术 | 说明 |
|------|------|------|
| 框架 | Vue3 + TypeScript | Composition API |
| 构建 | Vite | HMR 热更新 |
| UI | NaiveUI | 企业级组件库 |
| 状态管理 | Pinia | 轻量状态管理 |
| 路由 | Vue Router 4 | 前端路由 |
| HTTP | Axios | @/utils/request 封装 |
| 图表 | vue-echarts + ECharts | 掌握度雷达图、学情趋势 |
| 样式 | UnoCSS + Sass | @unocss/reset/tailwind-compat.css |
| 动画 | @vueuse/motion | 页面过渡 |
| 进度条 | nprogress | 路由切换 |
| 图标 | @vicons/ionicons5 | 图标库 |
| 日期 | Day.js | 日期处理 |
| 规范 | ESLint | 代码规范 |
| 包管理 | PNPM | 依赖管理 |
| Markdown | markdown-it + katex | 题目内容 + 数学公式 |
| PDF 预览 | vue-pdf-embed | 教材预览 |

### 3.2 后端

| 分类 | 技术 | 说明 |
|------|------|------|
| 框架 | FastAPI | 异步 Web 框架 |
| ORM | Tortoise-ORM | 异步 ORM，SQLite |
| 验证 | Pydantic V2 | schemas 输入/输出 |
| 日志 | loguru | 结构化日志 |
| 服务器 | Uvicorn | ASGI 服务器 |
| HTTP | httpx | 异步调用 LLM API |
| 配置 | python-dotenv | 环境变量 |
| 文件 | python-multipart | PDF 上传 |
| PDF | PyMuPDF (fitz) | PDF 文本提取 |
| 测试 | pytest + pytest-asyncio | 异步测试 |

### 3.3 AI

| 分类 | 技术 | 说明 |
|------|------|------|
| LLM | DeepSeek API | 知识提取/错题分析/学习建议 |
| Agent | OpenClaw | Agent 工作流编排 |

### 3.4 数据库

| 类型 | 技术 |
|------|------|
| 主数据库 | SQLite（Tortoise-ORM 驱动） |

---

## 4. 项目目录结构

### 4.1 前端

```
frontend/
├── public/
├── src/
│   ├── api/                       # API 接口
│   │   ├── subject.ts             # 学科接口
│   │   ├── textbook.ts            # 教材接口
│   │   ├── knowledge.ts           # 知识点接口
│   │   ├── question.ts            # 题目接口
│   │   ├── practice.ts            # 练习接口
│   │   ├── wrong-question.ts      # 错题接口
│   │   ├── learning-plan.ts       # 学习计划接口
│   │   ├── analysis.ts            # 学情分析接口
│   │   └── ai.ts                  # AI 能力接口
│   ├── assets/                    # 静态资源
│   ├── components/                # 公共组件
│   │   ├── QuestionCard.vue       # 题目卡片
│   │   ├── MasteryRadar.vue       # 掌握度雷达图
│   │   ├── KnowledgeTree.vue      # 知识点树
│   │   ├── Pagination.vue         # 分页组件
│   │   └── SearchFilter.vue       # 搜索筛选
│   ├── composables/               # 组合式函数
│   │   ├── useTable.ts            # 表格 hook（分页/排序/筛选）
│   │   └── useKnowledge.ts        # 知识点 hook
│   ├── layouts/                   # 布局
│   │   └── MainLayout.vue         # 主布局（侧边栏 + 头部）
│   ├── router/
│   │   └── index.ts
│   ├── stores/
│   │   ├── app.ts                 # 应用状态
│   │   └── knowledge.ts           # 知识点状态
│   ├── styles/
│   │   ├── variables.scss
│   │   └── global.scss
│   ├── types/
│   │   ├── api.d.ts               # API 响应类型
│   │   ├── model.d.ts             # 模型类型
│   │   └── common.d.ts            # 通用类型
│   ├── utils/
│   │   ├── request.ts             # Axios 封装
│   │   └── format.ts              # 格式化工具
│   ├── views/
│   │   ├── dashboard/
│   │   │   └── index.vue          # 学习仪表盘
│   │   ├── textbook/
│   │   │   ├── index.vue          # 教材列表
│   │   │   ├── upload.vue         # 教材上传
│   │   │   └── detail.vue         # 教材详情（章节 + 知识点）
│   │   ├── question/
│   │   │   ├── index.vue          # 题库列表
│   │   │   ├── create.vue         # 创建/编辑题目
│   │   │   └── detail.vue         # 题目详情
│   │   ├── practice/
│   │   │   ├── index.vue          # 练习入口（选学科/章节）
│   │   │   └── session.vue        # 做题页面
│   │   ├── wrong-question/
│   │   │   ├── index.vue          # 错题本列表
│   │   │   └── review.vue         # 错题复习
│   │   ├── learning-plan/
│   │   │   ├── index.vue          # 学习计划列表
│   │   │   └── detail.vue         # 计划详情
│   │   ├── analysis/
│   │   │   └── index.vue          # 个人学情分析
│   │   └── settings/
│   │       └── index.vue          # 系统设置（AI配置等）
│   ├── App.vue
│   └── main.ts
├── .env.development
├── .env.production
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
├── uno.config.ts
├── .eslintrc.cjs
└── pnpm-lock.yaml
```

### 4.2 后端

```
backend/
├── main.py                        # 应用入口
├── config/
│   ├── __init__.py
│   └── settings.py                # 全局配置
├── core/
│   ├── __init__.py
│   ├── database.py                # Tortoise-ORM 初始化
│   ├── middleware.py              # 全局异常处理
│   ├── logger.py                  # loguru 日志
│   └── response.py                # 统一响应工具
├── models/
│   ├── __init__.py
│   ├── subject.py                 # 学科模型
│   ├── textbook.py                # 教材模型
│   ├── chapter.py                 # 章节模型
│   ├── knowledge_node.py          # 知识点模型
│   ├── question.py                # 题目模型
│   ├── practice.py                # 练习记录模型
│   ├── wrong_question.py          # 错题模型
│   ├── student_knowledge.py       # 知识掌握度模型
│   └── learning_plan.py           # 学习计划模型
├── schemas/
│   ├── __init__.py
│   ├── common.py                  # 通用响应 Schema
│   ├── subject.py
│   ├── textbook.py
│   ├── knowledge.py
│   ├── question.py
│   ├── practice.py
│   ├── wrong_question.py
│   ├── learning_plan.py
│   └── analysis.py
├── routes/
│   ├── __init__.py
│   ├── subject.py                 # 学科路由
│   ├── textbook.py                # 教材路由
│   ├── knowledge.py               # 知识点路由
│   ├── question.py                # 题目路由
│   ├── practice.py                # 练习路由
│   ├── wrong_question.py          # 错题路由
│   ├── learning_plan.py           # 学习计划路由
│   ├── analysis.py                # 学情分析路由
│   └── ai.py                     # AI 能力路由
├── services/
│   ├── __init__.py
│   ├── subject_service.py
│   ├── textbook_service.py
│   ├── knowledge_service.py
│   ├── question_service.py
│   ├── practice_service.py
│   ├── wrong_question_service.py
│   ├── learning_plan_service.py
│   ├── analysis_service.py
│   └── ai_service.py             # LLM 调用封装
├── agents/                        # OpenClaw Agent
│   ├── __init__.py
│   ├── textbook_agent.py          # 教材解析 Agent
│   ├── question_agent.py          # 题目解析 Agent
│   ├── analysis_agent.py          # 学情分析 Agent
│   ├── wrong_question_agent.py    # 错题归纳 Agent
│   └── learning_plan_agent.py     # 学习规划 Agent
├── utils/
│   ├── __init__.py
│   ├── pdf_parser.py              # PDF 解析工具
│   └── mastery_calc.py            # 掌握度计算
├── data/
│   └── aiedu.db
├── docs/
├── logs/
├── scripts/
│   ├── init_db.py                 # 数据库初始化
│   └── seed_data.py               # 测试数据
├── uploads/                       # 教材 PDF 存储
├── .env.development
├── .env.production
├── requirements.txt
└── pytest.ini
```

---

## 5. 数据库设计

### 5.1 ER 关系

```
Subject ──1:N── Textbook ──1:N── Chapter
  │
  └──1:N── KnowledgeNode（自关联树，关联 Chapter）
                 │
                 └──N:M── Question（通过 QuestionKnowledgeMap）
                              │
                              ├──1:N── QuestionOption
                              │
                              └──1:N── PracticeAnswer ──N:1── PracticeSession
                                          │
                                          └── WrongQuestion

KnowledgeNode ──1:N── StudentKnowledgeState

LearningPlan（独立表）

AiAnalysisLog（独立表）
```

### 5.2 表结构

#### 学科表

```sql
subject
├── id              INTEGER PRIMARY KEY
├── name            VARCHAR(50) NOT NULL    -- 数学、物理、化学...
├── icon            VARCHAR(50)             -- 图标标识
├── description     TEXT
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 教材表

```sql
textbook
├── id              INTEGER PRIMARY KEY
├── subject_id      INTEGER FK → subject.id
├── title           VARCHAR(200) NOT NULL
├── publisher       VARCHAR(100)            -- 出版社
├── edition         VARCHAR(50)             -- 版本
├── grade           VARCHAR(20)             -- 年级
├── semester        VARCHAR(10)             -- 上册/下册
├── file_path       VARCHAR(500)            -- PDF 文件路径
├── parse_status    VARCHAR(20) DEFAULT 'pending'  -- pending/parsing/completed/failed
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 章节表

```sql
chapter
├── id              INTEGER PRIMARY KEY
├── textbook_id     INTEGER FK → textbook.id
├── title           VARCHAR(200) NOT NULL
├── parent_id       INTEGER FK → chapter.id  -- 多级章节
├── order_index     INTEGER
├── content_summary TEXT                     -- AI 摘要
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 知识点表（树形）

```sql
knowledge_node
├── id              INTEGER PRIMARY KEY
├── subject_id      INTEGER FK → subject.id
├── chapter_id      INTEGER FK → chapter.id  -- 关联章节（可空）
├── name            VARCHAR(200) NOT NULL
├── parent_id       INTEGER FK → knowledge_node.id  -- 树形自关联
├── difficulty      FLOAT DEFAULT 0.5        -- 难度 0~1
├── description     TEXT
├── order_index     INTEGER
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 题目表

```sql
question
├── id              INTEGER PRIMARY KEY
├── subject_id      INTEGER FK → subject.id
├── type            VARCHAR(20) NOT NULL     -- single_choice/multi_choice/fill/essay/calculate
├── difficulty      FLOAT DEFAULT 0.5        -- 难度 0~1
├── content         TEXT NOT NULL             -- Markdown 格式
├── answer          TEXT NOT NULL             -- 标准答案
├── analysis        TEXT                      -- 解题分析
├── source          VARCHAR(200)              -- 来源
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 题目选项表

```sql
question_option
├── id              INTEGER PRIMARY KEY
├── question_id     INTEGER FK → question.id
├── option_label    VARCHAR(10) NOT NULL      -- A / B / C / D
├── content         TEXT NOT NULL
├── is_correct      BOOLEAN DEFAULT FALSE
└── order_index     INTEGER
```

#### 题目-知识点关联表

```sql
question_knowledge_map
├── id              INTEGER PRIMARY KEY
├── question_id     INTEGER FK → question.id
├── knowledge_id    INTEGER FK → knowledge_node.id
└── weight          FLOAT DEFAULT 1.0
```

#### 练习会话表

```sql
practice_session
├── id              INTEGER PRIMARY KEY
├── subject_id      INTEGER FK → subject.id
├── title           VARCHAR(200)              -- 练习标题（自动/手动）
├── mode            VARCHAR(20)               -- random/knowledge/chapter/wrong_review
├── total_count     INTEGER DEFAULT 0
├── correct_count   INTEGER DEFAULT 0
├── start_time      DATETIME
├── end_time        DATETIME
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 练习答题明细表

```sql
practice_answer
├── id              INTEGER PRIMARY KEY
├── session_id      INTEGER FK → practice_session.id
├── question_id     INTEGER FK → question.id
├── answer          TEXT                      -- 用户答案
├── is_correct      BOOLEAN
├── time_spent      INTEGER                  -- 耗时（秒）
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 错题表

```sql
wrong_question
├── id              INTEGER PRIMARY KEY
├── question_id     INTEGER FK → question.id
├── source_type     VARCHAR(20)               -- practice
├── source_id       INTEGER                   -- practice_session.id
├── student_answer  TEXT                      -- 错误答案
├── error_type      VARCHAR(50)               -- concept/calculation/comprehension/careless
├── ai_analysis     TEXT                      -- AI 错因分析
├── first_wrong_at  DATETIME
├── last_review_at  DATETIME
├── review_count    INTEGER DEFAULT 0
├── mastered        BOOLEAN DEFAULT FALSE
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 知识掌握度表

```sql
student_knowledge_state
├── id              INTEGER PRIMARY KEY
├── knowledge_id    INTEGER FK → knowledge_node.id
├── mastery         FLOAT DEFAULT 0.0         -- 掌握度 0~1
├── confidence      FLOAT DEFAULT 0.0         -- 置信度 0~1
├── attempt_count   INTEGER DEFAULT 0         -- 答题次数
├── correct_count   INTEGER DEFAULT 0         -- 正确次数
├── last_practice   DATETIME
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 学习计划表

```sql
learning_plan
├── id              INTEGER PRIMARY KEY
├── subject_id      INTEGER FK → subject.id
├── title           VARCHAR(200)
├── plan_type       VARCHAR(20)               -- daily/weekly/review
├── start_date      DATE
├── end_date        DATE
├── content         JSON                      -- 计划详情
├── ai_generated    BOOLEAN DEFAULT TRUE
├── status          VARCHAR(20) DEFAULT 'active'  -- active/completed/expired
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### AI 分析日志表

```sql
ai_analysis_log
├── id              INTEGER PRIMARY KEY
├── analysis_type   VARCHAR(50)               -- textbook_parse/question_parse/mastery_update/wrong_analysis/plan_generation
├── input_data      JSON
├── output_data     JSON
├── agent_name      VARCHAR(100)
├── created_at      DATETIME
└── updated_at      DATETIME
```

---

## 6. 接口设计

### 6.1 统一响应格式

```json
{"code": 200, "message": "success", "data": {...}}
{"code": 400, "message": "错误信息", "data": null}
{"code": 200, "message": "success", "data": {"items": [...], "total": 100, "page": 1, "page_size": 20}}
```

### 6.2 接口列表

#### 学科 `/api/v1/subjects`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/subjects` | 学科列表 |
| POST | `/api/v1/subjects` | 创建学科 |
| PUT | `/api/v1/subjects/{id}` | 更新学科 |
| DELETE | `/api/v1/subjects/{id}` | 删除学科 |

#### 教材 `/api/v1/textbooks`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/textbooks` | 教材列表 |
| POST | `/api/v1/textbooks` | 创建教材 |
| POST | `/api/v1/textbooks/upload` | 上传教材 PDF |
| POST | `/api/v1/textbooks/{id}/parse` | 触发 AI 解析 |
| GET | `/api/v1/textbooks/{id}` | 教材详情（含章节） |
| PUT | `/api/v1/textbooks/{id}` | 更新教材 |
| DELETE | `/api/v1/textbooks/{id}` | 删除教材 |
| GET | `/api/v1/textbooks/{id}/chapters` | 章节列表 |

#### 知识点 `/api/v1/knowledge`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/knowledge?subject_id=&chapter_id=` | 知识点树 |
| GET | `/api/v1/knowledge/{id}` | 知识点详情 |

#### 题库 `/api/v1/questions`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/questions?subject_id=&type=&difficulty=&page=&page_size=` | 题目列表（分页/筛选） |
| POST | `/api/v1/questions` | 创建题目 |
| GET | `/api/v1/questions/{id}` | 题目详情 |
| PUT | `/api/v1/questions/{id}` | 更新题目 |
| DELETE | `/api/v1/questions/{id}` | 删除题目 |

#### 练习 `/api/v1/practice`

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/v1/practice/start` | 开始练习（指定模式/范围） |
| GET | `/api/v1/practice/{session_id}/questions` | 获取练习题目 |
| POST | `/api/v1/practice/{session_id}/answer` | 提交单题答案 |
| POST | `/api/v1/practice/{session_id}/finish` | 结束练习 |
| GET | `/api/v1/practice/history` | 练习历史记录 |
| GET | `/api/v1/practice/{session_id}/result` | 练习结果详情 |

#### 错题本 `/api/v1/wrong-questions`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/wrong-questions?subject_id=&error_type=&mastered=` | 错题列表（筛选） |
| GET | `/api/v1/wrong-questions/{id}` | 错题详情 + AI 分析 |
| POST | `/api/v1/wrong-questions/{id}/review` | 标记已复习 |
| PUT | `/api/v1/wrong-questions/{id}/mastered` | 标记已掌握 |
| DELETE | `/api/v1/wrong-questions/{id}` | 删除错题 |
| POST | `/api/v1/wrong-questions/review-session` | 开始错题复习练习 |

#### 学习计划 `/api/v1/learning-plans`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/learning-plans` | 计划列表 |
| POST | `/api/v1/learning-plans/generate` | AI 生成学习计划 |
| POST | `/api/v1/learning-plans` | 手动创建计划 |
| GET | `/api/v1/learning-plans/{id}` | 计划详情 |
| PUT | `/api/v1/learning-plans/{id}` | 更新计划 |
| PUT | `/api/v1/learning-plans/{id}/complete` | 完成计划 |
| DELETE | `/api/v1/learning-plans/{id}` | 删除计划 |

#### 学情分析 `/api/v1/analysis`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | `/api/v1/analysis/overview` | 学情总览（掌握度概览） |
| GET | `/api/v1/analysis/mastery?subject_id=` | 知识掌握度详情 |
| GET | `/api/v1/analysis/trend?subject_id=&days=` | 学习趋势 |
| GET | `/api/v1/analysis/weakness?subject_id=` | 薄弱知识点 |
| GET | `/api/v1/analysis/practice-stats` | 练习统计 |

#### AI 能力 `/api/v1/ai`

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/v1/ai/parse-textbook` | AI 解析教材 |
| POST | `/api/v1/ai/parse-question` | AI 解析题目（自动绑定知识点） |
| POST | `/api/v1/ai/analyze-wrong` | AI 分析错因 |
| POST | `/api/v1/ai/generate-plan` | AI 生成学习计划 |
| GET | `/api/v1/ai/config` | AI 配置信息 |
| PUT | `/api/v1/ai/config` | 更新 AI 配置 |

---

## 7. AI Agent 设计

### 7.1 Agent 工作流

```
教材上传 → 教材解析Agent → 章节拆分 + 知识点提取
                                 ↓
题目录入 → 题目解析Agent → 自动绑定知识点 + 难度评估
                                 ↓
         自主练习做题 → 学情分析Agent → 更新知识掌握度
                                 ↓
                      错题归纳Agent → 错题分类 + AI错因分析
                                 ↓
                      学习规划Agent → 生成个性化学习计划
```

### 7.2 Agent 职责

| Agent | 输入 | 输出 | 触发时机 |
|-------|------|------|----------|
| 教材解析Agent | 教材 PDF 文本 | 章节结构 + 知识点列表 | 教材上传后 |
| 题目解析Agent | 题目内容 | 知识点绑定 + 难度评估 | 题目创建时 |
| 学情分析Agent | 练习答题记录 | 知识掌握度更新 | 练习结束后 |
| 错题归纳Agent | 错误答案 + 正确答案 | 错因分类 + 分析解释 | 答错题时 |
| 学习规划Agent | 掌握度 + 错题 + 学科 | 学习计划（日/周） | 手动触发 |

### 7.3 掌握度计算

工程简化版本（Phase 1 使用，Phase 4 升级为 BKT）：

```python
mastery = (correct_count / attempt_count) × difficulty_weight × time_decay

# difficulty_weight: 难题答对权重更高
# time_decay: 长时间未练习掌握度衰减
```

---

## 8. 页面设计

| 页面 | 路由 | 功能描述 |
|------|------|----------|
| **学习仪表盘** | `/` | 今日学习计划、各学科掌握度概览、最近错题、练习统计 |
| **教材列表** | `/textbook` | 教材 CRUD、上传 PDF |
| **教材上传** | `/textbook/upload` | PDF 上传 + AI 解析触发 |
| **教材详情** | `/textbook/:id` | 章节树 + 知识点列表 |
| **题库列表** | `/question` | 题目列表（分页/筛选：学科/类型/难度） |
| **创建题目** | `/question/create` | 题目录入表单（支持 Markdown + 数学公式） |
| **题目详情** | `/question/:id` | 题目内容 + 答案 + 解析 + 关联知识点 |
| **练习入口** | `/practice` | 选择学科/章节/模式，开始练习 |
| **做题页面** | `/practice/:session` | 逐题作答，实时提交 |
| **错题本** | `/wrong-question` | 错题列表（筛选：学科/错因类型/是否掌握） |
| **错题复习** | `/wrong-question/review` | 错题重做 + AI 分析展示 |
| **学习计划** | `/learning-plan` | 计划列表 + AI 生成入口 |
| **计划详情** | `/learning-plan/:id` | 计划内容、进度跟踪 |
| **学情分析** | `/analysis` | 掌握度雷达图、趋势折线图、薄弱点列表 |
| **系统设置** | `/settings` | AI API 配置、学科管理 |

---

## 9. 环境变量

### 后端 `.env.development`

```env
APP_NAME=AIEDU
APP_ENV=development
APP_PORT=5000
DATABASE_URL=sqlite://data/aiedu.db
AI_API_KEY=your_deepseek_api_key
AI_API_BASE_URL=https://api.deepseek.com
AI_MODEL=deepseek-chat
CORS_ORIGINS=http://localhost:3000
LOG_LEVEL=DEBUG
```

### 前端 `.env.development`

```env
VITE_APP_TITLE=AIEDU智能学习助手
VITE_API_BASE_URL=http://localhost:5000/api/v1
```

---

## 10. 开发命令

### 后端

| 命令 | 说明 |
|------|------|
| `uv pip install -r requirements.txt` | 安装依赖 |
| `python main.py` | 启动开发服务（热重载） |
| `python scripts/init_db.py` | 初始化数据库 |
| `python scripts/seed_data.py` | 导入测试数据 |
| `pytest` | 运行测试 |

### 前端

| 命令 | 说明 |
|------|------|
| `pnpm install` | 安装依赖 |
| `pnpm run dev` | 启动开发服务（端口:3000） |
| `pnpm run build` | 构建前端 |
| `pnpm run lint` | ESLint 检查 |

---

## 11. 初始化数据

- 预置学科数据：数学、物理、化学、生物、语文、英语、历史、地理、政治
- 预置示例教材（可选）
- 预置示例题目（可选，用于快速体验）

---

## 12. 开发任务清单

| 序号 | 模块 | 工作内容 | 优先级 |
|------|------|----------|--------|
| 1 | 基础框架 | 前后端项目初始化、目录结构、基础配置 | P0 |
| 2 | 核心中间件 | database + middleware + logger + response | P0 |
| 3 | 学科管理 | 学科 CRUD（前后端） | P0 |
| 4 | 教材管理 | 教材 CRUD + PDF 上传 | P0 |
| 5 | 教材解析 | 教材解析Agent + PDF解析工具 | P0 |
| 6 | 知识点 | 知识点树展示（解析结果） | P0 |
| 7 | 题库 | 题目 CRUD + 知识点绑定 | P0 |
| 8 | 题目解析 | 题目解析Agent（自动绑定知识点） | P1 |
| 9 | 练习系统 | 练习模式选择 + 做题 + 提交 | P0 |
| 10 | 错题系统 | 自动收集错题 + 列表展示 | P0 |
| 11 | 错题分析 | 错题归纳Agent + AI错因分析 | P1 |
| 12 | 掌握度 | 掌握度计算 + 更新逻辑 | P1 |
| 13 | 学情分析 | 掌握度雷达图 + 趋势图 + 弱项 | P1 |
| 14 | 学习计划 | 学习规划Agent + 计划展示 | P1 |
| 15 | 仪表盘 | 学习仪表盘（汇总信息） | P1 |
| 16 | 设置 | AI 配置页面 | P2 |

---

## 13. 注意事项

- 本阶段无登录鉴权，所有接口直接访问
- 数据库表设计需预留 `user_id / student_id` 字段空间，便于 Phase 2 扩展（但本阶段不使用）
- 务必确保 UnoCSS 样式在 Naive UI 样式之后引入
- 使用 `@unocss/reset/tailwind-compat.css` 修正样式
- 前端统一使用 `@/utils/request` 封装的 Axios 实例
- 后端所有输入输出必须使用 Pydantic schemas
- 中间件统一放到 `core/`
- 支持热启动（Uvicorn --reload / Vite HMR）

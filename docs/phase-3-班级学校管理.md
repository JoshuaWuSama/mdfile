# Phase 3 - 班级学校管理

> 基于 Phase 2，增加学校/班级组织架构、统一作业考试管理、班级学情对比、教师教学分析

---

## 1. 阶段定位

将个人学习工具升级为 **学校级教学管理平台**：

- 学校 → 年级 → 班级 → 学生 组织架构
- 教师关联班级与学科
- 教师统一发布作业、管理考试
- 班级/学校维度的学情横向对比
- 教师教学效果分析
- 成绩录入与导出
- 学生批量管理

---

## 2. 系统架构（在 Phase 2 基础上扩展）

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
         ┌───────────────────┴───────────────────┐
         │                                       │
  ┌──────▼────────┐                    ┌────────▼────────┐
  │  业务服务层    │                    │   AI Agent层    │
  │  FastAPI      │                    │   OpenClaw      │
  │               │                    │                 │
  │ 认证系统       │                    │ 教材解析Agent   │
  │ 用户管理       │                    │ 题目解析Agent   │
  │ ★ 学校管理    │                    │ 学情分析Agent   │
  │ ★ 班级管理    │                    │ 错题归纳Agent   │
  │ ★ 学生管理    │                    │ 学习规划Agent   │
  │ ★ 教师管理    │                    │                 │
  │ 教材/题库     │                    │                 │
  │ ★ 作业系统    │                    │                 │
  │ ★ 考试系统    │                    │                 │
  │ 错题系统       │                    │                 │
  │ 学习计划       │                    │                 │
  │ ★ 班级学情    │                    │                 │
  │ ★ 横向对比    │                    │                 │
  │ ★ 教学分析    │                    │                 │
  └──────┬────────┘                    └────────┬────────┘
         │                                      │
  ┌──────▼────────┐                    ┌────────▼────────┐
  │   数据层       │                    │   AI 能力层     │
  │ SQLite        │                    │ LLM (DeepSeek)  │
  │ Tortoise-ORM  │                    │ httpx           │
  └───────────────┘                    └─────────────────┘

★ = Phase 3 新增模块
```

---

## 3. 新增技术栈

在 Phase 2 基础上新增：

| 分类 | 技术 | 说明 |
|------|------|------|
| 拖拽排序 | vuedraggable / sortablejs | 题目排序、学习计划调整 |
| 批量导入 | xlsx（前端解析） | 学生批量导入 |
| 定时任务 | APScheduler | 作业截止提醒、定时报告生成 |

---

## 4. 新增/变更目录结构

### 4.1 前端新增

```
frontend/src/
├── api/
│   ├── school.ts                  # ★ 学校管理接口
│   ├── classroom.ts               # ★ 班级管理接口
│   ├── student.ts                 # ★ 学生管理接口
│   ├── teacher.ts                 # ★ 教师管理接口
│   ├── homework.ts                # ★ 作业管理接口
│   └── exam.ts                    # ★ 考试管理接口
├── components/
│   ├── StudentRank.vue            # ★ 学生排名组件
│   ├── ClassCompare.vue           # ★ 班级对比组件
│   ├── ExportButton.vue           # ★ 通用导出按钮
│   └── BatchImport.vue            # ★ 批量导入组件
├── composables/
│   └── useClassroom.ts            # ★ 班级相关 hook
├── stores/
│   └── school.ts                  # ★ 学校/班级状态
├── views/
│   ├── school/
│   │   ├── index.vue              # ★ 学校管理
│   │   └── detail.vue             # ★ 学校详情
│   ├── classroom/
│   │   ├── index.vue              # ★ 班级列表
│   │   └── detail.vue             # ★ 班级详情（学生列表）
│   ├── student/
│   │   ├── index.vue              # ★ 学生管理列表
│   │   ├── detail.vue             # ★ 学生详情/学情
│   │   └── import.vue             # ★ 批量导入学生
│   ├── teacher/
│   │   ├── index.vue              # ★ 教师列表
│   │   └── detail.vue             # ★ 教师详情
│   ├── homework/
│   │   ├── index.vue              # ★ 作业列表
│   │   ├── create.vue             # ★ 发布作业（选题组卷）
│   │   ├── detail.vue             # ★ 作业详情/批改
│   │   └── submit.vue             # ★ 学生提交作业
│   ├── exam/
│   │   ├── index.vue              # ★ 考试列表
│   │   ├── create.vue             # ★ 创建考试/录入成绩
│   │   └── result.vue             # ★ 考试成绩分析
│   └── analysis/
│       ├── class-report.vue       # ★ 班级学情报告
│       ├── compare.vue            # ★ 横向对比（班级间）
│       └── teaching.vue           # ★ 教学效果分析
```

### 4.2 后端新增

```
backend/
├── models/
│   ├── school.py                  # ★ 学校模型
│   ├── classroom.py               # ★ 班级模型
│   ├── student.py                 # ★ 学生模型
│   ├── teacher.py                 # ★ 教师模型
│   ├── teacher_class_subject.py   # ★ 教师-班级-学科关联
│   ├── homework.py                # ★ 作业模型
│   ├── homework_question.py       # ★ 作业题目关联
│   ├── submission.py              # ★ 作业提交模型
│   ├── submission_answer.py       # ★ 提交答案明细
│   ├── exam.py                    # ★ 考试模型
│   ├── exam_question.py           # ★ 考试题目关联
│   ├── exam_result.py             # ★ 考试成绩
│   └── exam_answer.py             # ★ 考试答题明细
├── schemas/
│   ├── school.py                  # ★
│   ├── classroom.py               # ★
│   ├── student.py                 # ★
│   ├── teacher.py                 # ★
│   ├── homework.py                # ★
│   └── exam.py                    # ★
├── routes/
│   ├── school.py                  # ★
│   ├── classroom.py               # ★
│   ├── student.py                 # ★
│   ├── teacher.py                 # ★
│   ├── homework.py                # ★
│   └── exam.py                    # ★
├── services/
│   ├── school_service.py          # ★
│   ├── classroom_service.py       # ★
│   ├── student_service.py         # ★
│   ├── teacher_service.py         # ★
│   ├── homework_service.py        # ★
│   └── exam_service.py            # ★
```

---

## 5. 数据库变更

### 5.1 新增表

#### 学校表

```sql
school
├── id              INTEGER PRIMARY KEY
├── name            VARCHAR(100) NOT NULL
├── region          VARCHAR(100)
├── address         VARCHAR(255)
├── contact_phone   VARCHAR(20)
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 班级表

```sql
classroom
├── id              INTEGER PRIMARY KEY
├── school_id       INTEGER FK → school.id
├── grade           VARCHAR(20) NOT NULL       -- 七年级、八年级...
├── name            VARCHAR(50) NOT NULL       -- 1班、2班...
├── academic_year   VARCHAR(20)                -- 2025-2026
├── head_teacher_id INTEGER FK → user.id
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 学生表

```sql
student
├── id              INTEGER PRIMARY KEY
├── user_id         INTEGER FK → user.id
├── class_id        INTEGER FK → classroom.id
├── student_no      VARCHAR(50) UNIQUE         -- 学号
├── name            VARCHAR(50) NOT NULL
├── gender          VARCHAR(10)
├── enrollment_year INTEGER
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 教师表

```sql
teacher
├── id              INTEGER PRIMARY KEY
├── user_id         INTEGER FK → user.id
├── school_id       INTEGER FK → school.id
├── name            VARCHAR(50) NOT NULL
├── employee_no     VARCHAR(50) UNIQUE         -- 工号
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 教师-班级-学科关联表

```sql
teacher_class_subject
├── id              INTEGER PRIMARY KEY
├── teacher_id      INTEGER FK → teacher.id
├── class_id        INTEGER FK → classroom.id
├── subject_id      INTEGER FK → subject.id
├── academic_year   VARCHAR(20)
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 作业表

```sql
homework
├── id              INTEGER PRIMARY KEY
├── class_id        INTEGER FK → classroom.id
├── subject_id      INTEGER FK → subject.id
├── title           VARCHAR(200) NOT NULL
├── description     TEXT
├── teacher_id      INTEGER FK → teacher.id
├── publish_time    DATETIME
├── deadline        DATETIME
├── status          VARCHAR(20) DEFAULT 'draft'   -- draft/published/closed
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 作业-题目关联表

```sql
homework_question
├── id              INTEGER PRIMARY KEY
├── homework_id     INTEGER FK → homework.id
├── question_id     INTEGER FK → question.id
├── order_index     INTEGER
├── score           FLOAT DEFAULT 0
└── created_at      DATETIME
```

#### 作业提交表

```sql
submission
├── id              INTEGER PRIMARY KEY
├── homework_id     INTEGER FK → homework.id
├── student_id      INTEGER FK → student.id
├── submit_time     DATETIME
├── total_score     FLOAT
├── status          VARCHAR(20) DEFAULT 'pending'  -- pending/graded
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 提交答案明细表

```sql
submission_answer
├── id              INTEGER PRIMARY KEY
├── submission_id   INTEGER FK → submission.id
├── question_id     INTEGER FK → question.id
├── answer          TEXT
├── is_correct      BOOLEAN
├── score           FLOAT DEFAULT 0
├── time_spent      INTEGER                    -- 耗时（秒）
├── ai_feedback     TEXT                       -- AI 反馈
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 考试表

```sql
exam
├── id              INTEGER PRIMARY KEY
├── class_id        INTEGER FK → classroom.id
├── subject_id      INTEGER FK → subject.id
├── title           VARCHAR(200) NOT NULL
├── exam_type       VARCHAR(20)                -- unit/midterm/final/mock
├── total_score     FLOAT DEFAULT 100
├── exam_date       DATETIME
├── teacher_id      INTEGER FK → teacher.id
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 考试-题目关联表

```sql
exam_question
├── id              INTEGER PRIMARY KEY
├── exam_id         INTEGER FK → exam.id
├── question_id     INTEGER FK → question.id
├── order_index     INTEGER
├── score           FLOAT
└── created_at      DATETIME
```

#### 考试成绩表

```sql
exam_result
├── id              INTEGER PRIMARY KEY
├── exam_id         INTEGER FK → exam.id
├── student_id      INTEGER FK → student.id
├── total_score     FLOAT
├── rank            INTEGER
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 考试答题明细表

```sql
exam_answer
├── id              INTEGER PRIMARY KEY
├── exam_result_id  INTEGER FK → exam_result.id
├── question_id     INTEGER FK → question.id
├── answer          TEXT
├── is_correct      BOOLEAN
├── score           FLOAT DEFAULT 0
├── created_at      DATETIME
└── updated_at      DATETIME
```

### 5.2 变更表

Phase 2 的 `user` 表增加字段：

```sql
user + school_id    INTEGER FK → school.id   -- 所属学校（可空）
```

Phase 1 的 `wrong_question` 表扩展 `source_type`：

```sql
wrong_question.source_type  -- 增加 'homework' / 'exam' 值
```

Phase 1 的 `student_knowledge_state` 表关联学生：

```sql
student_knowledge_state + student_id  INTEGER FK → student.id  -- 替代 user_id
```

---

## 6. 新增接口

### 6.1 学校管理 `/api/v1/schools`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/schools` | 学校列表 | admin |
| POST | `/api/v1/schools` | 创建学校 | admin |
| GET | `/api/v1/schools/{id}` | 学校详情 | admin / school_admin |
| PUT | `/api/v1/schools/{id}` | 更新学校 | admin |
| DELETE | `/api/v1/schools/{id}` | 删除学校 | admin |

### 6.2 班级管理 `/api/v1/classes`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/classes?school_id=&grade=` | 班级列表 | teacher / admin |
| POST | `/api/v1/classes` | 创建班级 | admin |
| GET | `/api/v1/classes/{id}` | 班级详情 | teacher / admin |
| PUT | `/api/v1/classes/{id}` | 更新班级 | admin |
| DELETE | `/api/v1/classes/{id}` | 删除班级 | admin |
| GET | `/api/v1/classes/{id}/students` | 班级学生列表 | teacher / admin |

### 6.3 学生管理 `/api/v1/students`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/students?class_id=&name=&page=` | 学生列表 | teacher / admin |
| POST | `/api/v1/students` | 创建学生 | admin |
| POST | `/api/v1/students/batch-import` | 批量导入学生（Excel） | admin |
| GET | `/api/v1/students/{id}` | 学生详情 | teacher / admin |
| PUT | `/api/v1/students/{id}` | 更新学生 | admin |
| DELETE | `/api/v1/students/{id}` | 删除学生 | admin |
| GET | `/api/v1/students/{id}/analysis` | 学生学情报告 | teacher / admin / parent |
| GET | `/api/v1/students/export?class_id=` | 导出学生名单 | admin |

### 6.4 教师管理 `/api/v1/teachers`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/teachers?school_id=` | 教师列表 | admin |
| POST | `/api/v1/teachers` | 创建教师 | admin |
| GET | `/api/v1/teachers/{id}` | 教师详情 | admin |
| PUT | `/api/v1/teachers/{id}` | 更新教师 | admin |
| DELETE | `/api/v1/teachers/{id}` | 删除教师 | admin |
| POST | `/api/v1/teachers/{id}/assign` | 分配班级学科 | admin |
| GET | `/api/v1/teachers/my-classes` | 我的班级列表 | teacher |

### 6.5 作业管理 `/api/v1/homework`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/homework?class_id=&status=` | 作业列表 | teacher / student |
| POST | `/api/v1/homework` | 创建作业（选题组卷） | teacher |
| GET | `/api/v1/homework/{id}` | 作业详情 | teacher / student |
| PUT | `/api/v1/homework/{id}` | 更新作业 | teacher |
| POST | `/api/v1/homework/{id}/publish` | 发布作业 | teacher |
| POST | `/api/v1/homework/{id}/close` | 关闭作业 | teacher |
| DELETE | `/api/v1/homework/{id}` | 删除作业（仅草稿） | teacher |
| GET | `/api/v1/homework/{id}/submissions` | 提交情况一览 | teacher |
| POST | `/api/v1/homework/{id}/submit` | 学生提交作业 | student |
| GET | `/api/v1/homework/{id}/my-submission` | 我的提交详情 | student |
| POST | `/api/v1/homework/{id}/grade` | 批改作业 | teacher |
| GET | `/api/v1/homework/{id}/statistics` | 作业统计分析 | teacher |
| GET | `/api/v1/homework/export?class_id=` | 导出作业成绩 | teacher |

### 6.6 考试管理 `/api/v1/exams`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/exams?class_id=&type=` | 考试列表 | teacher / student |
| POST | `/api/v1/exams` | 创建考试 | teacher |
| GET | `/api/v1/exams/{id}` | 考试详情 | teacher / student |
| PUT | `/api/v1/exams/{id}` | 更新考试 | teacher |
| DELETE | `/api/v1/exams/{id}` | 删除考试 | teacher |
| POST | `/api/v1/exams/{id}/results` | 批量录入成绩 | teacher |
| POST | `/api/v1/exams/{id}/results/import` | Excel 导入成绩 | teacher |
| GET | `/api/v1/exams/{id}/results` | 成绩列表 | teacher |
| GET | `/api/v1/exams/{id}/my-result` | 我的考试成绩 | student |
| GET | `/api/v1/exams/{id}/statistics` | 考试统计分析 | teacher |
| GET | `/api/v1/exams/{id}/export` | 导出成绩单 | teacher |

### 6.7 班级学情分析 `/api/v1/analysis`（扩展）

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/analysis/class/{class_id}/overview` | 班级学情概览 | teacher |
| GET | `/api/v1/analysis/class/{class_id}/mastery` | 班级知识掌握分布 | teacher |
| GET | `/api/v1/analysis/class/{class_id}/wrong-stats` | 班级高频错题 | teacher |
| GET | `/api/v1/analysis/class/{class_id}/ranking` | 班级学生排名 | teacher |
| GET | `/api/v1/analysis/compare?class_ids=1,2,3` | 班级间横向对比 | teacher / admin |
| GET | `/api/v1/analysis/teaching/{teacher_id}` | 教学效果分析 | admin |
| GET | `/api/v1/analysis/school/{school_id}/overview` | 学校学情总览 | admin |
| GET | `/api/v1/analysis/class/{class_id}/export` | 导出班级报告 | teacher |

---

## 7. 角色权限扩展

Phase 2 的角色增加 `school_admin`：

| 角色 | 数据范围 |
|------|----------|
| **admin** | 全局所有数据 |
| **school_admin** | 本校所有数据 |
| **teacher** | 所授班级的学生/作业/考试/学情 |
| **student** | 自己的数据 + 所在班级公共数据 |
| **parent** | 绑定子女的数据 |

### 数据隔离规则

```
teacher 查询作业 → 只返回自己创建的作业 或 自己所授班级的作业
student 查询作业 → 只返回自己所在班级的已发布作业
teacher 查询学生 → 只返回自己所授班级的学生
admin 查询所有  → 不限制
school_admin    → 限制为本校数据
```

---

## 8. 页面设计

### 8.1 新增页面

| 页面 | 路由 | 角色 | 功能 |
|------|------|------|------|
| 学校管理 | `/school` | admin | 学校 CRUD |
| 学校详情 | `/school/:id` | admin | 学校信息、班级列表 |
| 班级列表 | `/classroom` | teacher/admin | 班级列表 |
| 班级详情 | `/classroom/:id` | teacher/admin | 学生列表、班级学情 |
| 学生管理 | `/student` | teacher/admin | 学生 CRUD + 批量导入 |
| 学生详情 | `/student/:id` | teacher/admin | 个人信息 + 学情报告 |
| 学生导入 | `/student/import` | admin | Excel 批量导入 |
| 教师管理 | `/teacher` | admin | 教师 CRUD + 班级分配 |
| 教师详情 | `/teacher/:id` | admin | 教师信息 + 所授班级 |
| 作业列表 | `/homework` | teacher/student | 教师：管理作业；学生：查看作业 |
| 发布作业 | `/homework/create` | teacher | 选择班级 → 选题 → 设置截止时间 |
| 作业详情 | `/homework/:id` | teacher/student | 教师：提交情况+批改；学生：答题 |
| 提交作业 | `/homework/:id/submit` | student | 在线答题提交 |
| 考试列表 | `/exam` | teacher/student | 考试管理 |
| 创建考试 | `/exam/create` | teacher | 创建考试 + 录入题目 |
| 考试成绩 | `/exam/:id/result` | teacher/student | 成绩录入/查看 + 统计 |
| 班级学情 | `/analysis/class/:id` | teacher | 班级知识掌握分布图 |
| 横向对比 | `/analysis/compare` | teacher/admin | 多班级学情雷达对比 |
| 教学分析 | `/analysis/teaching` | admin | 教师教学效果分析 |

### 8.2 改造页面

| 页面 | 改造内容 |
|------|----------|
| 教师仪表盘 | 增加：待批改作业、班级学情概览、最近考试比 |
| 学生仪表盘 | 增加：待完成作业、考试成绩、班级排名 |
| 管理员仪表盘 | 增加：学校统计、教师教学概览 |
| 家长看板 | 增加：子女作业完成情况、考试成绩趋势 |
| 错题本 | 错题来源增加「作业」「考试」标识 |
| 练习系统 | Phase 1 的自主练习保留不变 |

---

## 9. 业务流程

### 9.1 作业流程

```
教师端                          学生端
  │                               │
  ├─ 创建作业（选班级）           │
  ├─ 从题库选题/手动录题          │
  ├─ 设置截止时间                 │
  ├─ 发布作业 ──────────────────→ 收到作业通知
  │                               ├─ 查看题目
  │                               ├─ 在线答题
  │                               ├─ 提交作业
  ├─ 查看提交情况 ←───────────── 提交完成
  ├─ 批改/AI辅助批改              │
  ├─ 发布成绩 ──────────────────→ 查看批改结果
  │                               ├─ 错题自动归入错题本
  │                               └─ 知识掌握度自动更新
  └─ 查看作业统计
```

### 9.2 考试流程

```
教师端                          学生端
  │                               │
  ├─ 创建考试（选班级）           │
  ├─ 关联题目（可选）             │
  ├─ 设置考试日期                 │
  │                               │
  │  ← 线下考试 →                │
  │                               │
  ├─ 录入成绩（手动/Excel导入）   │
  ├─ 录入答题明细（可选）         │
  ├─ 发布成绩 ──────────────────→ 查看考试成绩 + 排名
  │                               ├─ 错题自动归入错题本
  │                               └─ 知识掌握度自动更新
  └─ 查看考试统计分析
```

---

## 10. 环境变量新增

### 后端 `.env.development` 新增

```env
# 定时任务
SCHEDULER_ENABLED=true
HOMEWORK_REMINDER_HOURS=2          # 作业截止前提醒（小时）
```

---

## 11. 初始化数据

在 Phase 2 基础上新增：

- 创建示例学校
- 创建示例班级（2个年级 × 2个班）
- 创建示例学生（每班 5 名）
- 创建示例教师（3名，分配班级）
- 创建示例作业（1份已发布）
- 创建示例考试（1次已完成）

---

## 12. 开发任务清单

| 序号 | 模块 | 工作内容 | 优先级 |
|------|------|----------|--------|
| 1 | 学校管理 | 学校 CRUD（后端 + 前端） | P0 |
| 2 | 班级管理 | 班级 CRUD + 学生列表 | P0 |
| 3 | 学生管理 | 学生 CRUD + 批量导入 | P0 |
| 4 | 教师管理 | 教师 CRUD + 班级分配 | P0 |
| 5 | 角色扩展 | 增加 school_admin 角色 + 数据隔离 | P0 |
| 6 | 作业系统 | 创建/发布/提交/批改 全流程 | P0 |
| 7 | 考试系统 | 创建/成绩录入/导入 | P0 |
| 8 | 作业做题 | 学生在线答题页面 | P0 |
| 9 | 错题扩展 | 作业/考试错题自动归入错题本 | P1 |
| 10 | 掌握度扩展 | 作业/考试答题后自动更新掌握度 | P1 |
| 11 | 班级学情 | 班级知识掌握分布图 | P1 |
| 12 | 横向对比 | 班级间学情雷达对比 | P1 |
| 13 | 教学分析 | 教师教学效果分析页面 | P2 |
| 14 | 成绩导出 | 作业/考试成绩 Excel 导出 | P2 |
| 15 | 仪表盘改造 | 各角色仪表盘适配班级/作业/考试数据 | P1 |
| 16 | 定时任务 | 作业截止提醒 | P2 |

---

## 13. 注意事项

- 数据隔离是本阶段核心难点，所有查询需基于当前用户角色过滤数据
- 教师只能操作自己所授班级的数据
- 作业提交答案后应自动触发学情分析Agent和错题归纳Agent
- 考试成绩录入后同样触发学情更新
- 批量导入学生需处理数据校验与重复检测
- 作业截止后自动关闭，不再接受提交
- 前后端字段校验保持一致（成绩范围、截止时间等）

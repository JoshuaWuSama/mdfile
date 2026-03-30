# Phase 2 - 鉴权与多角色

> 基于 Phase 1，增加用户认证体系、多角色权限、知识点管理、统计分析与数据导出

---

## 1. 阶段定位

在个人学习 OS 基础上，引入 **多用户体系**：

- JWT 认证系统（登录/注册/Token刷新）
- RBAC 角色权限（学生/教师/家长/管理员）
- 教师可管理知识点、编辑题库
- 家长可查看学生学情
- 管理员维护系统数据
- 统计仪表盘（不同角色看到不同数据）
- 数据导出（Excel）

---

## 2. 系统架构（在 Phase 1 基础上扩展）

```
              ┌───────────────────────────────┐
              │          前端层                │
              │    Vue3 + NaiveUI + Vite      │
              │                               │
              │ 学生端 / 教师端 / 家长端 / 管理端│
              └──────────────┬────────────────┘
                             │
                       Axios (JWT Token)
                             │
         ┌───────────────────┴───────────────────┐
         │                                       │
  ┌──────▼────────┐                    ┌────────▼────────┐
  │  业务服务层    │                    │   AI Agent层    │
  │  FastAPI      │                    │   OpenClaw      │
  │               │                    │                 │
  │ ★ 认证系统    │                    │ 教材解析Agent   │
  │ ★ 用户管理    │                    │ 题目解析Agent   │
  │ ★ 角色权限    │                    │ 学情分析Agent   │
  │ 教材管理       │                    │ 错题归纳Agent   │
  │ 题库管理       │                    │ 学习规划Agent   │
  │ 练习系统       │                    │                 │
  │ 错题系统       │                    │                 │
  │ ★ 知识点管理  │                    │                 │
  │ 学情分析       │                    │                 │
  │ ★ 统计仪表盘  │                    │                 │
  │ ★ 数据导出    │                    │                 │
  └──────┬────────┘                    └────────┬────────┘
         │                                      │
  ┌──────▼────────┐                    ┌────────▼────────┐
  │   数据层       │                    │   AI 能力层     │
  │ SQLite        │                    │ LLM (DeepSeek)  │
  │ Tortoise-ORM  │                    │ httpx           │
  └───────────────┘                    └─────────────────┘

★ = Phase 2 新增模块
```

---

## 3. 新增技术栈

在 Phase 1 基础上新增：

### 3.1 后端新增

| 分类 | 技术 | 说明 |
|------|------|------|
| JWT | PyJWT | Token 生成与验证 |
| 密码 | passlib[bcrypt] | 密码哈希 |
| 数据导出 | pandas + openpyxl | Excel 导出 |
| API 限流 | slowapi | 登录接口限流 |

### 3.2 前端新增

| 分类 | 技术 | 说明 |
|------|------|------|
| 导出 | xlsx + file-saver | Excel 导出 |
| 虚拟滚动 | vue-virtual-scroller | 大数据列表 |

---

## 4. 新增/变更目录结构

### 4.1 前端新增（相对 Phase 1）

```
frontend/src/
├── api/
│   ├── auth.ts                    # ★ 认证接口
│   └── user.ts                    # ★ 用户管理接口
├── composables/
│   ├── useAuth.ts                 # ★ 认证 hook
│   └── useExport.ts               # ★ 导出 hook
├── directives/
│   └── permission.ts              # ★ 权限指令 v-permission
├── layouts/
│   └── AuthLayout.vue             # ★ 登录布局
├── stores/
│   └── user.ts                    # ★ 用户状态（Token/角色）
├── utils/
│   ├── auth.ts                    # ★ Token 管理（localStorage）
│   └── export.ts                  # ★ 导出工具
├── views/
│   ├── auth/
│   │   └── login/
│   │       └── index.vue          # ★ 登录页
│   ├── dashboard/
│   │   └── index.vue              # ★ 改造：按角色展示不同仪表盘
│   ├── knowledge/
│   │   ├── index.vue              # ★ 知识点管理（树形CRUD）
│   │   └── detail.vue             # ★ 知识点详情/编辑
│   ├── system/
│   │   ├── user/
│   │   │   └── index.vue          # ★ 用户管理
│   │   └── profile/
│   │       └── index.vue          # ★ 个人信息
│   └── parent/
│       └── index.vue              # ★ 家长查看学情
```

### 4.2 后端新增（相对 Phase 1）

```
backend/
├── core/
│   └── auth.py                    # ★ JWT 认证中间件 + 权限守卫
├── models/
│   └── user.py                    # ★ 用户模型
├── schemas/
│   └── user.py                    # ★ 用户 Schema
├── routes/
│   ├── auth.py                    # ★ 认证路由
│   └── user.py                    # ★ 用户管理路由
├── services/
│   ├── auth_service.py            # ★ 认证业务
│   └── user_service.py            # ★ 用户业务
├── utils/
│   └── export.py                  # ★ 数据导出工具
```

---

## 5. 数据库变更

### 5.1 新增表

#### 用户表

```sql
user
├── id              INTEGER PRIMARY KEY
├── username        VARCHAR(50) UNIQUE NOT NULL
├── password_hash   VARCHAR(255) NOT NULL
├── real_name       VARCHAR(50)
├── role            VARCHAR(20) NOT NULL       -- admin / teacher / student / parent
├── email           VARCHAR(100)
├── phone           VARCHAR(20)
├── avatar          VARCHAR(500)
├── is_active       BOOLEAN DEFAULT TRUE
├── last_login      DATETIME
├── created_at      DATETIME
└── updated_at      DATETIME
```

#### 家长-学生关联表

```sql
parent_student
├── id              INTEGER PRIMARY KEY
├── parent_id       INTEGER FK → user.id
├── student_id      INTEGER FK → user.id       -- role=student 的用户
├── relationship    VARCHAR(20)                -- father/mother/guardian
├── created_at      DATETIME
└── updated_at      DATETIME
```

### 5.2 变更表

以下 Phase 1 表新增 `user_id` 字段（关联操作人）：

```
practice_session    + user_id    INTEGER FK → user.id
wrong_question      + user_id    INTEGER FK → user.id
student_knowledge_state + user_id INTEGER FK → user.id
learning_plan       + user_id    INTEGER FK → user.id
ai_analysis_log     + user_id    INTEGER FK → user.id
question            + created_by INTEGER FK → user.id  （已预留）
```

### 5.3 新增表 - 知识点管理增强

```sql
-- 知识点变更日志（教师编辑记录）
knowledge_edit_log
├── id              INTEGER PRIMARY KEY
├── knowledge_id    INTEGER FK → knowledge_node.id
├── user_id         INTEGER FK → user.id
├── action          VARCHAR(20)               -- create/update/delete
├── old_data        JSON
├── new_data        JSON
├── created_at      DATETIME
└── updated_at      DATETIME
```

---

## 6. 新增接口

### 6.1 认证 `/api/v1/auth`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| POST | `/api/v1/auth/login` | 登录 | 公开 |
| POST | `/api/v1/auth/register` | 注册 | 公开 |
| POST | `/api/v1/auth/refresh` | 刷新 Token | 需登录 |
| POST | `/api/v1/auth/logout` | 登出 | 需登录 |
| GET | `/api/v1/auth/me` | 当前用户信息 | 需登录 |

### 6.2 用户管理 `/api/v1/users`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/users?role=&page=&page_size=` | 用户列表 | admin |
| POST | `/api/v1/users` | 创建用户 | admin |
| GET | `/api/v1/users/{id}` | 用户详情 | admin |
| PUT | `/api/v1/users/{id}` | 更新用户 | admin |
| DELETE | `/api/v1/users/{id}` | 删除用户 | admin |
| PUT | `/api/v1/users/profile` | 更新个人信息 | 需登录 |
| PUT | `/api/v1/users/password` | 修改密码 | 需登录 |

### 6.3 知识点管理增强 `/api/v1/knowledge`

Phase 1 已有只读查询，Phase 2 新增教师维护能力：

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| POST | `/api/v1/knowledge` | 创建知识点 | teacher / admin |
| PUT | `/api/v1/knowledge/{id}` | 更新知识点 | teacher / admin |
| DELETE | `/api/v1/knowledge/{id}` | 删除知识点 | teacher / admin |
| PUT | `/api/v1/knowledge/{id}/move` | 移动知识点（调整父节点） | teacher / admin |
| GET | `/api/v1/knowledge/edit-log?knowledge_id=` | 编辑日志 | teacher / admin |

### 6.4 数据导出

在已有列表接口基础上，新增 `export` 端点：

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/questions/export?format=xlsx` | 导出题库 | teacher / admin |
| GET | `/api/v1/wrong-questions/export` | 导出错题 | student |
| GET | `/api/v1/analysis/export?subject_id=` | 导出学情报告 | student / teacher |
| GET | `/api/v1/practice/export?date_from=&date_to=` | 导出练习记录 | student |

### 6.5 统计仪表盘 `/api/v1/dashboard`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/dashboard/student` | 学生仪表盘数据 | student |
| GET | `/api/v1/dashboard/teacher` | 教师仪表盘数据 | teacher |
| GET | `/api/v1/dashboard/parent?student_id=` | 家长仪表盘数据 | parent |
| GET | `/api/v1/dashboard/admin` | 管理员仪表盘数据 | admin |

### 6.6 家长端 `/api/v1/parent`

| 方法 | 路径 | 说明 | 权限 |
|------|------|------|------|
| GET | `/api/v1/parent/children` | 获取绑定的学生列表 | parent |
| POST | `/api/v1/parent/bindchild` | 绑定子女（输入学生账号） | parent |
| GET | `/api/v1/parent/child/{student_id}/analysis` | 查看子女学情 | parent |
| GET | `/api/v1/parent/child/{student_id}/wrong-questions` | 查看子女错题 | parent |
| GET | `/api/v1/parent/child/{student_id}/plans` | 查看子女学习计划 | parent |

---

## 7. Phase 1 接口变更

所有 Phase 1 接口增加认证要求：

| 变更 | 说明 |
|------|------|
| 请求头 | 所有接口（除 auth）需携带 `Authorization: Bearer <token>` |
| 数据隔离 | 学生只能访问自己的练习/错题/学情数据 |
| 权限控制 | 题目创建/编辑权限细化（学生可创建个人题目，教师可管理公共题库） |
| 知识点 | GET 保持公开，CUD 操作需 teacher / admin 角色 |

---

## 8. 角色权限矩阵

| 功能 | admin | teacher | student | parent |
|------|-------|---------|---------|--------|
| 用户管理 | ✅ CRUD | ❌ | ❌ | ❌ |
| 学科管理 | ✅ CRUD | ✅ 查看 | ✅ 查看 | ❌ |
| 教材管理 | ✅ CRUD | ✅ CRUD | ✅ 查看 | ❌ |
| 知识点管理 | ✅ CRUD | ✅ CRUD | ✅ 查看 | ❌ |
| 公共题库 | ✅ CRUD | ✅ CRUD | ✅ 查看 | ❌ |
| 个人题目 | ✅ | ✅ | ✅ CRUD | ❌ |
| 练习做题 | ❌ | ❌ | ✅ | ❌ |
| 错题本 | ❌ | ✅ 查看学生的 | ✅ 自己的 | ✅ 查看子女 |
| 学情分析 | ✅ 全部 | ✅ 查看学生的 | ✅ 自己的 | ✅ 查看子女 |
| 学习计划 | ❌ | ✅ 查看学生的 | ✅ 自己的 | ✅ 查看子女 |
| 数据导出 | ✅ | ✅ | ✅ 自己的 | ✅ 子女的 |
| AI 配置 | ✅ | ❌ | ❌ | ❌ |

---

## 9. 页面设计

### 9.1 新增页面

| 页面 | 路由 | 角色 | 功能 |
|------|------|------|------|
| 登录 | `/login` | 公开 | 用户名密码登录 |
| 注册 | `/register` | 公开 | 账号注册（选择角色） |
| 用户管理 | `/system/user` | admin | 用户 CRUD + 角色分配 |
| 个人信息 | `/profile` | 全部 | 修改个人信息/密码 |
| 知识点管理 | `/knowledge` | teacher/admin | 知识点树 CRUD（拖拽排序） |
| 知识点详情 | `/knowledge/:id` | teacher/admin | 编辑知识点、查看关联题目 |
| 家长看板 | `/parent` | parent | 子女列表、学情概览 |

### 9.2 改造页面

| 页面 | 改造内容 |
|------|----------|
| 仪表盘 `/` | 根据角色展示不同内容（学生/教师/家长/管理员） |
| 题库 `/question` | 区分公共题库/个人题库，教师可管理公共题库 |
| 错题本 `/wrong-question` | 教师可查看指定学生错题 |
| 学情分析 `/analysis` | 教师可查看指定学生学情 |
| 所有列表页 | 增加导出按钮 |

### 9.3 路由守卫

```
- 未登录 → 重定向到 /login
- 登录后 → 根据角色重定向到对应仪表盘
- 无权限 → 显示 403 页面
- 动态菜单 → 根据角色过滤侧边栏菜单
```

---

## 10. 认证流程

```
登录请求
  │
  ▼
POST /api/v1/auth/login
  │ {username, password}
  ▼
验证密码（bcrypt）
  │
  ▼
生成 JWT（access_token + refresh_token）
  │
  ▼
前端存储 Token（localStorage）
  │
  ▼
后续请求自动携带 Authorization Header
  │
  ▼
后端中间件验证 Token → 注入当前用户
  │
  ▼
Token 过期 → 用 refresh_token 刷新
```

---

## 11. 环境变量新增

### 后端 `.env.development` 新增

```env
# JWT 配置
JWT_SECRET_KEY=随机生成的密钥
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE=30         # 分钟
JWT_REFRESH_TOKEN_EXPIRE=10080     # 分钟（7天）
```

---

## 12. 初始化数据

- 自动生成随机 JWT 密钥（init_db.py）
- 创建默认管理员账号：`admin / admin123`
- 创建测试教师账号：`teacher / teacher123`
- 创建测试学生账号：`student / student123`
- 创建测试家长账号：`parent / parent123`

---

## 13. 开发任务清单

| 序号 | 模块 | 工作内容 | 优先级 |
|------|------|----------|--------|
| 1 | 认证中间件 | core/auth.py（JWT 生成/验证/权限守卫） | P0 |
| 2 | 用户模型 | models/user.py + schemas/user.py | P0 |
| 3 | 认证接口 | 登录/注册/刷新/登出/当前用户 | P0 |
| 4 | 前端登录 | 登录页 + Token 管理 + 路由守卫 | P0 |
| 5 | 前端请求改造 | request.ts 增加 Token 注入 + 401 处理 | P0 |
| 6 | Phase 1 接口改造 | 所有接口增加认证 + 数据隔离 | P0 |
| 7 | 用户管理 | 用户 CRUD（后端 + 前端） | P1 |
| 8 | 角色权限 | 权限指令 + 动态菜单 + 路由守卫 | P1 |
| 9 | 知识点管理 | 知识点 CRUD 页面（教师端） | P1 |
| 10 | 家长端 | 家长绑定子女 + 查看学情 | P1 |
| 11 | 仪表盘改造 | 按角色展示不同仪表盘 | P1 |
| 12 | 数据导出 | 导出工具 + 各模块导出接口 | P2 |
| 13 | 个人信息 | 个人信息编辑 + 修改密码 | P2 |

---

## 14. 注意事项

- Phase 1 的所有数据表需迁移增加 `user_id` 字段
- 数据库迁移脚本需兼容已有数据
- 前端 `@/utils/request` 需增加 Token 自动注入和 401 拦截刷新逻辑
- 密码存储必须使用 bcrypt 哈希，禁止明文
- JWT 密钥通过环境变量配置，init_db.py 自动生成
- 前后端字段校验保持一致（用户名/密码长度等）

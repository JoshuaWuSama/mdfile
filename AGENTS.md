# AGENTS.md
## 0. 开发流程（必须遵循）

**开始任何代码开发前，必须按以下顺序执行：**

1. **生成方案文档** - 创建 `docs/方案文档.md`，包含：
   - 需求分析
   - 数据库设计（表结构、ER图）
   - 接口设计
   - 页面设计
   - 开发计划

2. **等待用户确认** - 方案文档必须经用户确认后，方可开始编码

3. **执行开发** - 按方案文档实施

⚠️ 违反此流程视为严重错误


## 1. **技术栈**:

- 前端：Vue3 + TypeScript + Vite + Pinia + Vue Router + Axios + vue-echarts + NaiveUI + PNPM +Day.js + UnoCSS +nprogress + @vicons/ionicons5 + eslint + 新版sass + @vueuse/motion + vue-virtual-scroller
- 后端：Python3.11+ + FastAPI + Tortoise-ORM最新版本 + Pydantic + PyJWT + loguru + Uvicorn
  - HTTP 客户端：httpx（异步请求）
  - 安全认证：passlib[bcrypt]（密码哈希）、python-multipart（文件上传）
  - 配置管理：python-dotenv（环境变量）
  - 数据处理：pandas、openpyxl（Excel 处理）
  - 测试框架：pytest、pytest-asyncio 
- 数据库：SQLite

## 2. 项目结构

### 前端

```
frontend
    src/
    ├── api/              # API 接口定义
    ├── assets/           # 静态资源（图片、字体等）
    ├── components/       # 公共组件
    ├── composables/      # 组合式函数（hooks）
    ├── layouts/          # 布局组件
    ├── router/           # 路由配置
    |—— directives        # 指令
    |—— styles            #公共样式
    ├── stores/           # 状态管理（Pinia）
    ├── types/            # TypeScript 类型定义
    ├── utils/            # 工具函数
    |——types/             # 全局公共类型
    └── views/            # 页面组件
            user/
              index.vue   #主页面
              detail.vue  #子页面
              edit.vue    #子页面
    dev.env               # 开发环境变量
    prod.env              # 生成环境变量
    public/               # 静态资源（不经构建）
    dist/                 # 构建产物
```

### 后端

```
backend
    main.py                # 应用入口
    config/               # 配置模块
    docs/                 # 文件文档
    data/                 # 数据库文件
    models/               # 数据模型
        user              # 用户模型
    core/                 # 中间件
        database          # 数据库中间件
        auth              # 认证中间件
        middleware        # 异常中间件
        logger            # 日志中间件
    routes/               # 路由模块
    services/             # 业务逻辑
    schemas/              # 数据验证模块
    utils/                # 工具函数
    logs/                 # 日志目录
    scripts/              # 脚本文件
    static/               # 前端构建目录
    dev.env               # 开发环境变量
    prod.env              # 生成环境变量
```

## 3. 开发命令

| 命令                                         | 说明         |
| -------------------------------------------- | ------------ |
| `uv pip install -r requirements.txt`            | 安装依赖     |
| `python main.py`                              | 启动开发服务 |
| `python init_db.py`                          | 初始化数据库 |
| `uvicorn app:main --host 0.0.0.0 --port 5000` | 生产环境启动 |

| 命令            | 说明         |
| --------------- | ------------ |
| `pnpm install`   | 安装依赖     |
| `pnpm run dev`   | 启动开发服务 |
| `pnpm run build` | 构建前端     |
| `端口:3000`     | 预览前端     |

## 4. 代码风格

- Python 文件使用 UTF-8 编码
- 遵循 PEP 8 规范
- 变量命名：snake_case
- 函数命名：snake_case
- 前端使用eslint 校验

## 5. 设计风格
- 后端输入，输出，响应 必须使用schemas(严格遵守)
- 功能模块根据模块分文件设计
- 前端页面组件根据子页面分文件
- 前端api 根据模块分文件
- 后端返回统一返回结构 
- 通用工具统一封装为工具调用
- 公共部分提取到通用文件中，比如utils
## 6.响应风格
    调用统一的响应schemas
    -成功响应
    {"code": 200, "message": "success", "data": {...}}

    -失败响应
    {"code": 400, "message": "错误信息", "data": null}

    - 后端接口RestFul 接口风格
## 7.UI风格
    - 表格选项颜色需要有不同的颜色
    - 按钮清晰简洁
## 8. 初始化数据
- 初始化生成随机jwt密钥
- 初始化导入测试数据[可选]
- 初始化生成用户基础数据
## 基础功能要求
- 前端报错友好显示输出
- 前后端字段校验保持一致
- 要求分页，排序，导出，查询筛选添加事件
- 验证页面逻辑

# 注意事项
- 需要支持热启动
- 开发完后进行前后端联调测试,确保页面和接口一致，特别是新增功能，和页面显示
- 前端@/utils/request
- 务必确保 UnoCSS 的样式在 Naive UI 的样式之后引入  
- @unocss/reset/tailwind-compat.css 进行修正
- 中间件统一放到core,比如常见的database,auth,middleware



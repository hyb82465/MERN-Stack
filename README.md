# MERN-Stack · 个人目标管理应用

一个跟随课程完成的 MERN 全栈练习项目（Goal Setter）。用户可以注册、登录，并管理自己的文字目标；React 前端通过 Express API 访问 MongoDB。

本仓库用于记录学习过程，主要练习前后端联调、JWT 身份认证、REST API 和 Redux 异步状态管理。以下说明依据仓库现有代码整理。

## 功能

- **账户**：姓名、邮箱和密码注册，邮箱密码登录，退出登录。
- **身份认证**：密码通过 bcryptjs 哈希存储；后端签发 JWT，受保护接口使用 Bearer Token。
- **目标管理**：登录后创建、查看和删除自己的目标，列表显示目标文本及创建时间。
- **数据归属**：后端按当前用户查询目标，并在更新、删除时检查目标所属用户。
- **交互**：登录状态持久化到 localStorage，使用加载状态及 Toast 消息反馈。

> 后端已提供更新目标的 `PUT` 接口，但当前前端没有编辑入口。目标数据只有所属用户、文本和时间戳，尚无完成状态、截止日期或分类。

## 技术栈

| 层次 | 主要技术 |
| --- | --- |
| 前端 | React 18、React Router 6、Redux Toolkit、React Redux |
| 请求与界面 | Axios、React Toastify、React Icons、CSS |
| 后端 | Node.js、Express 4、express-async-handler |
| 数据与认证 | MongoDB、Mongoose 8、jsonwebtoken、bcryptjs |
| 开发工具 | Create React App / react-scripts 5、nodemon、concurrently、dotenv |

具体依赖范围见根目录及 `frontend/` 下的 `package.json`，锁定版本见各自的 `package-lock.json`。

## 目录结构

```text
MERN-Stack/
├── backend/
│   ├── config/db.js         # MongoDB 连接
│   ├── controllers/         # 用户与目标业务逻辑
│   ├── middleware/          # JWT 认证、错误处理
│   ├── models/              # User、Goal 数据模型
│   ├── routes/              # 用户与目标 API 路由
│   └── server.js            # Express 入口及生产环境静态文件服务
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── app/store.js     # Redux store
│   │   ├── components/      # 目标表单、目标卡片、导航、加载组件
│   │   ├── features/        # auth / goals 的 slice 和 API service
│   │   └── pages/           # Dashboard、Login、Register
│   └── package.json
└── package.json             # 后端依赖及前后端联合启动脚本
```

## 本地运行

### 1. 准备环境与依赖

需要 Node.js、npm 和可连接的 MongoDB（本地实例或 MongoDB Atlas）。仓库未指定 Node.js 版本；旧依赖在不同 Node.js 版本下的兼容性需要自行验证。

```bash
git clone https://github.com/hyb82465/MERN-Stack.git
cd MERN-Stack
npm ci
npm ci --prefix frontend
```

### 2. 配置环境变量

在**仓库根目录**手动创建 `.env`（仓库没有 `.env.example`）：

```dotenv
NODE_ENV=development
PORT=5001
MONGO_URI=mongodb://127.0.0.1:27017/goalsetter
JWT_SECRET=replace_with_a_long_random_secret
```

| 变量 | 用途 |
| --- | --- |
| `NODE_ENV` | 开发时使用 `development`；`production` 会启用前端构建文件托管 |
| `PORT` | 后端端口，代码默认值为 `5001` |
| `MONGO_URI` | MongoDB 连接字符串，使用 Atlas 时替换为自己的连接信息 |
| `JWT_SECRET` | JWT 签名密钥，替换为自己的随机密钥 |

`.env` 已被 Git 忽略，不应提交真实数据库凭据或密钥。

### 3. 启动前后端

确认 MongoDB 可连接，然后在根目录执行：

```bash
npm run dev
```

- 前端：<http://localhost:3000>
- 后端：<http://localhost:5001>
- 前端开发代理在 `frontend/package.json` 中配置为 `http://localhost:5001`。修改后端端口时需同步调整代理。

访问前端后，先注册账户，再添加和删除目标。开发模式下直接打开后端根路径会看到 `Please set to production`，这是代码中的提示；前端开发入口仍是 3000 端口。

## 常用命令

以下命令均在仓库根目录执行：

| 命令 | 作用 |
| --- | --- |
| `npm run dev` | 同时启动后端热重载和前端开发服务 |
| `npm run server` | 仅启动后端，使用 nodemon |
| `npm run client` | 仅启动前端开发服务 |
| `npm start` | 使用 Node.js 启动后端，不自动构建前端 |
| `npm run build --prefix frontend` | 构建前端到 `frontend/build/` |
| `npm test --prefix frontend` | 运行前端测试脚本 |

### 本地验证生产模式

先构建前端，然后以生产模式启动后端（以下环境变量写法适用于 macOS / Linux）：

```bash
npm run build --prefix frontend
NODE_ENV=production npm start
```

此时 Express 会托管 `frontend/build/`，可通过 <http://localhost:5001> 访问应用。数据库与 JWT 环境变量仍然需要配置。

## API 概览

| 方法 | 路径 | 功能 | 需要登录 |
| --- | --- | --- | --- |
| POST | `/api/users` | 注册，提交 `name`、`email`、`password` | 否 |
| POST | `/api/users/login` | 登录，提交 `email`、`password` | 否 |
| GET | `/api/users/me` | 获取当前用户 | 是 |
| GET | `/api/goals` | 获取当前用户的目标 | 是 |
| POST | `/api/goals` | 创建目标，提交 `text` | 是 |
| PUT | `/api/goals/:id` | 更新自己的目标 | 是 |
| DELETE | `/api/goals/:id` | 删除自己的目标 | 是 |

受保护接口使用请求头 `Authorization: Bearer <token>`。更新目标时应只提交需要修改的 `text`。

## 项目状态

这是课程练习代码，保留了当时的实现，不应直接视为经过生产验证的服务。README 补写时核对了源码、依赖和脚本，未执行依赖安装、构建或数据库联调；启动命令是根据代码整理的操作说明，不代表已在当前环境完成运行验证。

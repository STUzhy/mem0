# OpenMemory Zeabur 部署指南

使用本指南将官方 mem0 OpenMemory 项目部署到 Zeabur 平台。

## 架构概览

OpenMemory 项目包含 4 个服务：
- **Qdrant**: 向量数据库，存储嵌入向量 (端口 6333)
- **PostgreSQL** (可选): 关系型数据库，存储元数据
- **openmemory-mcp**: 后端 API 和 MCP 服务器 (端口 8765)
- **openmemory-ui**: Next.js 前端界面 (端口 3000)

## 部署前准备

1. Fork 仓库: https://github.com/mem0ai/mem0
2. 拥有 Zeabur 账户和项目访问权限
3. OpenAI API 密钥（或兼容的 LLM 提供商）

## 部署步骤

### 1. 创建 Zeabur 项目

1. 登录 Zeabur 控制台
2. 创建新项目: `openmemory`
3. 按以下顺序部署服务:

### 2. 部署 Qdrant 向量数据库

1. 添加服务 → 模板 → Qdrant
2. 服务名称: `qdrant`
3. 端口: `6333` (自动配置)
4. 存储卷将自动挂载

### 3. 部署 PostgreSQL (可选)

1. 添加服务 → 模板 → PostgreSQL
2. 服务名称: `postgres`
3. 设置数据库名: `openmemory`
4. 存储卷将自动挂载

**注意**: 可以跳过 PostgreSQL，默认使用 SQLite。

### 4. 部署 OpenMemory MCP 后端

1. 添加服务 → Git 仓库
2. 连接你 Fork 的仓库
3. 服务名称: `openmemory-mcp`
4. 构建设置:
   - 根目录: `openmemory`
   - 构建命令: 自动检测
5. 环境变量:
   ```
   ZBPACK_DOCKERFILE_NAME=mcp
   OPENAI_API_KEY=sk-your-api-key-here
   USER=demo-user
   API_KEY=${OPENAI_API_KEY}
   QDRANT_URL=http://${QDRANT_PRIVATE_URL}:6333
   DATABASE_URL=${POSTGRES_DATABASE_URL}  # 使用 PostgreSQL 时可选
   ```
6. 存储: 挂载卷到 `/app/data` 用于 SQLite 持久化

### 5. 部署 OpenMemory UI 前端

1. 添加服务 → Git 仓库
2. 使用相同的仓库连接
3. 服务名称: `openmemory-ui`
4. 构建设置:
   - 根目录: `openmemory`
   - 构建命令: 自动检测
5. 环境变量:
   ```
   ZBPACK_DOCKERFILE_NAME=ui
   NEXT_PUBLIC_API_URL=https://${OPENMEMORY_MCP_URL}
   NEXT_PUBLIC_USER_ID=demo-user
   ```
6. 生成外部访问域名

## Dockerfile 配置说明

项目中包含两个 Dockerfile 文件：
- `Dockerfile.mcp`: 用于后端 MCP 服务
- `Dockerfile.ui`: 用于前端 UI 服务

### 配置方式

在每个服务的环境变量中设置 `ZBPACK_DOCKERFILE_NAME` 来指定使用的 Dockerfile：
- 后端服务设置: `ZBPACK_DOCKERFILE_NAME=mcp`
- 前端服务设置: `ZBPACK_DOCKERFILE_NAME=ui`

### 替代配置方法

你也可以使用以下任一方式替代环境变量配置：

1. **重命名 Dockerfile**: 
   - `openmemory-mcp.Dockerfile` (后端)
   - `openmemory-ui.Dockerfile` (前端)

2. **使用 zbpack.json**:
   在服务根目录创建 `zbpack.json` 文件：
   ```json
   {
     "dockerfile": {
       "name": "mcp"
     }
   }
   ```

## 环境变量参考

### 后端 (openmemory-mcp)
| 变量名 | 描述 | 必需 |
|--------|------|------|
| `ZBPACK_DOCKERFILE_NAME` | 指定 Dockerfile (设为 `mcp`) | 是 |
| `OPENAI_API_KEY` | OpenAI API 密钥 | 是 |
| `USER` | 默认用户 ID | 是 |
| `API_KEY` | API 密钥（通常与 OpenAI 相同） | 是 |
| `QDRANT_URL` | 向量数据库 URL | 是 |
| `DATABASE_URL` | PostgreSQL 连接字符串 | 否 |
| `OPENAI_BASE_URL` | 自定义 OpenAI 端点 | 否 |
| `LLM_MODEL` | LLM 模型名称 | 否 |
| `EMBED_MODEL` | 嵌入模型名称 | 否 |

### 前端 (openmemory-ui)
| 变量名 | 描述 | 必需 |
|--------|------|------|
| `ZBPACK_DOCKERFILE_NAME` | 指定 Dockerfile (设为 `ui`) | 是 |
| `NEXT_PUBLIC_API_URL` | 后端 API URL | 是 |
| `NEXT_PUBLIC_USER_ID` | 默认用户 ID | 是 |

## MCP 客户端设置

部署完成后，使用以下命令连接 MCP 客户端:

```bash
npx install-mcp https://your-backend-domain.zeabur.app/mcp/<client-name>/sse/<user-id> --client <client-name>
```

### 支持的客户端
- Claude Desktop
- Cursor
- Cline
- Windsurf

## 测试部署

1. 访问你的 UI 域名进入 Web 界面
2. 使用首选客户端测试 MCP 连接
3. 通过 UI 或 MCP 工具添加记忆
4. 验证记忆可搜索且持久化

## 故障排除

### 常见问题

1. **"服务器传输意外关闭"**
   - 检查 API 密钥权限
   - 验证存储卷正确挂载
   - 查看后端服务日志

2. **UI 无法加载**
   - 验证 `NEXT_PUBLIC_API_URL` 指向正确的后端域名
   - 检查前端构建日志错误

3. **数据库连接错误**
   - PostgreSQL: 验证 `DATABASE_URL` 格式
   - SQLite: 确保存储卷已挂载

4. **Qdrant 连接失败**
   - 验证 `QDRANT_URL` 使用内部服务 URL
   - 检查 Qdrant 服务状态

### 监控

- 在 Zeabur 控制台检查服务日志
- 监控资源使用和扩容情况
- 设置服务宕机警报

## 扩容考虑

- **后端**: 可通过负载均衡器水平扩容
- **前端**: 无状态，易于扩容
- **Qdrant**: 单实例，持久化存储
- **PostgreSQL**: 生产环境建议使用托管数据库

## 安全注意事项

- 使用强 API 密钥并定期轮换
- 为所有外部域名启用 HTTPS
- 考虑使用 VPC 网络进行内部通信
- 为公共端点实施速率限制

此部署提供具有 Web UI 和 MCP 服务器功能的完整 OpenMemory 实例。
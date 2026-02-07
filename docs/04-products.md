# 已落地产品矩阵

本社区以“可落地、可验证”为原则，优先建设具备真实场景价值的产品与工具。

## 1) DApp 应用商店（社区节点服务）
- **功能**：REST API（public/admin/internal）、SIWE 登录、UCAN 访问控制、WebDAV 存储直连、设计文档
- **状态**：Node.js 服务 + 可选 Web 前端，支持本地与 Docker 部署，配置模板齐全
- **下一步**：完成 `config.js` 配置，启动服务并联调 WebDAV 与前端
- **仓库**：https://github.com/kobofare/node

## 2) 钱包插件
- **功能**：创建/导入钱包、发送交易、网络切换、多账户、历史记录、二维码等；支持多项 EIP
- **状态**：已支持 EIP-1193/2255/712/4361/5573 等，EIP-55 部分支持
- **下一步**：扩展 NFT 支持、更多链与协议、补齐 EIP-55 校验
- **仓库**：https://github.com/kobofare/wallet

## 3) 数字仓库（WebDAV）
- **功能**：WebDAV 文件 CRUD、用户与配额管理、Basic/Web3/JWT/UCAN 认证、API 文档
- **状态**：配置模板 + 迁移工具 + 健康检查，可本地启动，支持 UCAN
- **下一步**：完成 `config.yaml` 与数据库初始化，接入客户端/应用商店
- **仓库**：https://github.com/kobofare/webdav

## 4) 知识库（RAG 中台）
- **功能**：多租户隔离、公共/私有知识库、Schema+向量字段、记忆服务、摄取作业、工作流与插件扩展、审计日志、控制台
- **状态**：后端 + 前端控制台齐全，文档更新时间 2026-02-02
- **下一步**：安装依赖并启动后端，访问控制台并完成应用接入
- **仓库**：https://github.com/kobofare/knowledge

## 5) 数字人面试官
- **功能**：简历解析、题目生成、面试管理、AI 评估报告（RAG 驱动）
- **状态**：Docker Compose 推荐，支持本地开发，默认端口 8080
- **下一步**：配置 QWEN/MinerU 等密钥，选择 Docker 或本地启动，接入真实面试场景
- **仓库**：https://github.com/kobofare/interviewer

## 6) 督学老师数字人
- **功能**：面向教育机构的做题系统，覆盖老师与学生角色
- **状态**：Java17 + Maven + Postgres + MinIO，支持 Docker 构建与部署
- **下一步**：配置 `.env` 与数据库/对象存储，构建镜像并启动服务
- **仓库**：https://github.com/kobofare/inspector

## 产品矩阵的整体目标
- 建立可迁移、可复用、可组合的用户智能体系
- 让数据主权从理念变为可交互的用户体验

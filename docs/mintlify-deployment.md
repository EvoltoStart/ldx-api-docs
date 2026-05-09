# Mintlify 部署流程

本文档说明当前项目的 Mintlify 文档发布方案。

## 架构方案

由于主项目托管在内网 GitLab，Mintlify Cloud 无法直接访问内网仓库，因此采用文档白名单同步方案。

```text
内网 GitLab：保存源码和完整文档
        ↓
GitLab CI：只同步白名单文档
        ↓
GitHub：保存 docs-only 发布仓库
        ↓
Mintlify：连接 GitHub 仓库并自动部署
```

## 外部文档仓库

当前 GitHub 文档发布仓库：

```text
https://github.com/EvoltoStart/ldx-api-docs
```

该仓库只用于 Mintlify 发布，不作为文档源头。不要手动长期维护该仓库内容，正式变更应提交到内网 GitLab 后由 CI 同步。

## GitLab CI 变量

在内网 GitLab 项目中配置以下变量：

```text
DOCS_REPO_HTTP_URL=github.com/EvoltoStart/ldx-api-docs.git
DOCS_REPO_BRANCH=main
DOCS_REPO_USER=x-access-token
DOCS_REPO_TOKEN=GitHub 写入 token
```

安全要求：

- `DOCS_REPO_TOKEN` 必须开启 Masked。
- 如果同步任务只在受保护分支运行，可以开启 Protected。
- Token 只授予 `ldx-api-docs` 仓库的写入权限，不要使用账号全权限 Token。

## 同步内容

当前 CI 只同步以下文件：

```text
docs.json
docs/index.mdx
docs/mintlify-deployment.md
docs/openapi/api.json
docs/openapi/relay.json
```

后续如需增加文档页面，应同时修改：

```text
docs.json
.gitlab-ci.yml 中的同步白名单
```

## 部署步骤

1. 在内网 GitLab 提交文档变更。
2. GitLab CI 执行 `sync_mintlify_docs`。
3. CI 清理 GitHub 文档仓库中的旧文件。
4. CI 复制白名单文档到 GitHub 仓库。
5. CI 提交并推送到 `DOCS_REPO_BRANCH`。
6. Mintlify 检测到 GitHub 仓库更新后自动部署。

## 验证方式

同步任务成功后，检查 GitHub 仓库是否只包含白名单文件。

随后在 Mintlify 控制台连接仓库：

```text
Repository: EvoltoStart/ldx-api-docs
Branch: main
Docs directory: 根目录
```

如果 Mintlify 部署成功，并能看到 “LDX API 文档” 首页和 API Reference，说明部署链路已经打通。

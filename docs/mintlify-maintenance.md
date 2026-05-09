# Mintlify 文档维护与配置说明

本文档用于说明当前项目的 Mintlify 文档发布方案、后续维护方式和常见配置项。

## 一、当前发布架构

当前项目主仓库在内网 GitLab，Mintlify Cloud 无法直接访问内网仓库，因此采用“内网源仓库 + GitHub 文档发布仓库”的方式。

```text
内网 GitLab 项目
        ↓ GitLab CI 同步白名单文档
GitHub 文档仓库 EvoltoStart/ldx-api-docs
        ↓ Mintlify 自动部署
Mintlify 文档站点
```

维护原则：

- 内网 GitLab 是唯一文档源头。
- GitHub 仓库只作为 Mintlify 发布仓库。
- 不要长期手动修改 GitHub 文档仓库，避免和内网 GitLab 源文档不一致。
- 不要把日志、测试报告、SQL、密钥、内网敏感说明同步到 GitHub。

## 二、GitLab CI 变量配置

在内网 GitLab 项目的 `Settings -> CI/CD -> Variables` 中配置：

```text
DOCS_REPO_HTTP_URL=github.com/EvoltoStart/ldx-api-docs.git
DOCS_REPO_BRANCH=main
DOCS_REPO_USER=x-access-token
DOCS_REPO_TOKEN=GitHub 文档仓库写入 Token
```

变量说明：

| 变量名 | 作用 |
| --- | --- |
| `DOCS_REPO_HTTP_URL` | GitHub 文档仓库地址，不需要带 `https://` |
| `DOCS_REPO_BRANCH` | GitHub 文档仓库目标分支，默认 `main` |
| `DOCS_REPO_USER` | GitHub Token 用户名，GitHub 推荐使用 `x-access-token` |
| `DOCS_REPO_TOKEN` | 用于 push 文档的 GitHub Token |

安全要求：

- `DOCS_REPO_TOKEN` 必须开启 `Masked`。
- 如果同步任务只允许在受保护分支运行，可以开启 `Protected`。
- Token 只授予 `EvoltoStart/ldx-api-docs` 仓库的写入权限。
- 不要把 Token 写入 `.gitlab-ci.yml`、代码、文档或聊天记录。

## 三、Mintlify 配置文件

Mintlify 入口配置在项目根目录：

```text
docs.json
```

当前配置主要包含两类内容：

```text
项目文档
API Reference
```

项目文档通过 `navigation.tabs[].groups[].pages` 配置。

OpenAPI 接口文档通过 `navigation.tabs[].openapi` 配置：

```json
"openapi": [
  "docs/openapi/api.json",
  "docs/openapi/relay.json"
]
```

注意：

- `docs.json` 中页面路径不写文件扩展名。
- 例如 `docs/ionet-client.md` 在 `docs.json` 中写成 `docs/ionet-client`。
- 新增文档页面后，必须同时更新 `docs.json` 和 `.gitlab-ci.yml` 的同步白名单。

## 四、当前同步白名单

当前 `.gitlab-ci.yml` 中 `sync_mintlify_docs` 任务只同步以下文件：

```text
docs.json
docs/installation/BT.md
docs/channel/other_setting.md
docs/api-key-invite-payment-code-route.md
docs/api-key-invite-payment-code-map.md
docs/subscription-redemption-order-routes.md
docs/ionet-client.md
docs/mintlify-maintenance.md
docs/openapi/api.json
docs/openapi/relay.json
```

同步任务不会清空 GitHub 仓库，因此如果从白名单中删除某个文件，需要手动到 GitHub 文档仓库删除对应旧文件。

## 五、新增文档页面流程

新增一篇项目文档时，按以下步骤操作：

1. 在内网 GitLab 项目的 `docs` 目录下新增 Markdown 文件。
2. 确认文档不包含密钥、内网敏感地址、客户数据或测试日志。
3. 在 `docs.json` 中把页面路径加入对应导航分组。
4. 在 `.gitlab-ci.yml` 的 `sync_mintlify_docs` 任务中加入对应 `cp` 命令。
5. 提交代码到内网 GitLab。
6. 等待或手动运行 `sync_mintlify_docs`。
7. 到 GitHub `EvoltoStart/ldx-api-docs` 仓库确认文件已同步。
8. 到 Mintlify 控制台确认部署成功。

示例：

新增文件：

```text
docs/example-api.md
```

`docs.json` 中增加：

```json
"docs/example-api"
```

`.gitlab-ci.yml` 中增加：

```bash
cp "${CI_PROJECT_DIR}/docs/example-api.md" docs/example-api.md
```

## 六、更新 OpenAPI 文档

当前 OpenAPI 文件位于：

```text
docs/openapi/api.json
docs/openapi/relay.json
```

更新后需要确认：

- 文件是合法 JSON。
- OpenAPI 版本为 `3.0` 或更高。
- 不包含不应公开的内部地址、测试 Token 或真实用户数据。
- GitLab CI 成功同步到 GitHub。
- Mintlify 的 API Reference 页面能正常展示。

本地可使用 Node 做基础 JSON 校验：

```bash
node -e "JSON.parse(require('fs').readFileSync('docs/openapi/api.json','utf8')); JSON.parse(require('fs').readFileSync('docs/openapi/relay.json','utf8')); console.log('openapi ok')"
```

## 七、本地校验方式

基础校验：

```bash
node -e "JSON.parse(require('fs').readFileSync('docs.json','utf8')); console.log('docs.json ok')"
```

如果已安装 Mintlify CLI，可以执行：

```bash
mint validate
```

本地预览：

```bash
mint dev
```

注意：

- `mint dev` 需要在包含 `docs.json` 的目录执行。
- `mint validate` 会检查文档构建和 OpenAPI 配置，适合上线前执行。

## 八、常见问题

### 1. GitLab CI 提示认证失败

重点检查：

- `DOCS_REPO_TOKEN` 是否正确。
- Token 是否有 GitHub 文档仓库写入权限。
- Token 是否过期。
- GitLab 变量是否开启了 `Protected`，但当前分支不是受保护分支。

### 2. GitHub 仓库没有更新

重点检查：

- `sync_mintlify_docs` 任务是否执行。
- CI 日志是否显示 `No Mintlify docs changes to sync.`。
- 文档是否已经加入 `.gitlab-ci.yml` 同步白名单。

### 3. Mintlify 页面没有新文档

重点检查：

- GitHub 仓库是否已经出现对应文件。
- `docs.json` 是否已经加入对应页面路径。
- Mintlify 部署记录是否成功。

### 4. GitHub 仓库残留旧文档

当前 CI 不执行清空操作，因此旧文件不会自动删除。

处理方式：

- 手动删除 GitHub 文档仓库中的旧文件。
- 同步确认 Mintlify 页面不再引用旧文件。

## 九、上线前检查清单

每次调整 Mintlify 文档配置前，建议检查：

- `docs.json` 能被 JSON 正常解析。
- `docs.json` 中的页面路径都能找到对应 `.md` 或 `.mdx` 文件。
- `.gitlab-ci.yml` 同步白名单包含所有需要发布的文档。
- OpenAPI 文件是合法 JSON。
- 文档不包含敏感信息。
- GitLab CI 同步成功。
- Mintlify 部署成功。

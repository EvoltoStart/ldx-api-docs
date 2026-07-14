# LDX API Mintlify 文档发布说明

本仓库是 LDX API 的 Mintlify 文档发布仓库，由内网 GitLab 项目通过 CI 自动同步生成。

## 发布链路

```text
内网 GitLab 项目
  -> GitLab CI 同步文档白名单
  -> GitHub 仓库 EvoltoStart/ldx-api-docs
  -> Mintlify 自动部署
```

## 维护规则

- 内网 GitLab 是文档源头。
- GitHub 仓库只作为 Mintlify 发布仓库。
- 不建议直接在 GitHub 仓库长期手动修改文档。
- 新增页面需要同时更新内网项目里的 `docs.json` 和 `.gitlab-ci.yml` 同步白名单。
- OpenAPI 页面只发布 `docs/zh/openapi/relay-api.mintlify.json` 和 `docs/en/openapi/relay-api.mintlify.json`（已合并原先的 core-api 与 compatibility-api）。
- `api.json`（已重命名为 `platform-api.json`）与 `relay.json` 已移除，不再作为发布文件。

## 目录说明

```text
docs/
  zh/
    getting-started/    中文 API 入门说明
    guides/             中文 Agent 接入指南
    openapi/            中文 OpenAPI 文件
  en/
    getting-started/    英文 API 入门说明
    guides/             英文 Agent 接入指南
    openapi/            英文 OpenAPI 文件
  images/                Mintlify 发布使用的 logo 与 favicon
```

## 本地校验

```bash
node -e "JSON.parse(require('fs').readFileSync('docs.json','utf8')); console.log('docs.json ok')"
node docs/zh/openapi/generate-mintlify-openapi.js
npx -y @redocly/cli lint docs/zh/openapi/relay-api.mintlify.json docs/en/openapi/relay-api.mintlify.json --max-problems 30
```

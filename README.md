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
- OpenAPI 页面使用 `docs/zh/openapi/*.mintlify.json` 和 `docs/en/openapi/*.mintlify.json`。

## 目录说明

```text
docs/
  zh/
    project/    中文项目文档
    business/   中文业务接口说明
    glossary/   中文术语表
    openapi/    中文 OpenAPI 文件
  en/
    project/    英文项目文档
    business/   英文业务接口说明
    glossary/   英文术语表
    openapi/    英文 OpenAPI 文件
```

## 本地校验

```bash
node -e "JSON.parse(require('fs').readFileSync('docs.json','utf8')); console.log('docs.json ok')"
node docs/zh/openapi/generate-mintlify-openapi.js
npx -y @redocly/cli lint docs/zh/openapi/api.mintlify.json docs/zh/openapi/relay.mintlify.json docs/en/openapi/api.mintlify.json docs/en/openapi/relay.mintlify.json --max-problems 10
```


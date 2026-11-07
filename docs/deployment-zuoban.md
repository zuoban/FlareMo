# zuoban/FlareMo 部署

这个 fork 使用 `.github/workflows/deploy-cloudflare.yml` 部署到 Cloudflare Workers。

## 自动发布

推送代码到 `main`（包括合并 PR、通过 GitHub Sync fork 同步上游）会自动发布。
上游仓库的更新需要先同步到这个 fork，才能触发部署。

工作流使用锁文件安装依赖，创建或复用 D1、R2、Queues 和 Vectorize 资源，
构建前端，执行远端 D1 migrations，发布 Worker，同步认证 secrets，然后检查线上服务。
生产部署串行执行，避免同时运行 migration 和发布。

也可以在 Actions → Deploy to Cloudflare → Run workflow 手动运行；
勾选 `dry_run` 时只构建并只读检查现有资源，不迁移数据库、不发布、不更新密钥。

## 凭据

以下内容保存在 GitHub 仓库 Actions secrets：

- `CLOUDFLARE_API_TOKEN`：长期部署 API Token。
- `CLOUDFLARE_ACCOUNT_ID`：目标 Cloudflare 账号。
- `BETTER_AUTH_SECRET`：登录 session 签名密钥。
- `FLAREMO_BOOTSTRAP_SECRET`：首次初始化口令。

不要在仓库、日志或聊天里记录密钥。轮换 GitHub secret 后手动执行一次正式部署，
让 Worker 使用更新后的密钥。

## 首次初始化

打开工作流摘要里的 `/setup` 链接，由部署者填写初始化口令和自己的登录资料。
初始化完成后使用 `/login` 登录；后续代码发布不会重置账号或笔记。

## 自定义域名

在 Cloudflare 将域名绑定到 Worker，然后在 GitHub Actions variables 中设置
`FLAREMO_PUBLIC_URL` 为完整 HTTPS origin，再重新部署。

## 排查发布失败

打开 Actions 中最近一次 Deploy to Cloudflare 运行，查看失败步骤。
排除问题后可以重新运行工作流。数据库迁移先于 Worker 发布，
代码更新需要遵守上游的向后兼容 migration 约定；回滚 Worker 不会回滚数据库。

# admin 与 mss-boot 易用性/完整度推进记忆

时间：2026-06-10 09:12 CST
更新：2026-06-10 09:35 CST

## 背景

用户要求继续推进 `admin` 和 `mss-boot` 的整体完整度，并优化易用性。当前目标偏向开源项目第一体验、GitHub AI/CI 可维护性、以及本地开发路径一致性。

## 本轮处理范围

### mss-boot

- PR：https://github.com/mss-boot-io/mss-boot/pull/386
- 分支：`codex/open-source-security-baseline-20260607`
- 最终提交：`9e82bd8 fix: align version ldflags`
- 校对 GitHub Release 后确认最新版本为 `v0.7.3`，README 原先仍写 `v0.7.2`，安装命令仍写 `v0.7.1`。
- 更新 `README.md` / `README.Zh-cn.md`：
  - 最新版本统一为 `v0.7.3`。
  - 快速安装命令统一为 `go get github.com/mss-boot-io/mss-boot@v0.7.3`。
  - 增加 Go 1.26+ 要求与 `make tidy/test/coverage/lint` 本地检查入口。
  - 补充 v0.7.x 亮点：GORM 查询缓存失效、Mongo ObjectID 安全校验、结构化 issue forms 等。
- 更新 `CHANGELOG.md`，补齐 `v0.7.3` 条目。
- 修正 `Makefile` 中旧组织路径 `github.com/matchstalk/mss-boot` 为 `github.com/mss-boot-io/mss-boot`。
- 给 `Makefile` 增加 `test`、`coverage`、`tidy` 目标。
- 按 Copilot Review 意见修复 `Makefile` 版本注入路径：`LDFLAGS` 现在写入 `github.com/mss-boot-io/mss-boot/pkg/version.buildDate`、`gitCommit`、`gitVersion`，并使用 UTC ISO8601 构建时间。

### mss-boot-admin

- PR：https://github.com/mss-boot-io/mss-boot-admin/pull/390
- 分支：`codex/fix-language-public-beta`
- 最终提交：`1e0bea8 Merge remote-tracking branch 'origin/main' into codex/fix-language-public-beta`
- 更新 `README.md` / `README.zh-CN.md`：
  - Go 版本统一为 1.26+。
  - 本地快速开始改为 SQLite 默认路径，避免继续误导贡献者以为 `DB_DSN` 环境变量在 local 配置下必然生效。
  - MySQL/Redis 改为可选集成依赖。
  - 前端启动命令改为 `corepack enable`、`pnpm install`、`pnpm start:dev`。
- 更新 `.github/copilot-instructions.md`：
  - 说明本地默认读取 `config/application.yml`，当前基线为 SQLite。
  - `DB_DSN` / `DB_DRIVER` 改为仅在 GORM/远程配置源或自定义数据库配置时需要。
  - 故障排查从优先检查 `DB_DSN` 改为优先检查 `config/application.yml` 的 database 配置。
- 修复 `Makefile`：把 `test`、`deps`、`generate`、`lint`、`fix-lint` 加入 `.PHONY`。本轮发现原先 `make test` 会因为本地存在 `test` 目标而输出 `make: 'test' is up to date.`，没有真正跑测试。
- 注意：`testdata/test.tar.gz` 与 `testdata/test.zip` 在本轮开始前已经是脏文件，本轮未修改、未提交。

### mss-boot-admin-antd

- PR：https://github.com/mss-boot-io/mss-boot-admin-antd/pull/105
- 分支：`codex/open-source-issue-templates-20260607`
- 最终提交：`0cdbbfc docs: align frontend startup commands`
- 更新 `package.json`：
  - `engines.node` 从 `>=12.0.0` 更新为 `>=22.0.0`。
  - 增加 `engines.pnpm >=9.0.0`。
  - 将内部脚本链路从 `npm run` 改为 `pnpm`。
  - 修正文案中的 Ant Design / Umi 描述，并弱化已降级的虚拟模型方向。
- 增加 `.nvmrc`，内容为 `22`。
- 更新 `README.md` / `README.zh-CN.md`：
  - 前端 Node.js 22+、pnpm 9+。
  - 后端本地默认 SQLite。
  - 启动命令统一为 `corepack enable`、`pnpm install`、`pnpm start:dev`。
- 更新 `.github/copilot-instructions.md`：
  - 增加运行时基线：Node 22+、pnpm 9+、不要新增 npm/yarn lockfile。
- 优化 `src/requestErrorConfig.ts`：
  - 拆出纯函数 `src/util/requestError.ts`。
  - HTTP 错误消息现在可安全处理 `msg`、`errorMessage`、`message`、`error`、`detail`、`reason`、字符串 body、空 body。
  - 移除响应拦截器中的泛化 `请求失败！`，避免和错误处理器重复弹框。
- 增加 `src/util/requestError.test.ts` 覆盖错误消息提取。
- 更新 `src/typings.d.ts`，补齐 `REACT_APP_ENV` 的 `local`、`alpha`、`beta`、`prod` 类型。
- 合并 `origin/main` 后保留主线依赖升级和 `pnpm.overrides`，并按 Copilot Review 意见将 README 启动命令统一为当前真实脚本：`pnpm dev` / `pnpm start:no-mock`。

## 验证结果

- `mss-boot`: `make test` 通过。
- `mss-boot`: 手动验证 `go test -ldflags ... ./pkg/version` 通过，确认 Makefile 版本注入路径有效。
- `mss-boot-admin`: `make test` 通过，确认 Makefile `.PHONY` 修复后真实执行了测试。
- `mss-boot-admin`: `make build` 通过；构建生成的本地 `admin` 二进制已清理。
- `mss-boot-admin`: GitHub PR 检查全部通过，包括 CI build、CodeQL、govulncheck、Docs Drift、PR Guard、Copilot Setup、Dependabot 配置检查。
- `mss-boot-admin-antd`: `pnpm test -- --runInBand src/util/requestError.test.ts` 通过。
- `mss-boot-admin-antd`: `pnpm tsc` 通过。
- `mss-boot-admin-antd`: `pnpm lint:js` 通过。
- `mss-boot-admin-antd`: `pnpm build:local` 通过。
- `mss-boot`: GitHub PR 检查全部通过，包括 ci test/lint、govulncheck、CodeQL Advanced、Docs Drift、PR Guard。
- `mss-boot-admin-antd`: GitHub PR 检查全部通过，包括 CI build、CodeQL、Docs Drift、PR Guard。

## 当前状态

- 三个 PR 均已推送并通过本地验证和 GitHub Actions。
- 三个 PR 当前均为 `REVIEW_REQUIRED`，`mergeStateStatus` 显示 `BLOCKED` 的原因是仓库规则要求人工 review。
- 本轮未合并 PR、未发布镜像、未发布 Cloudflare。
- `mss-boot-admin` 工作区仍保留用户原有脏文件：`testdata/test.tar.gz`、`testdata/test.zip`，不要在后续无关任务中提交或回滚它们。

## 后续建议

- `mss-boot-admin` 后续应补一个更轻量的本地依赖路径：单节点 Redis compose 或明确说明当前 Redis compose 是主从哨兵拓扑，避免新人误以为必须启动全套。
- `mss-boot-admin` 可以继续把 `config/application.yml` 的 SQLite 默认体验做成更明确的 `make dev` / `make migrate` / `make run`。
- `mss-boot-admin-antd` 后续可处理 Browserslist 数据库陈旧问题，但应评估是否会带来 lockfile 变化。
- 前端 Cloudflare 发布仍遵守低频原则：本地和 CI 验证充分后再发布 beta/prod。

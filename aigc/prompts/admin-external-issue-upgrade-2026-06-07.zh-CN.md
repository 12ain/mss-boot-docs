# mss-boot-admin 外部 Issue 升级记录

日期：2026-06-07

## 范围

本轮只评估 `mss-boot-io/mss-boot-admin` 中外部用户提交的 open issues：

- `#126` 添加查看在线用户、强退功能
- `#105` 数据更新后接口查询数据未刷新
- `#89` migration 支持纯 SQL 脚本和回滚
- `#75` Bug Report

排除 maintainer 自己创建的 issue、bot issue、依赖升级 PR。

## 结论

- `#105` 有足够信号做一期修复：外部用户反馈启用 `queryCache` 后更新数据仍读到旧详情。
  当前 admin 启动时没有把 mss-boot query cache 初始化回调传入 `Cache.Init`，
  也没有把更新/删除后的 table tag 清理函数绑定到 mss-boot GORM actions。
- `#126` 是合理功能方向，但普通登录 JWT 当前不是服务端会话模型；仓库已有个人访问
  token 的撤销能力，仍不足以直接实现“在线用户 + 强退登录会话”。应单独设计会话模型、
  revoke 策略、审计与前端联动。
- `#89` 是迁移系统大功能，应先做 RFC：一期 forward SQL 脚本发现/排序/执行/版本记录，
  二期 rollback 命名、顺序、失败策略。不能混入小修 PR。
- `#75` 混合多个 bug 和一个功能建议；其中“删除模型后 MenuBar 残留生成菜单”
  已能从当前代码直接确认，适合独立小 PR 修复。其余问题仍需要拆分复现。

## 已推进

- 新建 PR：`mss-boot-admin#371`，标题 `fix: wire query cache invalidation`。
- 代码改动：
  - `config/config.go`：将 `cache.queryCache` 初始化接入 GORM query cache plugin。
  - `config/config.go`：绑定 `response/actions/gorm.CleanCacheFromTag`，更新/删除后按
    `gorm.cache:<table>` 清理关联缓存 key。
  - `config/query_cache_test.go`：覆盖 query cache 初始化和 tag 前缀清理。
- 新建 PR：`mss-boot-admin#372`，标题 `fix: clean generated menus after model delete`。
- 代码改动：
  - `apis/model.go`：为模型删除增加 after delete hook。
  - 删除模型后递归软删除 `/virtual/<model.path>` 菜单树。
  - 删除这些菜单/API 路径对应的 Casbin policy，并重新加载 policy。
  - `apis/model_test.go`：覆盖生成菜单树和 policy 被清理、无关菜单和 policy 不受影响。
- 社区治理：
  - `#105` 已回复进展并关联 `#371`，标签整理为 `bug`、`type/bug`、`area/backend`。
  - `#75` 已回复进展并关联 `#372`，明确只处理其中一个可验证子问题。
  - 为 `#126`、`#89` 创建并添加 `needs-rfc` 标签，表达需要设计/RFC 后再实现。

## 验证

```text
go test ./config
go test ./config ./middleware ./apis ./models
go test ./apis
go test ./apis ./models ./middleware
```

## 后续

- PR 合并后，在 `#105` 中确认这是一期修复；如果用户仍能复现，需要补充版本、接口路径、
  缓存配置、更新接口和读取接口。
- `#126` 建议另开设计 issue 或 RFC，再实现后端会话表/会话撤销/审计/权限控制。
- `#89` 建议先补迁移 RFC，不直接改运行时迁移器。
- `#75` 的模型删除菜单残留由 `#372` 处理；其余子问题继续保持 `needs-info`，
  等待拆分复现后逐项处理。

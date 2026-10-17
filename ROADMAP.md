# moon-paseto 路线图

目标：11 月轮交付 M1-M3（v4 两种模式 + 声明校验 + 官方向量全通过），M4 PASERK 与发布。
窗口：11 月轮日期未在章程流程表列出，以赛事群通知为准（按月度节奏推算 11 月第一周开始、
下旬截止）；本路线图按"窗口内四周"排布，开工日期以窗口开启为准。

约定：每个里程碑合入前 `moon check` / `moon test` / `moon fmt --check` 全绿；
官方测试向量是主要验收证据；每个里程碑 ≥2 个有意义的 commit。

## M0 工程骨架（立项即完成）✅

- [x] moon new 模板定制：模块名、LICENSE(Apache-2.0)、pre-commit(moon check)
- [x] CI：三平台 `moon check/test --target all` + `moon fmt --check`
- [x] README / ROADMAP / PROPOSAL 框架
- 验收：可构建工程 + 文档齐备

## M1 v4.public（Ed25519 签名令牌）

- [ ] 依赖接入：mooncry（Ed25519、BLAKE2b）
- [ ] PAE（Pre-Auth Encoding）实现 + 官方向量单测
- [ ] v4.public sign/verify：`v4.public.<payload>.<footer>` 打包与解析
- [ ] 官方 v4.public 测试向量全通过（含篡改/错误 key 负样本）
- [ ] CLI demo：签发与验证示例令牌
- 验收：官方向量 100% 通过；错误 key / 被篡改 payload 一律 verify 失败

## M2 v4.local（XChaCha20-Poly1305 加密令牌）

- [ ] mooncry XChaCha20-Poly1305 接入；key 拆分（split keys：ek/nk 与 PAE）
- [ ] v4.local encrypt/decrypt + 官方向量全通过
- [ ] 错误处理：认证失败不得泄漏原因细节（规范要求 constant-ish 失败路径）
- 验收：官方向量 100% 通过；随机篡改 bit 的令牌解密必败

## M3 footer / implicit assertions / 注册声明

- [ ] footer 的 JSON 解析与可选返回（规范行为：footer 不参与签名校验失败披露）
- [ ] implicit assertions（v4 特性）注入 PAE
- [ ] 注册声明校验工具：exp / nbf / iat / iss / aud / sub / tokenIdent
  （校验是可选工具层，核心库保持"只管令牌、不管语义"的边界）
- [ ] 互操作冒烟：与官方实现（paseto.js 或 aeronauticaly/paseto）交换令牌各 1 例
- 验收：含 footer/implicit 的官方向量通过；互操作冒烟通过

## M4 PASERK 子集 + 发布

- [ ] PASERK 序列化子集：k*.local / k*.public / k*.lid（key id）
- [ ] README 完善（含安全注意事项：算法确定、footer 不敏感声明）
- [ ] mooncakes.io 发布（0.1.0）
- [ ] 申报材料定稿（人工撰写）与提交前自查
- 验收：CI 三平台绿 + mooncakes 可安装 + 申报材料齐备

## 边界（明确不做）

- v1 / v2 / v3 遗留版本；PASERK 完整集（wrap/pme 系列按需后补）
- 密码学原语的自实现（一律依赖 mooncry）
- JWS/JWT 互转工具（保持单一职责）

## 风险登记

| 风险 | 缓解 |
| --- | --- |
| 密码学正确性责任 | 不自造原语（依赖 mooncry）；官方向量全量对照；篡改负样本测试 |
| 11 月轮窗口未确认 | 立即进赛事群确认；路线图按四周排布可压缩 |
| 生态定位被质疑（JWT 已有） | README/申报书写明 PASETO 与 JWT 的威胁模型差异及 mooncred 的互补关系 |
| spec 版本漂移 | 锚定 ve-02 当前版；声明遵循的测试向量日期 |

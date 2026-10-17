# moon-paseto 项目申报书（草案框架）

> **提交前必读**：章程要求申报书人工撰写——明显 AI 套话或看不出选手对选题有深刻认识的
> 会被驳回。本框架由开发事实整理而来，所有【】由你本人填写，"项目简介"与"实现路径"
> 两段务必用自己的语言重写后誊入飞书问卷。

## 基本信息

- 项目名称：moon-paseto —— PASETO v4 安全令牌的 MoonBit 实现
- 项目方向：安全基础设施 / 协议规范实现。通用性：任何后端服务签发与校验自包含
  认证令牌的场景（API 认证、服务间凭证、无状态会话）。
- 参赛方向：新生态项目建设（个人 /【团队按实际填写】）
- 是否为移植项目：是
- 原项目：PASETO 规范（ve-02）及官方测试向量；
  参考实现 aeronauticaly/paseto、paseto.js（均为思路参考，无代码复制）
- 原项目许可证：官方测试向量 ISC（Paragon Initiative Enterprises，
  已按要求保留声明，见 THIRD_PARTY_NOTICES.md）；参考实现 MIT
- 本项目许可证：Apache-2.0
- GitHub 仓库链接：https://github.com/xiguaAp6y3/moon-paseto
- 联系方式：【填写】

## 项目简介

MoonBit 生态的凭据层已有 mooncred（JWT/JOSE/JWK）、moon-token（会话生命周期）、
oauth2 客户端，但"自包含可验证令牌格式"这一层是空白。PASETO（ve-02 规范）用
"版本即算法承诺"的设计消灭了 JWT 最大的攻击面——算法混淆（`alg: none`、RS/HS
混用）：v4 版本固定 Ed25519（public）与 XChaCha20-Poly1305（local），实现方无从
选错。本项目按规范实现 v4 两种模式，密码学原语全部依赖 mooncry，本库只做 PAE
编码、令牌打包与校验，官方测试向量作为一等测试基线。

## 三个完整的预期使用场景

1. 无状态 API 认证：服务签发 v4.public 令牌给客户端，各实例只需内置公钥即可
   验证，无需共享会话存储；exp/nbf/iat 由校验工具层兜底。
2. 服务间凭证交换：两个内部服务用 v4.local 交换加密凭据（含敏感字段），
   只有持有对称 key 的对端能解——对应微服务间传递 PII 的合规需求。
3. 令牌内容审计：footer 携带 key-id 与非敏感元数据，配合 PASERK key-id
   支持 key 轮换期间的平滑验证。

每个场景写清楚：谁在什么情况下用、解决什么具体痛点。【结合你自己的后端经验补充】

## 拟实现的核心功能（含当前进度）

- M1：PAE 编码 + v4.public（Ed25519）sign/verify + 官方向量全通过
- M2：v4.local（XChaCha20-Poly1305）encrypt/decrypt + 官方向量全通过
- M3：footer / implicit assertions / 注册声明校验工具（exp/nbf/iat/iss/aud/sub）
- M4：PASERK 序列化子集（k*.local / k*.public / k*.lid）+ mooncakes 发布
- 已完成：工程骨架、三平台 CI、路线图与申报材料

## 生态位与竞品说明

- mooncred 已覆盖 JWT/JOSE/JWK：本项目不做 JWT，也不做 JWS 互转——PASETO 与 JWT
  是同一问题（自包含令牌）的两种威胁模型，规范层面 PASETO 删除了算法选择自由度。
  两者互补：需要与既有 JWT 体系互操作用 mooncred，新建体系追求"实现不可能选错
  算法"用 moon-paseto。
- moon-token 是会话生命周期管理层（多设备、轮换、踢出），消费令牌格式而不定义
  令牌格式，可接 PASETO 作为其令牌编码。
- mooncakes.io 现状：PASETO / PASERK 关键词零命中（2026-10 查重，覆盖协议名、
  功能词、宿主框架子包扫描——mooncred/moonbitstack 全栈核对过）。

## 明确不做

v1/v2/v3 遗留版本；密码学原语自实现（依赖 mooncry）；JWS/JWT 互转；
PASERK wrap/pme 系列（按需后补）。边界收缩是安全决策：多支持一个遗留版本
就多一类算法混淆责任。

## 实现路径（技术理解）

PAE（pre-authentication encoding）是 PASETO 安全性的核心：把 version、purpose、
payload、footer、implicit assertions 全部编码进认证输入，任何字段被替换都会
破坏认证标签——本库优先实现 PAE 并用官方向量单独测它。v4.public 只做签名校验
（公开可验证），v4.local 的 key 拆分（ek/nk）与 implicit assertions 的 PAE
注入顺序是规范细节中最容易做错的两处，均以向量钉死。校验失败路径不区分
"签名错/格式错"的细节（规范建议），避免成为 oracle。

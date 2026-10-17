# moon-paseto

> PASETO v4（Platform-Agnostic SEcurity TOkens）的 MoonBit 实现 —— 创建与验证防篡改、可自校验的安全令牌。

PASETO 是 JWT 的现代替代：算法不由攻击者选择（不存在 `alg: none` / 算法混淆攻击面），
版本即承诺——**v4** 固定为 Ed25519（`v4.public`）与 XChaCha20-Poly1305（`v4.local`）。
本库按官方规范实现 v4 全部两种模式，并以规范自带的测试向量作为一等测试。

## 为什么是 PASETO

MoonBit 生态的凭据版图：[mooncred](https://mooncakes.io/docs/moonbitstack/mooncred) 提供
JWT/JOSE/JWK，[moon-token](https://mooncakes.io/docs/mldong/moon-token) 管理会话生命周期，
oauth2 客户端已有——**自包含可验证令牌格式（PASETO）是空缺的一块**。
它与本作者的其他安全库组成完整栈：

```
moon-paseto（认证令牌：你是谁）
  → moon-httpsig（请求签名：请求未被篡改）
    → moonbit-casbin（授权决策：你能否做这件事）
```

## 当前状态（M1 前夜，详见 [ROADMAP.md](ROADMAP.md)）

- [x] 工程骨架、路线图、CI
- [ ] M1：v4.public（Ed25519 签名令牌）+ 官方测试向量
- [ ] M2：v4.local（XChaCha20-Poly1305 加密令牌）
- [ ] M3：footer / implicit assertions / 注册声明（exp/nbf/iat/iss/aud/sub）校验
- [ ] M4：PASERK 子集（key 序列化）与 mooncakes 发布

## 设计边界

- **不新造密码学**：Ed25519 / XChaCha20-Poly1305 / BLAKE2b 由
  [mooncry](https://mooncakes.io/docs/cc06b/mooncry) 提供，本库只做规范层的
  PAE（pre-authentication encoding）、令牌打包与校验逻辑。
- **只做 v4**：v1/v2/v3 为遗留版本（NIST 算法 + 各种历史包袱），官方亦建议仅用于
  兼容场景；不支持它们是有意的安全决策，不是功能缺失。
- 目标平台：native / wasm / js 全后端（mooncry 支持的范围内）。

## 开发

```bash
moon check
moon test
moon run cmd/main
```

## 上游与许可证

- 规范：[PASETO Specification（ve-02）](https://github.com/paseto-standard/paseto-spec)，
  官方测试向量作为本库测试基线（向量版权归 Paragon Initiative Enterprises，
  ISC 许可，见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)）；参考实现为
  aeronauticaly/paseto、paseto.js 等（思路参考，无代码复制）。
- 密码学原语：[cc06b/mooncry](https://mooncakes.io/docs/cc06b/mooncry)（Apache-2.0）。
- 本项目采用 Apache-2.0 许可证（见 [LICENSE](LICENSE)）。

# Git 使用 SSH 认证的原理与多账号隔离

## 核心结论

Git 使用 SSH 连接 GitHub 时，解决的是两个方向的身份确认：

- GitHub 通过公钥验证本机持有对应私钥，确认“你是谁”。
- 本机通过 `known_hosts` 验证 GitHub 的主机密钥，确认“对方确实是 GitHub”。

认证成功后，能否读取或写入仓库仍由 GitHub 上的账号和仓库权限决定。SSH 不会绕过权限控制。

## HTTPS 与 SSH 的区别

Git 访问 GitHub 常用两种协议：

```text
HTTPS：https://github.com/user/repo.git
SSH：  git@github.com:user/repo.git
```

两者都会加密代码传输，主要区别是证明用户身份的方式。

| 维度 | HTTPS | SSH |
|---|---|---|
| 用户凭证 | Personal Access Token / OAuth | SSH 私钥 |
| GitHub 保存 | Token 相关记录 | SSH 公钥 |
| 本机保存 | Token | SSH 私钥 |
| 默认端口 | 443 | 22 |
| 多账号隔离 | 依赖凭据管理器 | 可为不同账号指定不同密钥 |
| 常见问题 | Token 无效或未保存，反复要求密码 | 密钥未添加、选错密钥、主机指纹未信任 |

GitHub 已不支持使用账号密码进行 Git 推送。使用 HTTPS 时，界面中的“密码”通常实际指 Personal Access Token。若 Token 没有被 macOS Keychain 等凭据管理器正确保存，Obsidian Git 等客户端就可能反复询问密码。

## 公钥与私钥如何生成

公钥和私钥由本机的 `ssh-keygen` 同时生成：

```bash
ssh-keygen -t ed25519 -C "github-personal" -f ~/.ssh/id_ed25519_github_personal
```

生成两个文件：

```text
~/.ssh/id_ed25519_github_personal      # 私钥，只保存在本机
~/.ssh/id_ed25519_github_personal.pub  # 公钥，可以添加到 GitHub
```

生成过程：

1. `ssh-keygen` 在本机使用安全随机数生成私钥。
2. 根据私钥计算出对应的公钥。
3. 将公钥添加到 GitHub 账号。
4. 私钥始终留在本机，不会发送给 GitHub。

只有公钥无法推导出私钥，也无法冒充私钥持有者。私钥一旦泄露，应立即从 GitHub 删除对应公钥并生成新密钥。

建议为私钥设置 passphrase，并通过 `ssh-agent` 或 macOS Keychain 管理，降低私钥文件泄露后的风险。

## SSH 如何证明本机身份

认证过程可以简化为：

```text
本机请求连接 GitHub
    ↓
GitHub 发出一次性挑战
    ↓
本机使用私钥对挑战进行签名
    ↓
GitHub 使用账号中保存的公钥验证签名
    ↓
验证成功，GitHub 确认本机持有对应私钥
```

关键点：

- 私钥参与签名，但不会离开本机。
- GitHub 保存的是公钥，只负责验证签名。
- 挑战是一次性的，旧签名不能直接用于下一次登录。
- 公钥绑定在哪个 GitHub 账号，认证后就代表哪个账号。

验证当前 SSH 身份：

```bash
ssh -T git@github.com
```

成功时 GitHub 会返回当前认证账号。GitHub 不提供 SSH Shell，因此命令可能以非零状态退出，这不代表认证失败。

## `known_hosts` 如何确认服务端身份

公私钥认证解决的是 GitHub 如何确认客户端身份，但客户端也必须确认自己连接的确实是 GitHub。

首次连接时，SSH 会展示 GitHub 的主机密钥指纹。确认后，主机密钥会记录在：

```text
~/.ssh/known_hosts
```

之后再次连接时：

- 主机密钥一致：继续连接。
- 主机密钥发生变化：SSH 发出警告并拒绝连接。

这能降低中间人冒充 GitHub、截获连接的风险。出现 `Host key verification failed`，通常表示主机密钥尚未被信任，或保存的主机密钥与当前服务端不一致。

## 公司与个人 Git 密钥如何隔离

不同身份应使用不同密钥，例如：

```text
~/.ssh/id_ed25519                  # 公司内部 Git
~/.ssh/id_ed25519_github_personal  # 个人 GitHub
```

最小影响范围的做法，是只为某个仓库指定密钥：

```bash
git config --local core.sshCommand \
  "ssh -i ~/.ssh/id_ed25519_github_personal -o IdentitiesOnly=yes"
```

其中：

- `--local`：配置只写入当前仓库的 `.git/config`，不影响其他仓库。
- `-i`：指定当前仓库使用的私钥。
- `IdentitiesOnly=yes`：只尝试指定密钥，避免 SSH 自动尝试公司密钥或其他密钥。

然后将远程地址切换为 SSH：

```bash
git remote set-url origin git@github.com:user/repo.git
```

查看当前仓库是否正确隔离：

```bash
git remote -v
git config --local --get core.sshCommand
```

仓库级配置适合少量个人仓库。若长期维护多个 GitHub 账号，可以进一步在 `~/.ssh/config` 中为不同账号配置不同 Host 别名。

## 常见故障的判断顺序

### 反复要求输入密码

先检查远程地址：

```bash
git remote -v
```

- 地址以 `https://` 开头：正在使用 HTTPS，检查 Token 和凭据管理器；也可以切换到 SSH。
- 地址以 `git@` 开头：正在使用 SSH，问题通常不是 GitHub 密码。

### `Permission denied (publickey)`

说明服务端无法使用提供的公钥确认身份。依次检查：

1. 公钥是否添加到了正确的 GitHub 账号。
2. 当前仓库是否使用了正确的私钥。
3. `IdentitiesOnly=yes` 是否避免了选错密钥。
4. 认证账号是否拥有目标仓库权限。

### `Host key verification failed`

说明本机尚未信任服务端主机密钥，或已保存的主机密钥不匹配。应核对服务端官方公布的指纹，不能为了省事长期关闭主机校验。

### 验证认证成功但无法访问仓库

身份认证与仓库授权是两件事：

- `ssh -T git@github.com` 验证“你是谁”。
- `git ls-remote origin` 验证“这个身份能否访问该仓库”。

如果前者成功、后者失败，应检查仓库地址和账号权限。

## 实践判断

- 只使用一个账号、网络限制较多：HTTPS 配合凭据管理器通常最简单。
- 需要稳定的无人值守同步：SSH 通常更合适。
- 同时使用公司和个人账号：为不同身份创建不同密钥，并显式指定使用哪一把。
- 安全的关键不是选择 HTTPS 还是 SSH，而是保护 Token 或私钥、限制权限范围，并能在泄露后及时撤销。

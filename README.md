# mihomo-config

可复用的 Mihomo 公共配置模板：设备节点与密钥留在本地，DNS、策略组和分流规则通过同一个主订阅更新。

**本仓库不提供 VPN 节点、密钥或免费代理。主配置单独导入不能使用，首次需配置自己的本地节点文件。**

## 主订阅地址

```text
https://raw.githubusercontent.com/sx3111212/mihomo-config/main/shared-profile.yaml
```

初次下载和之后更新均建议使用已经可用的代理。中国大陆直连 GitHub Raw 可能较慢或失败。
保留一份已验证可用的旧配置，先连接，再更新；不要把每次启动依赖于实时下载。

## 哪些内容会更新

| 内容 | 保存位置 | 更新方式 |
|---|---|---|
| 客户端私钥、服务端公钥、隧道 IP、节点 endpoint | 各设备本地节点文件 | 由设备拥有者维护 |
| DNS、策略组、`rules:`、规则源定义 | 本仓库主订阅 | 在客户端更新主订阅 |
| 社区中国域名/IP 清单 | MetaCubeX/meta-rules-dat | Mihomo 每 86400 秒检查更新 |
| 客户端自身的 TUN/系统代理开关与界面覆盖 | 各客户端设置 | 由客户端管理 |

所有使用者可以读取同一份公共主订阅，各设备仍使用自己的节点和密钥。
更新主订阅与更新 rule-provider 是两件事；主订阅的更新时间/代理方式在 App 中设置。

## 首次接入

1. 准备自己的本地节点文件，文件结构是 `proxies:` 下的节点列表。
2. 将它放到当前 Mihomo **HomeDir** 下的 `providers/local-wireguard.yaml`。
3. 保持现有连接可用，通过代理导入上面的主订阅地址。
4. 核对 `PROXY` 组能列出自己的节点，再手动选择新配置测试。
5. 在客户端设置主订阅定时更新，例如每日一次。

本地节点文件示意（下列全是占位符，不能直接连接）：

```yaml
proxies:
  - name: My-WireGuard
    type: wireguard
    server: YOUR_SERVER_HOST
    port: 51820
    ip: YOUR_ASSIGNED_CLIENT_IPV4
    private-key: YOUR_CLIENT_PRIVATE_KEY
    public-key: YOUR_SERVER_PUBLIC_KEY
    # 服务端要求预共享密钥时才填写：
    # pre-shared-key: YOUR_PRESHARED_KEY
    allowed-ips:
      - 0.0.0.0/0
    udp: true
    mtu: 1408
    persistent-keepalive: 25
```

节点名称不必与示例相同，`PROXY` 策略组通过本地代理集合导入节点。
各台直接连接 WireGuard 服务器的设备，应使用各自已在服务器登记的客户端密钥和隧道地址。

## Clash Verge Rev / Windows

默认 HomeDir 通常是：

```text
%APPDATA%\io.github.clash-verge-rev.clash-verge-rev
```

因此本地节点文件通常放在：

```text
%APPDATA%\io.github.clash-verge-rev.clash-verge-rev\providers\local-wireguard.yaml
```

以实际启动参数和 App 配置目录为准。这个路径是核心工作目录下的文件，不是导入订阅时的任意下载目录。
Clash Verge 的界面覆盖可能改变端口、TUN 和 DNS；应检查生成的实际配置。
主模板不主动开启 TUN，基线为 System / MTU 1400，是否启用由使用者确认后设置。
主模板不公开监听管理 API，不包含面板密码。

## 手机和路由器

Mihomo 内核支持 `type: file` 的代理集合，但各 App 的文件访问和订阅导入流程不同。
需要确认 App 能将本地节点文件放到其 HomeDir，并在订阅更新后保留它。
Android/iOS 的“下载”目录不等于核心工作目录，不能仅凭“支持 YAML”断言可直接使用。
手机 App 的具体部署未在本项目中完成实机验证。
此主模板不是 MCLite 等路由器插件的专用安装包；插件可能要求额外端口字段或重写配置，需独立适配。

## 分流和 DNS

- 已列出的 Google、YouTube、GitHub、OpenAI 等域名强制走 `PROXY`。
- 局域网域名与私有 IP 直连。
- 社区中国域名和中国 IP 规则命中时直连。
- 其他流量默认走 `PROXY`。
- DNS 使用 Fake-IP，指定的境外域名通过 `PROXY` 使用 Cloudflare DoH；国内解析使用国内解析器。
- DNS 中的 `geosite:cn` 策略仍依赖客户端已有 geosite 数据库，此数据库的更新需由客户端/插件管理。

社区分类可能有误判，不保证每个网站都符合个人偏好。
本地文件中的节点也不保证线路可用；此配置不承诺修复启动等待、丢包或吞吐问题。

## 维护与隐私

日常公共策略修改只编辑 `shared-profile.yaml`，使用者更新主订阅即可获得改动。
主订阅可以改变 DNS 和分流目标，使用者应信任维护者并保留已知可用配置。
不要把 `providers/local-wireguard.yaml`、私钥、预共享密钥、订阅 token 或完整私密客户端配置提交到公共仓库。
`.gitignore` 只是防误提交辅助，不能替代提交前检查。

## 参考与验证范围

- [Mihomo 代理集合](https://wiki.metacubex.one/config/proxy-providers/)
- [Mihomo 规则集合](https://wiki.metacubex.one/config/rule-providers/)
- [MetaCubeX 社区规则](https://github.com/MetaCubeX/meta-rules-dat)

发布前使用本机 Mihomo v1.19.33、测试用节点密钥及已下载的社区规则完成配置解析检查。
这不等于所有手机 App、路由器插件和真实网络的端到端验证。

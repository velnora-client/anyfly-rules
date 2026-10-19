# Nautilus Rules

Nautilus Rules 汇集域名与 IP 数据，生成代理内核使用的地理数据库和规则集。构建流程整理直连、代理、广告与服务类别，输出完整和精简数据，以及 Mihomo MRS 和 sing-box 规则文件。

## 获取数据

数据库和打包文件由 [.github/workflows/release.yml](.github/workflows/release.yml) 生成。工作流成功发布后，可在 [Releases](https://github.com/velnora-client/nautilus-rules/releases) 获取 `latest` 数据版本；使用前确认所需版本和文件已经存在。

| 格式 | 构建输出 |
| --- | --- |
| 国家 IP 数据 | `country.mmdb`、`country-lite.mmdb` |
| V2Ray 地理数据 | `geoip.dat`、`geoip-lite.dat`、`geosite.dat`、`geosite-lite.dat` |
| sing-box 数据库 | `geoip.db`、`geoip-lite.db`、`geosite.db`、`geosite-lite.db` |
| Meta IP 数据库 | `geoip.metadb`、`geoip-lite.metadb` |
| ASN 数据 | `GeoLite2-ASN.mmdb` |
| MRS 集合 | `BundleMRS.7z` |

构建会为发布文件生成 `.sha256sum`。下载时保留对应校验文件，在同一目录验证，例如：

```sh
sha256sum -c geosite.dat.sha256sum
```

生成的分支布局为：

- `release`：数据库、压缩包和校验文件。
- `meta`：Mihomo MRS 规则，包含 `geo/geosite`、`geo/geoip`、精简规则和 ASN。
- `sing`：sing-box 规则集，采用对应目录分类。

这些是工作流的输出目标；若当前尚无对应发布或分支，应先完成数据构建。`latest` 是滚动更新版本，内容会随上游数据变化。

## 在 Mihomo 中使用

将构建输出的中国域名规则 `geo/geosite/cn.mrs` 放到本地 `./provider/rule-set/geosite-cn.mrs` 后，可使用以下配置将匹配流量直连：

```yaml
rule-providers:
  cn:
    type: file
    behavior: domain
    format: mrs
    path: ./provider/rule-set/geosite-cn.mrs
rules:
  - RULE-SET,cn,DIRECT
  - MATCH,DIRECT
```

使用前确认文件存在且内核支持 MRS。需要 HTTP 自动更新时，将 provider 改为 `type: http`，配置实际可访问的数据 URL 和 `interval`；私有仓库的原始文件地址需要认证，不能直接当作匿名分发地址。完整 `geosite` 数据和精简数据的分类、覆盖范围不同，不能假设二者可无损互换。规则决定匹配和路由，不能保证服务解锁或替代实际网络验证。

## 数据与构建流程

[resouces](resouces) 保存本项目的增补与删除列表；目录拼写为源码中的实际名称。工作流拉取社区域名源，合并和去重，调用上游构建器生成 geosite，再转换 geoip/geosite 数据、生成规则文件与校验和，最后发布 Release 和数据分支。

主工作流每日 `22:30 UTC` 计划运行，也支持手动触发；计划是否实际执行取决于仓库 Actions 配置、权限和依赖服务。完整流程会访问多个外部数据源，使用 Go、Python、Bash、GNU 文本工具、`geo`、7-Zip 和 GitHub CLI，并需要发布权限。

本仓库不是 Go 或 Node 应用，没有统一的 `make build` 或测试入口。重建时应按工作流准备多个上游 checkout 和中间文件，不能直接在根目录运行 `go build`。

## 本地辅助脚本

[scripts/convert-clash.sh](scripts/convert-clash.sh) 从当前目录的 `geoip.dat`、`geosite.dat` 生成 Clash 列表和 YAML，需要当前目录存在可执行的 `./mosdns`：

```sh
bash scripts/convert-clash.sh
```

[scripts/convert.sh](scripts/convert.sh) 还使用当前目录的 `./sing-box` 编译 sing-box 规则。这些脚本假设输入和工具已经备齐，使用 GNU `sed`，并会写入或替换当前目录中的 `geoip/`、`geosite/` 内容；应在专用工作目录中运行。主发布流程以工作流中的转换器为准。

## 许可证与来源

仓库包含 [GPL-3.0 LICENSE](LICENSE)。汇集的数据和工具仍需遵循各自来源的许可证与署名要求，不能将仓库许可视为所有外部数据库的统一授权。

主要构建来源还包括 [MetaCubeX/meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter)、[MetaCubeX/geo](https://github.com/MetaCubeX/geo) 和 [xishang0128/geoip](https://github.com/xishang0128/geoip)。


## 致谢


- [@Loyalsoldier/geoip](https://github.com/Loyalsoldier/geoip)
- [@v2fly/domain-list-community](https://github.com/v2fly/domain-list-community)
- [@Loyalsoldier/domain-list-custom](https://github.com/Loyalsoldier/domain-list-custom)
- [@felixonmars/dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list)
- [@gfwlist/gfwlist](https://github.com/gfwlist/gfwlist)
- [@cokebar/gfwlist2dnsmasq](https://github.com/cokebar/gfwlist2dnsmasq)
- [@Loyalsoldier/cn-blocked-domain](https://github.com/Loyalsoldier/cn-blocked-domain)
- [@AdblockPlus/EasylistChina+Easylist.txt](https://easylist-downloads.adblockplus.org/easylistchina+easylist.txt)
- [@AdGuard/DNS-filter](https://kb.adguard.com/en/general/adguard-ad-filters#dns-filter)
- [@PeterLowe/adservers](https://pgl.yoyo.org/adservers)
- [@DanPollock/hosts](https://someonewhocares.org/hosts)
- [@crazy-max/WindowsSpyBlocker](https://github.com/crazy-max/WindowsSpyBlocker)
- [@blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)

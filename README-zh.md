# RealiTLScanner（修改版）

基于 [XTLS/RealiTLScanner](https://github.com/XTLS/RealiTLScanner) 的修改版，新增 REALITY 中转过滤与本机网页界面，并提供 Android 直装版。

> ⚠️ 免责声明：本工具仅供网络研究和授权测试使用，请遵守当地法律法规。扫描他人服务器属于敏感行为，请仅对你有权测试的网段使用。原作者 README 中的提醒依然有效：在云服务器上运行扫描可能导致 VPS 被主机商标记，建议在本地网络环境运行。

## 相比原版的改动

### 1. REALITY 中转过滤

扫描一个正在运行 REALITY 的服务器时，它会把未认证的握手转发给自己偷的 dest 站，扫描器拿到的证书其实是"别人的站"，污染 dest 候选列表。

本版对每个握手成功的结果做反查：**证书域名解析不到被扫 IP 的，判定为可疑（REALITY 中转 / 反代入口 / CDN 边缘），直接丢弃**。原理是 REALITY 对外表现为端口转发，被偷的站不可能解析回中转者的 IP；而套了 CDN 的站本来就不适合当 REALITY 的 dest（会成为免费的 CF 优选入口，很快烧光流量）。

- 命令行：`-filter=true`（默认开启），配 `-v` 可看到被过滤原因
- 网页模式：勾选"过滤 REALITY 中转"，实时统计里单独显示"已过滤"数量

已知局限：偷自己 / 偷邻居（域名确实解析回该 IP）的 REALITY 站无法识别——这在协议层无解，本工具的目标是让扫描结果干净，不是 100% 灭灯。

### 2. 网页模式（适合不想敲命令的人）

直接双击运行（不带任何参数）、或加 `-web` 参数，即在本机启动网页界面：

- 自动打开浏览器 `http://127.0.0.1:8080`（只监听 127.0.0.1，不对外）
- 文本框粘贴扫描目标：IP / CIDR 网段 / 域名，每行一个，可混合
- 可调端口、线程、超时；实时显示已扫描 / 命中 / 已过滤；一键停止；下载 CSV
- 命令行模式完整保留：带 `-addr` / `-in` / `-url` 参数时行为与原版一致

### 3. Android 版

`RealiTLScanner.apk`（arm64，debug 签名，直装）：

- WebView 壳 + 内嵌 Go 二进制（targetSdk 28 以允许应用在自身数据目录执行自带程序）
- 点开图标即是扫描界面，与电脑网页版完全一致
- 壳会把系统当前网络的 DNS 传给扫描器（安卓不向应用提供 /etc/resolv.conf，纯 Go 解析器在安卓上拿不到 DNS，这是本版特修的坑）
- 结果 CSV 保存到 `Android/data/com.reality.scanner/files/Download/`

## 用法速查

```
# 网页模式（推荐新手）
RealiTLScanner.exe            # 双击即可，浏览器自动打开 127.0.0.1:8080

# 命令行模式
RealiTLScanner.exe -addr 1.2.3.0/24 -thread 16 -timeout 3 -out result.csv
RealiTLScanner.exe -in targets.txt -thread 16
RealiTLScanner.exe -url "https://launchpad.net/ubuntu/+archivemirrors"
```

建议参数：单段 /24 用线程 16、超时 3 秒，约一分钟扫完；大段同样组合即可，不用更激进。

## 从源码构建

需要 Go 1.26+（国内可设 `GOPROXY=https://goproxy.cn,direct`）。

```
# Windows
go build -ldflags "-s -w" -o RealiTLScanner.exe .

# Android（产物放入 android/app/src/main/assets/scanner）
CGO_ENABLED=0 GOOS=android GOARCH=arm64 go build -trimpath -ldflags "-s -w" -o android/app/src/main/assets/scanner .
```

APK 构建见 `android/README.md`（需 JDK 17 + Android SDK，`gradle assembleDebug`）。

## 目录说明

- `scanner.go` / `dns_*.go` / `main.go` / `utils.go` / `geo.go` —— 扫描核心与入口
- `web.go` —— 内嵌网页界面（含 HTML/CSS/JS）
- `android/` —— Android WebView 壳工程

## 致谢

- [XTLS/RealiTLScanner](https://github.com/XTLS/RealiTLScanner) 原作者及 [Xray-core](https://github.com/XTLS/Xray-core) 社区
- GeoIP 功能需自行下载 MaxMind `Country.mmdb` 放到程序同目录（可选）

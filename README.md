# ipReverse

多源 IP 反查域名工具，支持批量查询、合并去重、来源标记、**代理转发**。

## 功能

- 多数据源查询（ip138 + hackertarget），结果自动合并去重
- 多源命中的域名标记所有来源
- 支持 IP、URL、域名混合输入，域名自动解析为 IP 后反查
- GOV/EDU 自动识别并高亮
- **支持代理转发（http/socks/带认证），分散出口 IP 防止被封**
- 全局限速，防止被源站封禁
- Ctrl+C 安全中断
- 默认输出 TXT，可选 XLSX

## 安装

```bash
pip install -r requirements.txt
```

> SOCKS 代理（`socks4://` / `socks5://`）需要 `PySocks`，已在 `requirements.txt` 中包含。

## 使用

```bash
# 单个目标
python3 ip_reverse.py -u 220.181.38.251

# 批量查询（默认输出txt）
python3 ip_reverse.py -l targets.txt

# 输出xlsx
python3 ip_reverse.py -l targets.txt -f xlsx -o result.xlsx

# 自定义线程和限速
python3 ip_reverse.py -l targets.txt -t 3 -r 3

# 走 HTTP 代理（防封）
python3 ip_reverse.py -l targets.txt -p http://127.0.0.1:8080 -r 1

# 走 SOCKS5 代理（带认证）
python3 ip_reverse.py -l targets.txt -p socks5://user:pass@127.0.0.1:1080 -r 0
```

## 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `-u, --url` | 单个目标（IP/URL/域名） | - |
| `-l, --list` | 批量文件（一行一个，自动跳过 `#` 注释行） | - |
| `-o, --output` | 输出文件名 | `ip_reverse_results.{fmt}` |
| `-f, --format` | 输出格式（txt/xlsx） | `txt` |
| `-t, --threads` | 线程数 | `5` |
| `-r, --rate` | 请求间隔（秒） | `2` |
| `-p, --proxy` | 代理地址，所有请求走该代理 | - |
| `-h, --help` | 显示帮助 | - |

## 代理使用（防封 IP）

源站（尤其 ip138）对单 IP 高频请求会临时封禁。使用代理可把出口 IP 换成代理服务器，大幅降低被封概率：

```bash
# HTTP / HTTPS 代理
python3 ip_reverse.py -l targets.txt -p http://127.0.0.1:8080

# 带认证的 HTTP 代理
python3 ip_reverse.py -l targets.txt -p http://user:pass@10.0.0.1:8080

# SOCKS5 代理（需 PySocks）
python3 ip_reverse.py -l targets.txt -p socks5://127.0.0.1:1080

# 不带协议时默认按 http 代理处理
python3 ip_reverse.py -l targets.txt -p 127.0.0.1:8080
```

代理格式：
- `http://[user:pass@]host:port`
- `https://[user:pass@]host:port`
- `socks4://[user:pass@]host:port`
- `socks5://[user:pass@]host:port`

**提示**：启用代理后，限速（`-r`）可适当调低（如 `-r 1` 或 `-r 0`）以提高速度，因为出口 IP 已由代理分散。

> 代理不可达或超时时，对应请求会按失败处理（计入来源统计），不影响其他目标。代理请求超时固定为 20s（`PROXY_TIMEOUT`）。

## 输出格式

### TXT（默认）

```
http://218.202.50.42    218.202.50.42    62    hackertarget    EDU
    cwxt.ljnu.edu.cn
    bkjxpg.lj-edu.cn
    ...
```

每行格式：`原始输入<TAB>IP<TAB>域名数<TAB>来源<TAB>标签`，下方逐行列出域名。

### XLSX

| 原始输入 | IP地址 | 关联域名 | 域名数 | 来源 | 标签 |
|----------|--------|----------|--------|------|------|

## 输入文件格式

支持纯 IP、URL、域名混合，`#` 开头为注释：

```
# 教育机构
http://cwxt.ljnu.edu.cn
http://218.202.50.42
https://cw.gzcmc.edu.cn:9999

# 注释行会被跳过
118.89.110.123
```

域名输入会自动解析为 IP 后执行反查。

## 数据源

| 源 | 说明 |
|----|------|
| ip138 | site.ip138.com，主力源 |
| hackertarget | api.hackertarget.com，备用源 |

ip138 临时封禁时自动降级到 hackertarget，恢复后无需修改代码。

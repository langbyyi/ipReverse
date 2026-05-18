# ipReverse

多源 IP 反查域名工具，支持批量查询、合并去重、来源标记。

## 功能

- 多数据源查询（ip138 + hackertarget），结果自动合并去重
- 多源命中的域名标记所有来源
- 支持 IP、URL、域名混合输入，域名自动解析为 IP 后反查
- GOV/EDU 自动识别并高亮
- 全局限速，防止被源站封禁
- Ctrl+C 安全中断
- 默认输出 TXT，可选 XLSX

## 安装

```bash
pip install -r requirements.txt
```

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
| `-h, --help` | 显示帮助 | - |

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

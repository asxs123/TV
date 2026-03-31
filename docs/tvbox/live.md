# 直播来源格式说明

LiveParser 根据内容自动检测格式，支持三种直播来源：

| 格式                 | 判断条件                          |
|--------------------|-------------------------------|
| [JSON](#json-数组格式) | 内容为 JSON（`[` 开头）              |
| [M3U](#m3u-格式)     | 任意行含 `#EXTM3U`（且不含 `#genre#`） |
| [TXT](#txt-格式)     | 其他                            |

---

## 目录

- [TXT 格式](#txt-格式)
- [M3U 格式](#m3u-格式)
- [JSON 数组格式](#json-数组格式)
- [频道指令](#频道指令)
- [DRM 声明](#drm-声明)
- [追看/时移](#追看时移)

---

## TXT 格式

每行以逗号 `,` 分为两列。含 `#genre#` 的行声明分组，含 `://` 的行为频道。

```
分组名称,#genre#
频道名称,播放URL
```

**行的解析规则：**

| 行的形式             | 说明                              |
|------------------|---------------------------------|
| `名称,#genre#`     | 声明新分组，后续频道归入此分组                 |
| `名称,URL`         | 频道项目（第二列含 `://`）                |
| `名称_密码,#genre#`  | 创建密码保护的隐藏分组                     |
| 不含 `://` 的行      | 频道[指令行](#频道指令)，作用至下一个 `#genre#` |
| 首个频道前无 `#genre#` | 自动创建无名默认分组                      |

**多线路备援**：以 `#` 分隔多个 URL，播放器依次尝试：

```
CCTV1,http://cdn1.example.com/cctv1.m3u8#http://cdn2.example.com/cctv1.m3u8
```

**行内标头**：URL 后接 `|key=value`，多个以 `&` 连接，多线路时每段各自带标头：

```
CCTV1,http://cdn1.example.com/cctv1.m3u8|User-Agent=okhttp#http://cdn2.example.com/cctv1.m3u8|Referer=https://example.com/
```

**示例：**

```
新闻台,#genre#
CCTV1,http://cdn1.example.com/cctv1.m3u8#http://cdn2.example.com/cctv1.m3u8
凤凰资讯,http://example.com/phoenix.m3u8|User-Agent=Mozilla/5.0

体育台,#genre#
ua=Mozilla/5.0
referer=https://sports.example.com/
CCTV5,http://example.com/cctv5.m3u8
CCTV5+,http://example.com/cctv5plus.m3u8

电影台,#genre#
header={"Authorization":"Bearer token123"}
HBO,http://example.com/hbo.m3u8

成人_secretpass,#genre#
某频道,http://example.com/adult.m3u8
```

---

## M3U 格式

以 `#EXTM3U` 开头，每个频道由 `#EXTINF:` 行及其后的 URL 行组成，中间可插入任意指令行。

```m3u
#EXTM3U [全局属性...]
#EXTINF:-1 [频道属性...],频道显示名称
[可选指令行...]
http://example.com/stream.m3u8[|行内标头]
```

### `#EXTM3U` 全局属性

| 属性                    | 说明                                       |
|-----------------------|------------------------------------------|
| `tvg-url="…"`         | XMLTV EPG 节目表 URL（仅当 Live 配置未设置 EPG 时生效） |
| `url-tvg="…"`         | 同 `tvg-url`，备用写法（同上条件）                   |
| `catchup="…"`         | 全局默认追看类型                                 |
| `catchup-source="…"`  | 全局追看 URL 模板                              |
| `catchup-replace="…"` | 全局追看 URL 替换字符串                            |

```m3u
#EXTM3U tvg-url="https://epg.example.com/xmltv.xml" catchup="append" catchup-source="?playseek=${(b)yyyyMMddHHmmss}-${(e)yyyyMMddHHmmss}"
```

### `#EXTINF` 频道属性

属性写在逗号前，频道显示名称写在逗号后至行尾。

| 属性                    | 说明                 |
|-----------------------|--------------------|
| `tvg-id="…"`          | EPG 频道 ID          |
| `tvg-name="…"`        | EPG 频道名称（可与显示名称不同） |
| `tvg-chno="…"`        | 频道号码               |
| `tvg-logo="…"`        | 频道 Logo URL        |
| `group-title="…"`     | 所属分组名称             |
| `http-user-agent="…"` | 播放请求 User-Agent    |
| `catchup="…"`         | 此频道追看类型，覆盖全局设定     |
| `catchup-source="…"`  | 此频道追看 URL 模板       |
| `catchup-replace="…"` | 此频道追看替换字符串          |

```m3u
#EXTINF:-1 tvg-id="CCTV1" tvg-name="CCTV-1" tvg-chno="1" tvg-logo="https://example.com/logo/cctv1.png" group-title="央视",CCTV-1
```

### M3U 专属指令行

在 `#EXTINF:` 与 URL 行之间插入，作用于紧接的下一个 URL（[通用指令](#频道指令)同样适用）。

| 指令                             | 说明                             |
|--------------------------------|--------------------------------|
| `#EXTHTTP:{"Key":"Value"}`     | JSON 格式 HTTP 标头                |
| `#EXTVLCOPT:http-user-agent=…` | VLC 风格 User-Agent              |
| `#EXTVLCOPT:http-referrer=…`   | VLC 风格 Referer                 |
| `#EXTVLCOPT:http-origin=…`     | VLC 风格 Origin                  |
| `#KODIPROP:…`                  | DRM 与媒体格式，详见 [DRM 声明](#drm-声明) |

**行内标头**：URL 末接 `|key=value&key2=value2`：

```m3u
http://example.com/stream.m3u8|User-Agent=Mozilla/5.0&Referer=https://example.com/
```

**示例：**

```m3u
#EXTM3U tvg-url="https://epg.example.com/xmltv.xml.gz" catchup="append" catchup-source="?playseek=${(b)yyyyMMddHHmmss}-${(e)yyyyMMddHHmmss}"

#EXTINF:-1 tvg-id="CCTV1" tvg-chno="1" tvg-logo="https://example.com/logo/cctv1.png" group-title="央视",CCTV-1
http://cdn1.example.com/cctv1.m3u8

#EXTINF:-1 tvg-id="CCTV5" group-title="央视",CCTV-5 体育
#EXTVLCOPT:http-user-agent=Mozilla/5.0
#EXTVLCOPT:http-referrer=https://sports.example.com/
http://cdn1.example.com/cctv5.m3u8|User-Agent=Mozilla/5.0

#EXTINF:-1 group-title="加密频道",Premium HD
#KODIPROP:inputstream.adaptive.license_type=widevine
#KODIPROP:inputstream.adaptive.license_key=https://license.example.com/widevine|User-Agent=Mozilla/5.0
format=mpd
http://example.com/premium.mpd

#EXTINF:-1 group-title="加密频道",PlayReady 频道
#KODIPROP:inputstream.adaptive.drm_legacy=playready|https://license.example.com/playready
http://example.com/playready.mpd

#EXTINF:-1 group-title="一般频道",需解析频道
parse=1
click=https://example.com/click
http://example.com/parse-needed.m3u8

#EXTINF:-1 group-title="带标头频道",自定义标头
#EXTHTTP:{"Authorization":"Bearer mytoken","X-Custom":"value"}
http://example.com/auth-stream.m3u8
```

---

## JSON 数组格式

内容以 `[` 开头时，直接反序列化为 `List<Group>`，结构与 Live 配置的 `groups` 字段相同。

```json
[
  {
    "name": "新闻台",
    "channel": [
      {
        "name": "TVBS新闻台",
        "urls": [
          "http://cdn1.example.com/tvbs.m3u8",
          "http://cdn2.example.com/tvbs.m3u8"
        ],
        "number": "56",
        "logo": "https://example.com/logo/tvbs.png",
        "tvgId": "TVBS",
        "ua": "Mozilla/5.0",
        "catchup": {
          "type": "append",
          "source": "?playseek=${(b)yyyyMMddHHmmss}-${(e)yyyyMMddHHmmss}"
        }
      }
    ]
  },
  {
    "name": "加密频道",
    "pass": "secretpass",
    "channel": [
      {
        "name": "Premium",
        "urls": [
          "http://example.com/premium.mpd"
        ],
        "format": "application/dash+xml",
        "drm": {
          "type": "widevine",
          "key": "https://license.example.com/widevine",
          "header": {
            "User-Agent": "Mozilla/5.0"
          }
        }
      }
    ]
  }
]
```

> 完整字段定义见 [CONFIG.md — channel 频道项目](CONFIG.md#channel--频道项目)。

---

## 频道指令

以下指令在 **M3U** 与 **TXT** 格式中均适用，写法完全一致。

- **TXT**：指令行写在频道行前，作用至下一个 `#genre#`（包含该分组内所有后续频道及其多线路 URL）。
- **M3U**：指令行写在 `#EXTINF:` 与 URL 行之间，仅作用于紧接的下一个 URL（每个 URL 处理后立即清除）。

| 指令          | 示例                             | 说明                                    |
|-------------|--------------------------------|---------------------------------------|
| `ua=`       | `ua=Mozilla/5.0`               | 播放请求 User-Agent                       |
| `origin=`   | `origin=https://example.com`   | 请求 Origin 标头                          |
| `referer=`  | `referer=https://example.com/` | 请求 Referer 标头（`referrer=` 双 r 写法同样接受） |
| `header=`   | `header={"X-Token":"abc"}`     | 任意 HTTP 标头（JSON 格式）                   |
| `format=`   | `format=mpd`                   | 强制指定媒体格式                              |
| `parse=`    | `parse=1`                      | `1` = 需通过解析器处理此 URL                   |
| `click=`    | `click=https://example.com/c`  | 点击拦截处理 URL                            |
| `forceKey=` | `forceKey=true`                | 强制使用 DRM 密钥                           |

**`format` 可选值：**

| 值              | 说明                                   |
|----------------|--------------------------------------|
| `hls`          | HLS 串流（`application/x-mpegURL`）      |
| `dash` 或 `mpd` | MPEG-DASH 串流（`application/dash+xml`） |

---

## DRM 声明

仅 M3U 格式支持，通过 `#KODIPROP:` 行声明，写在 `#EXTINF:` 与 URL 行之间。

| 指令                                                  | 说明                                           |
|-----------------------------------------------------|----------------------------------------------|
| `#KODIPROP:inputstream.adaptive.license_type=…`     | DRM 类型：`widevine` / `playready` / `clearkey` |
| `#KODIPROP:inputstream.adaptive.license_key=…`      | DRM 授权服务器 URL（或 ClearKey JSON）               |
| `#KODIPROP:inputstream.adaptive.drm_legacy=类型\|URL` | 快速声明，类型与授权 URL 合一                            |
| `#KODIPROP:inputstream.adaptive.manifest_type=…`    | 媒体格式：`mpd` / `dash` / `hls`                  |
| `#KODIPROP:inputstream.adaptive.stream_headers=…`   | 串流请求标头（`key=val&key2=val2`）                  |
| `#KODIPROP:inputstream.adaptive.common_headers=…`   | 通用请求标头（同上格式）                                 |

**授权服务器标头**：附加于 `license_key` URL 后，以 `|` 分隔：

```m3u
#KODIPROP:inputstream.adaptive.license_key=https://license.example.com/widevine|User-Agent=Mozilla/5.0&token=abc
```

**标准写法（Widevine）：**

```m3u
#EXTINF:-1 group-title="加密频道",Premium HD
#KODIPROP:inputstream.adaptive.license_type=widevine
#KODIPROP:inputstream.adaptive.license_key=https://license.example.com/widevine|User-Agent=Mozilla/5.0
format=mpd
http://example.com/premium.mpd
```

**快速写法（`drm_legacy`）：**

```m3u
#EXTINF:-1 group-title="加密频道",PlayReady 频道
#KODIPROP:inputstream.adaptive.drm_legacy=playready|https://license.example.com/playready
http://example.com/playready.mpd
```

**ClearKey（本地 JSON 密钥）：**

```m3u
#EXTINF:-1 group-title="加密频道",ClearKey 频道
#KODIPROP:inputstream.adaptive.license_type=clearkey
#KODIPROP:inputstream.adaptive.license_key={"keys":[{"kty":"oct","k":"base64key","kid":"base64kid"}],"type":"temporary"}
http://example.com/clearkey.mpd
```

**ClearKey（简短 hex 格式）：** 以 `kid_hex:key_hex` 表示，多对以逗号分隔，框架自动转换为标准 JSON：

```m3u
#KODIPROP:inputstream.adaptive.license_type=clearkey
#KODIPROP:inputstream.adaptive.license_key=a7e61c373e219033c21091c302f996c5:100b6c20940f779a4589152b57d2dacb
http://example.com/clearkey.mpd
```

**`stream_headers` 也支持 `drmScheme` / `drmLicense` 键名：**

```m3u
#KODIPROP:inputstream.adaptive.stream_headers=drmScheme=widevine&drmLicense=https://license.example.com/
```

---

## 追看/时移

在 `#EXTM3U`、`#EXTINF` 或 Live 配置的 `catchup` 字段中设置，频道层级设置覆盖全局。

**`type` 可选值：**

| 值         | 说明                            |
|-----------|-------------------------------|
| `append`  | 将 `source` 模板附加至原始 URL 末尾（默认） |
| `default` | 以格式化后的 `source` 完全替换原始 URL    |

> **PLTV 自动追看**：若频道未设置任何追看，但播放 URL 含有 `/PLTV/`，框架会自动套用内置的 PLTV 追看设置（`append` 类型，7 天，替换 `/PLTV/` → `/TVOD/`，模板 `?playseek=${(b)yyyyMMddHHmmss}-${(e)yyyyMMddHHmmss}`），无需手动设置。

**`source` 模板变量：**

| 变量                     | 说明               | 输出示例             |
|------------------------|------------------|------------------|
| `${(b)yyyyMMddHHmmss}` | 节目开始时间           | `20260312140000` |
| `${(e)yyyyMMddHHmmss}` | 节目结束时间           | `20260312150000` |
| `{utc:}`               | 节目开始 Unix 时间戳（秒） | `1741780800`     |
| `{utcend:}`            | 节目结束 Unix 时间戳（秒） | `1741784400`     |

**示例（`append` 类型）：**

```
catchup="append" catchup-source="?playseek=${(b)yyyyMMddHHmmss}-${(e)yyyyMMddHHmmss}"
```

**`catchup-replace` 用途**：

组合时移 URL 前，先对原始 URL 执行字符串替换。格式为 `原字符串,新字符串`（逗号分隔）：

```
catchup-replace="/PLTV/,/TVOD/"
```
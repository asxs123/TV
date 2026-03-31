# 本地 HTTP API

应用程序启动后会在本地绑定一个 HTTP 服务器，端口号从 **9978** 开始依次尝试至 **9998**，获取第一个可用端口。

```
http://127.0.0.1:{port}
```

> 实际端口号依系统可用情况而定，默认起始为 9978。

所有端点支持 GET 与 POST（除特别标注外），参数可放在 Query String 中。响应若无特别说明皆为 `text/plain`，成功回传 `OK`，失败回传 `500` 与错误信息。

---

## 目录

- [/action — 动作指令](#action--动作指令)
    - [do=control — 播放控制](#docontrol--播放控制)
    - [do=refresh — 刷新指令](#dorefresh--刷新指令)
    - [do=push — 推送播放](#dopush--推送播放)
    - [do=file — 打开文件](#dofile--打开文件)
    - [do=search — 触发搜索](#dosearch--触发搜索)
    - [do=setting — 加载配置](#dosetting--加载配置)
    - [do=cast — 投放媒体](#docast--投放媒体)
    - [do=sync — 同步数据](#dosync--同步数据)
- [/cache — 缓存操作](#cache--缓存操作)
- [/media — 播放状态](#media--播放状态)
- [/file — 本地文件系统](#file--本地文件系统)
- [/upload — 上传文件](#upload--上传文件)
- [/newFolder — 新建文件夹](#newfolder--新建文件夹)
- [/delFolder — 删除文件夹](#delfolder--删除文件夹)
- [/delFile — 删除文件](#delfile--删除文件)
- [/parse — 解析页面](#parse--解析页面)
- [/proxy — 爬虫代理](#proxy--爬虫代理)
- [/device — 设备信息](#device--设备信息)
- [端点总览](#端点总览)

---

## /action — 动作指令

通过 `do` 参数分派不同动作。

```
GET/POST http://127.0.0.1:9978/action?do={do}&...
```

---

### do=control — 播放控制

控制当前播放器的播放状态。

```
http://127.0.0.1:9978/action?do=control&type={type}
```

| 参数     | 说明        |
|--------|-----------|
| `type` | 控制指令，见下表。 |

**`type` 可选值：**

| `type`   | 说明       |
|----------|----------|
| `play`   | 播放       |
| `pause`  | 暂停       |
| `stop`   | 停止       |
| `replay` | 重新播放     |
| `prev`   | 上一集      |
| `next`   | 下一集      |
| `loop`   | 切换循环播放模式 |

**示例：**

```
http://127.0.0.1:9978/action?do=control&type=play
http://127.0.0.1:9978/action?do=control&type=pause
http://127.0.0.1:9978/action?do=control&type=next
```

---

### do=refresh — 刷新指令

触发应用程序重新加载指定页面数据，或推送内容至播放器。

```
http://127.0.0.1:9978/action?do=refresh&type={type}&...
```

**`type` 可选值：**

| `type`     | 额外参数   | 说明                                |
|------------|--------|-----------------------------------|
| `live`     | —      | 重新刷新直播页面。                         |
| `detail`   | —      | 重新刷新视频详情页。                        |
| `player`   | —      | 重新刷新播放页面。                         |
| `subtitle` | `path` | 推送字幕至当前播放器，`path` 为字幕文件 URL。       |
| `danmaku`  | `path` | 推送弹幕至当前播放器，`path` 为弹幕文件 URL。       |
| `vod`      | `json` | 推送 Vod 对象更新，`json` 为 Vod JSON 字符串。 |

**示例：**

```
http://127.0.0.1:9978/action?do=refresh&type=detail
http://127.0.0.1:9978/action?do=refresh&type=subtitle&path=http://example.com/sub.srt
http://127.0.0.1:9978/action?do=refresh&type=danmaku&path=http://example.com/danmaku.xml
```

---

### do=push — 推送播放

推送一个 URL 至应用程序进行播放。

```
http://127.0.0.1:9978/action?do=push&url={url}
```

| 参数    | 说明                    |
|-------|-----------------------|
| `url` | 要播放的媒体 URL（需 URL 编码）。 |

**示例：**

```
http://127.0.0.1:9978/action?do=push&url=http%3A%2F%2Fexample.com%2Fvideo.m3u8
```

---

### do=file — 打开文件

指定本地文件路径，依文件后缀名执行对应动作。

```
http://127.0.0.1:9978/action?do=file&path={path}
```

| 参数     | 说明         |
|--------|------------|
| `path` | 本地文件的绝对路径。 |

**依文件后缀名的行为：**

| 后缀名                  | 行为            |
|----------------------|---------------|
| `.apk`               | 触发 APK 安装流程。  |
| `.srt` `.ssa` `.ass` | 注入字幕至当前播放器。   |
| 其他                   | 触发设置页面打开对应文件。 |

---

### do=search — 触发搜索

在应用程序界面触发关键字搜索。

```
http://127.0.0.1:9978/action?do=search&word={word}
```

| 参数     | 说明     |
|--------|--------|
| `word` | 搜索关键字。 |

**示例：**

```
http://127.0.0.1:9978/action?do=search&word=%E9%A3%9F%E7%A5%9E
```

---

### do=setting — 加载配置

加载配置内容或指定名称的配置。

```
http://127.0.0.1:9978/action?do=setting&text={text}&name={name}
```

| 参数     | 说明             |
|--------|----------------|
| `text` | 配置内容字符串或配置 URL。 |
| `name` | 配置显示名称（选填）。    |

---

### do=cast — 投放媒体

将指定媒体投放至远程设备播放。

```
http://127.0.0.1:9978/action?do=cast&config={config}&device={device}&history={history}
```

| 参数        | 说明                                 |
|-----------|------------------------------------|
| `config`  | Config 对象的 JSON 字符串，指定要投放的配置。       |
| `device`  | 目标设备的 Device 对象 JSON 字符串（含 IP 等信息）。 |
| `history` | History 对象的 JSON 字符串，包含播放历史。        |

---

### do=sync — 同步数据

在多个设备间同步观看记录（`history`）或收藏列表（`keep`）。

```
POST http://127.0.0.1:9978/action?do=sync&type={type}&device={device}&force={force}&mode={mode}
```

**Query 参数：**

| 参数       | 说明                                         |
|----------|--------------------------------------------|
| `type`   | 同步类型：`"history"`（观看记录）或 `"keep"`（收藏）。      |
| `device` | 目标设备的 Device 对象 JSON 字符串。                   |
| `force`  | `"true"` = 先删除后合并；其他 = 直接合并。               |
| `mode`   | `"0"` = 双向（发送+接收）；`"1"` = 仅接收；`"2"` = 仅发送。 |
| `config` | （`history` 用）Config 对象 JSON 字符串。            |

**POST Body（`application/x-www-form-urlencoded`）：**

| 参数        | 说明                                |
|-----------|-----------------------------------|
| `targets` | History 或 Keep 对象数组的 JSON 字符串。     |
| `configs` | （`keep` 用）Config URL 数组的 JSON 字符串。 |

---

## /cache — 缓存操作

访问应用程序的键值缓存（基于 SharedPreferences），可用于爬虫在不同请求间共享数据。

```
GET/POST http://127.0.0.1:9978/cache?do={do}&...
```

**Key 计算规则：** `"cache_" + (rule 为空 ? "" : rule + "_") + key`

---

### do=get — 读取缓存

```
http://127.0.0.1:9978/cache?do=get&key={key}&rule={rule}
```

| 参数     | 说明                      |
|--------|-------------------------|
| `key`  | 缓存键名。                   |
| `rule` | 命名空间前缀，用于隔离不同爬虫的缓存（选填）。 |

**响应：** `200 OK`，回传存储的字符串值（若不存在则为空字符串）。

---

### do=set — 写入缓存

```
http://127.0.0.1:9978/cache?do=set&key={key}&value={value}&rule={rule}
```

| 参数      | 说明          |
|---------|-------------|
| `key`   | 缓存键名。       |
| `value` | 要存储的字符串值。    |
| `rule`  | 命名空间前缀（选填）。 |

---

### do=del — 删除缓存

```
http://127.0.0.1:9978/cache?do=del&key={key}&rule={rule}
```

| 参数     | 说明          |
|--------|-------------|
| `key`  | 要删除的缓存键名。   |
| `rule` | 命名空间前缀（选填）。 |

---

## /media — 播放状态

获取当前播放器的媒体信息与播放状态。

```
GET http://127.0.0.1:9978/media
```

**响应格式：** `application/json`

**播放器未启动时：**

```json
{}
```

**播放器启动时：**

```json
{
  "url": "https://cdn.example.com/video.m3u8",
  "state": 3,
  "speed": 1.0,
  "title": "示例电影",
  "artist": "来源名称",
  "artwork": "https://example.com/cover.jpg",
  "duration": 7200000,
  "position": 1234567
}
```

**字段说明：**

| 字段         | 类型        | 说明                                                       |
|------------|-----------|----------------------------------------------------------|
| `url`      | `string`  | 当前流 URL，无则为 `""`。                                       |
| `state`    | `integer` | PlaybackStateCompat 状态码。`1`=缓冲中，`2`=暂停，`3`=播放中，无则为 `-1`。 |
| `speed`    | `float`   | 播放速率（`1.0` = 正常速度），无则为 `-1`。                             |
| `title`    | `string`  | 媒体标题，无则为 `""`。                                           |
| `artist`   | `string`  | 艺术家或来源名称，无则为 `""`。                                       |
| `artwork`  | `string`  | 封面图 URI，无则为 `""`。                                        |
| `duration` | `long`    | 媒体总时长（毫秒），无则为 `-1`。                                      |
| `position` | `long`    | 当前播放位置（毫秒），无则为 `-1`。                                     |

---

## /file — 本地文件系统

浏览或下载应用程序的本地存储空间。

```
GET http://127.0.0.1:9978/file/{path}
```

| 参数     | 说明                      |
|--------|-------------------------|
| `path` | 相对于应用程序根目录的路径。省略时列出根目录。 |

**目录响应（JSON）：**

```json
{
  "parent": "videos",
  "files": [
    {
      "name": "movie.mp4",
      "path": "videos/movie.mp4",
      "time": "2025/03/05 12:00:00",
      "dir": 0
    },
    {
      "name": "subtitles",
      "path": "videos/subtitles",
      "time": "2025/03/05 10:00:00",
      "dir": 1
    }
  ]
}
```

**`files` 数组字段说明：**

| 字段     | 类型        | 说明                               |
|--------|-----------|----------------------------------|
| `name` | `string`  | 文件或目录名称。                         |
| `path` | `string`  | 相对于根目录的路径。                       |
| `time` | `string`  | 最后修改时间，格式 `yyyy/MM/dd HH:mm:ss`。 |
| `dir`  | `integer` | `1` = 目录，`0` = 文件。               |

**`parent` 字段说明：**

| 值                  | 意义             |
|--------------------|----------------|
| `"."`              | 当前即为根目录（无上一层）。 |
| `""`               | 上一层为根目录。       |
| `"path/to/parent"` | 上一层目录的相对路径。    |

**文件响应：** 直接流式传输文件内容，支持 Range 请求（`206 Partial Content`）与 ETag 缓存（`304 Not Modified`）。

---

## /upload — 上传文件

上传文件至指定目录，`.zip` 文件会自动解压缩。

```
POST http://127.0.0.1:9978/upload?path={path}
Content-Type: multipart/form-data
```

| 参数     | 说明               |
|--------|------------------|
| `path` | 目标目录，相对于应用程序根目录。 |

| 文件类型   | 行为                   |
|--------|----------------------|
| `.zip` | 解压缩至 `path` 目录。      |
| 其他     | 复制至 `path/filename`。 |

---

## /newFolder — 新建文件夹

在指定路径下创建新目录。

```
GET http://127.0.0.1:9978/newFolder?path={path}&name={name}
```

| 参数     | 说明                |
|--------|-------------------|
| `path` | 父目录路径，相对于应用程序根目录。 |
| `name` | 要创建的文件夹名称。        |

---

## /delFolder — 删除文件夹

删除指定目录及其所有内容。

```
GET http://127.0.0.1:9978/delFolder?path={path}
```

| 参数     | 说明                   |
|--------|----------------------|
| `path` | 要删除的目录路径，相对于应用程序根目录。 |

---

## /delFile — 删除文件

删除指定文件。

```
GET http://127.0.0.1:9978/delFile?path={path}
```

| 参数     | 说明                   |
|--------|----------------------|
| `path` | 要删除的文件路径，相对于应用程序根目录。 |

---

## /parse — 解析页面

将解析器脚本与目标 URL 嵌入 HTML 模板后回传，通常供 WebView 内部使用。

```
GET http://127.0.0.1:9978/parse?jxs={jxs}&url={url}
```

| 参数    | 说明            |
|-------|---------------|
| `jxs` | 解析器脚本标识符或内容。  |
| `url` | 待解析的媒体页面 URL。 |

**响应格式：** `text/html`，回传渲染后的 `parse.html` 页面。

---

## /proxy — 爬虫代理

将请求转发至爬虫的 `proxy()` 方法处理，供爬虫自定义响应（如转发流、修改标头等）。

```
GET/POST http://127.0.0.1:9978/proxy?...
```

所有 Query String 参数、请求标头与 POST Body 会合并后传入 `BaseLoader.get().proxy(params)`。响应由爬虫 `proxy()` 决定，框架原封不动地转发爬虫回传的流与标头。

爬虫如何实现 `proxy()` 方法及获取代理 URL，见 [SPIDER.md — 爬虫本地代理 URL](SPIDER.md#爬虫本地代理-url)。

---

## /device — 设备信息

获取本机设备信息。

```
GET http://127.0.0.1:9978/device
```

**响应格式：** `text/plain`，内容为设备信息的 JSON 字符串。

**响应字段：**

| 字段     | 类型       | 说明            |
|--------|----------|---------------|
| `name` | `string` | 设备显示名称。       |
| `ip`   | `string` | 设备局域网 IP 地址。 |

---

## 端点总览

| 端点                   | 方法       | 主要参数                              | 说明                                          |
|----------------------|----------|-----------------------------------|---------------------------------------------|
| `/action?do=control` | GET/POST | `type`                            | 播放控制（play/pause/stop/prev/next/loop/replay） |
| `/action?do=refresh` | GET/POST | `type`, `path`, `json`            | 刷新页面或推送字幕/弹幕                                |
| `/action?do=push`    | GET/POST | `url`                             | 推送 URL 播放                                   |
| `/action?do=file`    | GET/POST | `path`                            | 打开本地文件                                      |
| `/action?do=search`  | GET/POST | `word`                            | 触发关键字搜索                                     |
| `/action?do=setting` | GET/POST | `text`, `name`                    | 加载配置                                        |
| `/action?do=cast`    | GET/POST | `config`, `device`, `history`     | 投放媒体至远程设备                                   |
| `/action?do=sync`    | POST     | `type`, `device`, `force`, `mode` | 多设备数据同步                                     |
| `/cache?do=get`      | GET/POST | `key`, `rule`                     | 读取缓存值                                       |
| `/cache?do=set`      | GET/POST | `key`, `value`, `rule`            | 写入缓存值                                       |
| `/cache?do=del`      | GET/POST | `key`, `rule`                     | 删除缓存值                                       |
| `/media`             | GET      | —                                 | 获取播放状态 JSON                                 |
| `/file/{path}`       | GET      | —                                 | 浏览目录或下载文件（支持 Range）                         |
| `/upload`            | POST     | `path`（multipart）                 | 上传文件（支持 .zip 解压）                            |
| `/newFolder`         | GET      | `path`, `name`                    | 创建文件夹                                       |
| `/delFolder`         | GET      | `path`                            | 删除文件夹                                       |
| `/delFile`           | GET      | `path`                            | 删除文件                                        |
| `/parse`             | GET      | `jxs`, `url`                      | 获取渲染后的解析 HTML 页面                            |
| `/proxy`             | GET/POST | 自定义（转发至爬虫）                         | 爬虫代理转发                                      |
| `/device`            | GET      | —                                 | 获取设备信息                                      |
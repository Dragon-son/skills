---
name: 视频链接解析
description: 解析视频分享链接，获取无水印视频下载地址。当用户想要下载视频、解析抖音/快手/小红书/B站链接、获取无水印视频时使用此 skill。
category: tool
tags: [视频下载, 去水印, 抖音, 快手, 小红书, B站, 免费]
featured: false
---

# 视频链接解析

通过本地 `parse-video` HTTP 接口解析各大平台的视频分享链接，获取无水印视频下载地址。

## 工作流程

1. **解析链接** - 调用本地 HTTP 接口 `http://127.0.0.1:8080/video/share/url/parse?url=...`
2. **下载资源** - 如需保存到本地，再执行 `scripts/download.py`

## 接口要求

默认依赖本机运行中的 `parse-video` 服务：

- HTTP API: `http://127.0.0.1:8080`
- 解析接口: `GET /video/share/url/parse?url=<分享链接或整段分享文案>`

如果服务未启动，先启动本地 `parse-video` 容器或进程。

## 支持平台

- 抖音 (Douyin)
- 快手 (Kuaishou)
- 小红书 (Xiaohongshu)
- 哔哩哔哩 (Bilibili)
- 微博 (Weibo)
- 其他 parse-video 已支持的平台

## 使用方法

### 直接调用 HTTP 接口

```bash
curl -sS 'http://127.0.0.1:8080/video/share/url/parse?url=https://v.douyin.com/xxxx/'
```

如果用户给的是整段分享文案，也可以直接 URL encode 后传入。

### 返回格式

接口返回 JSON，典型结构如下：

```json
{
  "code": 200,
  "msg": "解析成功",
  "data": {
    "author": {
      "uid": "xxx",
      "name": "作者名",
      "avatar": "https://..."
    },
    "title": "视频标题",
    "video_url": "https://...mp4",
    "music_url": "https://...",
    "cover_url": "https://...",
    "images": []
  }
}
```

常用字段：

- `data.title` - 视频标题
- `data.video_url` - 无水印视频直链
- `data.music_url` - 音频链接（有时为空）
- `data.cover_url` - 封面图链接
- `data.images` - 图集内容（图集类型时使用）
- `data.author.name` - 作者名

## 示例

### 示例 1：解析抖音分享文案

```bash
python3 - <<'PY'
import urllib.parse, subprocess
text = '0.76 04/18 Y@z.gB Vlp:/ 几秒钟的世界 https://v.douyin.com/dEufUwHOAdg/ 复制此链接，打开Dou音搜索，直接观看视频！'
url = 'http://127.0.0.1:8080/video/share/url/parse?url=' + urllib.parse.quote(text, safe='')
subprocess.run(['curl', '-sS', url])
PY
```

### 示例 2：解析 B 站链接

```bash
curl -sS 'http://127.0.0.1:8080/video/share/url/parse?url=https://www.bilibili.com/video/BV1xx411c7mD'
```

### 示例 3：下载解析出的资源

先解析得到 `video_url` 或图片 URL，再调用下载脚本：

```bash
python3 scripts/download.py \
  --video "https://xxx.mp4" \
  --name "搞笑视频" \
  --output ~/Downloads/videos
```

## 常见问题

### Q: 解析失败怎么办？

1. 检查 `parse-video` 服务是否已启动
2. 检查分享链接或文案是否完整
3. 某些私密、删除或限地区内容可能无法解析
4. 部分平台可能临时变更接口，需等待上游修复

### Q: 支持图集吗？

支持。图集内容通常会出现在 `data.images` 字段中。

### Q: 需要 MCP 吗？

不需要。这个 skill 现在默认走本地 HTTP 接口，不再依赖 MCP。

## 下载脚本

脚本位置：

```text
scripts/download.py
```

脚本只负责下载，不负责解析。解析请先调用本地 HTTP API。

## 定价

免费，本地运行即可。

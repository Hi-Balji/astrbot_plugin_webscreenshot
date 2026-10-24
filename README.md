# astrbot_plugin_webscreenshot
一个能让你的bot实现网页截图功能的插件(代码是GLM写的)
<div align="center">

# 网页截图 · astrbot_plugin_webscreenshot

_✨ 被@时自动对链接生成网页截图,与「万能解析器」智能协同的 AstrBot 插件 ✨

![AstrBot](https://img.shields.io/badge/AstrBot-v4%2B-orange)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![License](https://img.shields.io/badge/License-MIT-green)

</div>

---

## 📖 简介

在群聊/私聊中 **@机器人** 并附带链接时,自动调用云端截图服务生成**网页截图**并发送。

本插件与 [astrbot_plugin_parser(万能解析器)](https://github.com/Zhalslar/astrbot_plugin_parser) 智能协同:

- **普通网页链接**(知乎、新闻、博客、GitHub 等解析器不认识的)→ 直接截图发送;
- **解析器负责的平台链接**(B站、抖音、小红书、微博等)→ 自动让路,由解析器发视频/图集;
- **解析器让路后没出结果**(Cookie 失效、平台风控、解析失败等)→ 等待数秒后**自动兜底截图**,不再出现"两个插件都不理你"的尴尬。

## ✨ 功能特性

- **@触发**:仅在@机器人(或引用消息后@)时工作,平时群里发链接不打扰;私聊始终触发
- **裸域名支持**:`www.baidu.com`、`zhihu.com/xxx` 等不带 `http://` 前缀的链接也能识别,自动补全协议
- **引用解析**:先发链接、再引用该消息并@机器人,同样能触发
- **域名让路**:内置解析插件负责的平台域名清单,自动避开、不抢活
- **失败兜底**:解析插件超时未出结果时自动接管,生成网页截图
- **双服务容灾**:主截图服务失败自动重试,仍失败切换备用服务再重试,单次超时可配
- **防刷屏**:同一会话同一链接冷却时间内不重复截图
- **强制截图**:`/截图 <网址>` 指令可绕过让路清单强制出图

## 📦 安装

### 方式一:WebUI 导入(推荐)

1. 获取 `astrbot_plugin_webscreenshot.zip`
2. AstrBot WebUI → **插件管理** → **从插件包安装** → 选择 zip 文件
3. 安装完成后重启/重载插件

### 方式二:手动安装

将插件文件夹放入 AstrBot 的 `data/plugins/` 目录,重启 AstrBot:

```
data/plugins/astrbot_plugin_webscreenshot/
├── main.py             # 插件逻辑
├── metadata.yaml       # 插件元数据
├── _conf_schema.json   # 配置项定义
└── requirements.txt    # 依赖(仅 aiohttp,AstrBot 自带)
```

> 💡 无需额外安装依赖,直接可用。

## 🚀 使用方法

| 场景 | 消息示例 | 行为 |
|------|---------|------|
| @机器人 + 普通网页链接 | `@机器人 看看这个 https://www.zhihu.com/question/xxx` | ✅ 回复网页截图 |
| 引用链接消息并@机器人 | 引用一条含链接的消息,回复 `@机器人 截一下` | ✅ 回复网页截图 |
| @机器人 + B站/抖音等链接 | `@机器人 https://www.bilibili.com/BV1xx411` | ⏸️ 让路,由解析插件发视频 |
| 解析插件没出结果 | 同上,但解析插件解析失败 | 🛟 等待数秒后自动兜底发截图 |
| 强制截图指令 | `/截图 https://www.bilibili.com/BV1xx411` | ✅ 无视让路清单,直接截图 |
| 只发链接不@ | `https://example.com`(不@) | ❌ 不触发 |
| 私聊发链接 | 私聊发送任意链接 | ✅ 直接截图 |

## ⚙️ 配置项

所有配置可在 **WebUI → 插件管理 → 网页截图** 中修改:

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `require_at` | bool | `true` | 是否需要@机器人才触发(私聊不受影响) |
| `cooldown` | int | `300` | 同一会话同一链接的截图冷却(秒) |
| `timeout` | int | `30` | 单次截图请求超时(秒) |
| `show_hint` | bool | `true` | 截图前是否发送"正在生成"提示 |
| `screenshot_api` | string | microlink | 主截图 API 模板,`{url}` 为目标链接占位符 |
| `backup_screenshot_api` | string | thum.io | 备用截图 API 模板,留空则不启用备用 |
| `retry_delay` | int | `2` | 每次失败后的重试间隔(秒) |
| `fallback_on_parser_fail` | bool | `true` | 解析插件让路后未出结果时是否兜底截图 |
| `fallback_wait` | int | `15` | 兜底前等待解析插件的时间(秒) |
| `skip_domains` | list | 见下文 | 让路域名清单,可自行增删 |

> 💡 截图**不落盘**:图片从内存直接发送,用完即弃,不占用手机存储。

### 让路域名默认清单

`bilibili.com` / `b23.tv` / `douyin.com` / `xiaohongshu.com` / `xhslink.com` / `weibo.com` / `x.com` / `twitter.com` / `pixiv.net` / `kuaishou.com` / `acfun.cn` / `qzone.qq.com` / `xiaoheihe.com` 等。

解析插件日后支持新平台时,在配置里把新域名加进 `skip_domains` 即可。

## 🔀 与解析插件的协同逻辑

```
@机器人 + 链接
     │
     ├─ 普通网页域名 ──────────────→ 立即截图发送
     │
     └─ 解析器负责的域名(让路)
            │
            └─ 后台等待 fallback_wait 秒
                   │
                   ├─ 期间解析插件发出了视频/语音 → 安静退场
                   │
                   └─ 没有结果(解析失败) → 兜底截图发送
```

> ⚠️ 判定"解析成功"的依据是**该消息收到了视频或语音**。图集(纯图片)场景不参与判定;若解析插件下载大视频超过等待时间,可能出现"截图先到、视频后到"的情况,可将 `fallback_wait` 调大。

## 🌐 截图服务说明

| 服务 | 模板 | 说明 |
|------|------|------|
| [microlink](https://microlink.io)(默认主服务) | `https://api.microlink.io/?url={url}&screenshot=true&embed=screenshot.url&screenshot.timeout=30000` | 渲染质量好,免费额度约每天数十次 |
| [thum.io](https://thum.io)(默认备用) | `https://image.thum.io/get/width/800/{raw_url}` | 无需注册,速度较慢 |
| WordPress mShots(可选替换) | `https://s.wordpress.com/mshots/v1/{url}?w=800` | 免费无需注册,可作为主/备替换 |

模板占位符:`{url}` 会被替换为 **URL编码后** 的目标地址(query 参数型服务用,如 microlink);`{raw_url}` 替换为**原始**地址(路径拼接型服务用,如 thum.io)。两者均无需 API Key。若国内访问不畅或额度耗尽,可替换为任意"模板式"截图服务(只要 GET 请求直接返回图片)。

## ❓ 常见问题

**Q: 发了链接为什么没反应?**
确认是否@了机器人(私聊除外);确认链接不在让路清单中;查看是否处于冷却时间。

**Q: B站链接解析失败,截图也没出?**
兜底默认开启,等待 `fallback_wait` 秒(默认15秒)后才会出截图,请稍候;若解析插件发送了失败提示文字但长期不出图,可把日志反馈给插件作者。

**Q: QQ 把链接转成卡片消息能识别吗?**
纯文本链接识别最可靠;部分卡片消息文本中不含 URL,插件无法感知(此类多为B站分享,本就由解析插件负责,影响不大)。

**Q: 截图返回了一张白图?**
百度系(贴吧等)等网站对海外截图服务有反爬,可能渲染为空白页。插件已内置**空白页检测**:检测到白图会自动视为失败,重试并切换备用服务;若主备均渲染失败,会回复明确报错。此类站点属于截图服务的限制,暂无完美解法。

**Q: 截图很慢?**
云端渲染需要时间,属正常现象;插件已内置"主→重试→备用→重试"的完整容灾链路,单次超时默认 60 秒。

## 📜 许可

[MIT License](https://opensource.org/licenses/MIT) © Balji

数据均来自第三方截图服务(microlink.io / thum.io),本插件不存储任何网页数据。

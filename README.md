# MorePlay

MorePlay 是面向 Docker 和 NAS 的影视源多协议桥接服务。添加自己的 CatPawOpen 猫源或兼容的影视仓 / TVBox 输入后，可以在 Forward / Rex、Emby、STRM 媒体库和 T4 播放器中浏览、搜索与播放。

> 影视目录和播放资源均来自用户配置的站源。请使用可信的来源。

本文的功能说明按 **3.1** 整理。当前 Docker Hub 正式镜像为 **3.0**，支持 `linux/amd64` 和 `linux/arm64`。可用镜像见 [Docker Hub 标签](https://hub.docker.com/r/mobai1231/catpawopen-fw/tags)。

[核心功能](#核心功能) · [Docker / NAS 部署](#docker--nas-部署) · [首次使用](#入口与首次配置) · [各协议使用](#各协议使用) · [更新与备份](#数据目录与升级) · [问题反馈](#问题反馈)

## 3.1 更新日志

修复了许多bug，增加了许多功能，请自行探索

## 核心功能

- **统一管理站源**。添加多个猫源、兼容影视仓、单站 T4 或直播输入，并为各输入单独配置代理。
- **Forward / Rex**。订阅模块浏览站源分类，按设置聚合匹配播放线路。
- **Emby 兼容服务**。提供多账号、浏览目录、预热、线路选择、TMDB 资料补全和视频参数探针。
- **STRM**。支持自动同步、手动入库、订阅、TMDB 资料、动态 STRM 下载和后台视频下载。
- **T4**。向 OK 影视等兼容播放器提供订阅。分类浏览和搜索按站点独立提供，分集按原线路显示。
- **播放诊断**。在管理台查看播放失败原因、站源状态和运行活动。

## Docker / NAS 部署

在支持 Docker Compose 的 NAS 或电脑上，将下面的内容保存为 `compose.yaml`。先替换管理密码，再运行 `docker compose up -d`。

```yaml
services:
  moreplay:
    image: mobai1231/catpawopen-fw:latest
    container_name: moreplay
    restart: unless-stopped
    ports:
      - "2333:2333"
    environment:
      TZ: Asia/Shanghai
      CATPAW_SETTINGS_PASSWORD: "请替换为至少8位的管理密码"
    volumes:
      - moreplay-data:/data
      - ./strm-library:/strm

volumes:
  moreplay-data:
```

- `latest` 跟随 Docker Hub 上的正式镜像。要固定版本，可改用已经发布的 `mobai1231/catpawopen-fw:3.0`。
- `2333:2333` 左侧是宿主机端口。修改后，管理台与播放器地址都使用新的宿主机端口。
- NAS 上可以将 `./strm-library` 换成允许媒体库读取的宿主机目录。
- 猫源可以在管理台添加。需要部署时指定，可在 `environment` 中填写 `CATPAW_SOURCE_URL`。

## 入口与首次配置

以 `http://<宿主机地址>:2333` 为基础地址，打开 `/bridge` 并使用部署时配置的管理密码登录。

| 功能 | 路径 |
| --- | --- |
| 管理台 | `/bridge` |
| 站源管理 | `/bridge/sites` |
| Forward / Rex 管理 | `/bridge/fw` |
| Forward / Rex 模块订阅 | `/bridge/widget.js` |
| Emby 管理 | `/bridge/emby` |
| STRM 管理 | `/bridge/strm` |
| T4 管理 | `/bridge/t4` |
| T4 播放器订阅 | `/t4/config.json` |
| 播放诊断 | `/bridge/playback-diagnostics` |
| 日志 | `/bridge/logs` |

1. 在**站源管理**添加可信猫源或影视仓。在线猫源一般由作者提供以 `index.js.md5` 结尾的地址，也可以在管理台导入本地四文件。
2. 检查站点状态。需要网盘账号的站源，从对应猫源卡片打开设置页配置账号。多个猫源的设置分别保存。
3. 启用需要的协议，并在各协议页面选择浏览或搜索范围。全局可用范围由站源管理统一维护。
4. 从协议页面复制对应地址，在播放器中添加。Emby 和 STRM 服务器订阅使用各自页面配置的账号。

## 各协议使用

### Forward / Rex

在播放器导入 `/bridge/widget.js`。到 MorePlay 的 **Forward / Rex** 页选择浏览站点和资源匹配范围。播放器支持内置 TMDB 时，可开启模块的 **TMDB 资料与海报**。

### Emby

在 **Emby** 页开启服务并设置账号。播放器填写 `http://<宿主机地址>:2333`，登录后浏览。可先完成首页和分类预热，再打开播放器查看。需要刷新目录时，可按首页、站点或分类选择更新范围。

在 **TMDB 海报与简介** 中配置 TMDB 连接，测试通过后选择 MorePlay 刮削或由支持的播放器自行刮削。MorePlay 可交付影片海报、简介、分集剧照、演员与相关影片，资料缺失时按字段回退。首页和分类使用猫源封面，封面读取失败时依次使用 TMDB 和兜底图。

在 **视频参数探针** 中设置开关和并发数。探测成功后，支持的播放器可显示分辨率、帧率、色彩、音轨等参数。

### STRM

在 **STRM** 页开启服务。选择入库展示站点后可以同步分类，选择搜索发现站点后可以手动入库或订阅。把 `/strm` 对应的宿主机目录加入媒体库扫描。生成的 `.strm` 文件由 MorePlay 按需解析播放。

- **对外访问地址**填写媒体服务器或播放器实际可达的 MorePlay 地址。留空会采用保存设置时的管理入口。
- **动态 STRM**可在媒体库按当前访问地址下载，也可以复制链接给支持 HTTP STRM 的客户端。
- **影视订阅**可以按更新周期自动补集。支持 MoviePilot 的播放器可通过 STRM 页的服务器订阅设置连接。

### T4 / TVBox

开启 T4 后，将 `/t4/config.json` 导入 OK 影视等客户端。T4 提供独立的站点入口，播放器搜索或换源时可查询配置允许的站点。使用支持 VOD 接口的 VidPlay 时，可在 T4 页面复制专用订阅地址。

### 播放链路

在系统设置选择直连或 NAS 转发。直连要求播放器能访问实际媒体地址并支持该线路所需的请求头。NAS 转发由 MorePlay 取流，适合依赖源站请求头或局域网访问的线路。STRM 在自己的**播放设置**中配置自动或 NAS 转发。

## 数据目录与升级

| 容器目录 | 用途 |
| --- | --- |
| `/data` | 配置、账号、缓存和索引 |
| `/source` | 使用本地猫源时的文件目录 |
| `/strm` | STRM、NFO、图片及下载的媒体 |

更新镜像前备份 `/data`、`/strm` 及使用中的本地猫源目录。保留原端口和挂载，在 Compose 目录执行：

```bash
docker compose pull
docker compose up -d
```

## 常见问题

**登录失败怎么办？** 检查部署配置中的 `CATPAW_SETTINGS_PASSWORD`。该变量配置的密码随容器设置生效。已有数据目录的登录配置会随 `/data` 保留。

**容器已启动，但站源打不开？** 在站源管理查看站点扫描结果、代理和源地址。`/check` 的 `sourceReady` 可用于确认源是否就绪。

**有线路但播放失败？** 到播放诊断核对失败原因，并检查对应网盘的登录配置、线路有效性和播放模式。反馈时提供作品、站点、线路、播放器及尝试时间即可。

**STRM 内网能播，外网一直加载？** 检查实际读取 `.strm` 或代为取流的设备能否访问文件中的 MorePlay 地址。外网入口变化时可通过当前入口重新下载动态 STRM。

**为什么有些资料和图片来自猫源？** TMDB 缺少对应资料，或季集暂时无法可靠匹配时，会继续显示可用的猫源资料。分集简介优先采用中文，缺少中文时可用英文。

## 问题反馈

点击 WebUI 右上角的 GitHub 图标可打开反馈主题。提交问题时请附版本、设备、播放器、站点、作品及必要的脱敏诊断信息。 [MorePlay 论坛反馈](https://club.ugnas.com/forum.php?mod=viewthread&tid=6122)

## 3.0 更新日志

1. 修复 STRM 部分分类未正确扫描的问题。
2. 修复 T4 会话过期且不会自动重连的问题。
3. 修复模块聚合搜索刷新后未覆盖旧结果的问题。
4. 修复 Emby 在 Rex 上未正确显示站源名 + 线路名的问题。
5. 新增 Emby 支持使用 TMDB 刮削海报。
6. GitHub 账号已被封禁两次，待申诉成功后恢复 GitHub 发布；问题反馈请先前往论坛，点击右上角 GitHub 图标即可跳转。

## 2.9.4 更新日志

1. 修复 Emby 需要清除播放器缓存才能重新读取封面的错误逻辑。
2. 修复 AfuseKt 播放器连接不上的问题。
3. 因 GitHub 账号被未知原因封禁，已更换 GitHub 地址。

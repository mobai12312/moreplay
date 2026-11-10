# MorePlay

<p align="center">
  <img src="docs/logo-moreplay.png" alt="MorePlay Logo" width="160">
</p>

**NAS / Docker 影视源多协议桥接服务 · 从安装到播放器接入的新手教程**

MorePlay 可以将你自行添加的 **CatPawOpen 猫源、兼容的影视仓 / TVBox 输入**，整理为不同客户端可使用的服务。只需要在 NAS 或 Linux 服务器上部署一次，就可以按需使用 **Emby、Forward / Rex、STRM、T4** 等功能。

> **请先了解：** MorePlay 不是播放器，不自带影视资源，也不会因为安装成功就自动出现影片。目录与播放线路来自你配置的第三方站源，请自行选择可信、合法的数据源。

**支持环境：** Docker Compose；镜像支持 `linux/amd64`、`linux/arm64`。以下以 NAS 部署为例，普通 Linux 服务器也可以使用。

**版本提示：** 本文整合了仓库中按 **3.1** 编写的功能介绍和新手部署说明。镜像版本以 [Docker Hub 标签列表](https://hub.docker.com/r/mobai1231/catpawopen-fw/tags) 为准；不同版本的功能入口可能略有差异。

[界面预览](#界面预览) · [快速安装](#一5-分钟安装-moreplay) · [添加站源](#二首次登录与配置站源) · [连接播放器](#三连接播放器选择适合自己的功能) · [高级功能](#四按需使用的进阶功能) · [更新备份](#五更新与备份) · [常见问题](#六常见问题) · [更新说明](#七版本更新说明)

## MorePlay 能做什么？

| 功能 | 适合谁 | 主要能力 |
| --- | --- | --- |
| **Forward / Rex** | 使用对应模块的播放器 | 导入模块、分类浏览、搜索和匹配播放线路 |
| **Emby** | 使用 Emby 兼容客户端 | 账号登录、目录浏览、预热、TMDB 资料、播放线路与视频参数探针 |
| **STRM** | 维护本地媒体库的用户 | 生成 `.strm` / `.nfo`、同步目录、订阅与入库、按需解析播放 |
| **T4 / TVBox** | 使用 OK 影视等兼容客户端 | 提供 T4 配置、按站点浏览与搜索 |
| **站源管理与播放诊断** | 所有用户 | 统一管理站源、各站点单独代理、排查播放失败与查看日志 |


## 界面预览

<p align="center">
  <img src="docs/screenshots/dashboard-overview.webp" alt="MorePlay 管理台概览" width="100%">
</p>

<p align="center"><sub>管理台概览</sub></p>

<table>
  <tr>
    <td width="50%" align="center">
      <img src="docs/screenshots/sites-management.webp" alt="MorePlay 站源管理" width="100%">
      <br><sub>站源管理</sub>
    </td>
    <td width="50%" align="center">
      <img src="docs/screenshots/services-management.webp" alt="MorePlay 媒体服务" width="100%">
      <br><sub>媒体服务</sub>
    </td>
  </tr>
  <tr>
    <td width="50%" align="center">
      <img src="docs/screenshots/emby-management.webp" alt="MorePlay Emby 管理" width="100%">
      <br><sub>Emby 兼容服务</sub>
    </td>
    <td width="50%" align="center">
      <img src="docs/screenshots/strm-management.webp" alt="MorePlay STRM 媒体库" width="100%">
      <br><sub>STRM 媒体库</sub>
    </td>
  </tr>
</table>

## 一、5 分钟安装 MorePlay

### 1. 安装前准备

准备一台已安装 **Docker / Docker Compose** 的 NAS 或服务器，并确认：

- 知道 NAS 的局域网 IP，例如 `192.168.1.10`（仅为示例）。
- 能在 NAS 上创建容器项目，或通过终端执行 Docker 命令。
- 有可信的猫源地址更方便测试；没有也可以先部署，稍后在管理台添加。

### 2. 创建 Compose 项目

在 NAS 的 Docker 应用中找到 **「项目 / Compose / 新建项目」**（各品牌名称可能不同），项目名称可填 `moreplay`。粘贴以下配置；命令行部署时保存为 `compose.yaml`。

```yaml
services:
  moreplay:
    image: mobai1231/catpawopen-fw:latest
    container_name: moreplay
    restart: unless-stopped
    ports:
      - "2333:2333"
    environment:
      #CATPAW_SETTINGS_PASSWORD: "请修改为你自己的管理密码，不设置的话可以删除这行，默认密码88888888"
      # 如需部署时指定在线猫源，可替换下方地址，也可以部署后在管理台添加或修改
      CATPAW_SOURCE_URL: "https://9280.kstore.vip/cat/index.js.md5"
    volumes:
      - moreplay-data:/data
      - ./strm-library:/strm

volumes:
  moreplay-data:
```

**第一次只需要修改管理密码。** 猫源可以登录后台后再添加。

| 配置 | 作用 | 要不要修改 |
| --- | --- | --- |
| `mobai1231/catpawopen-fw:latest` | 从 Docker Hub 拉取正式镜像 |想固定版本可选用 Docker Hub 上实际存在的标签，例如 `:3.0` |
| `2333:2333` | 左侧为 NAS 端口，右侧为容器端口 | 有端口冲突时只改左侧，例如 `2233:2333` |
| `CATPAW_SETTINGS_PASSWORD` | MorePlay 管理台登录密码 | 
| `CATPAW_SOURCE_URL` | 可选的在线猫源入口 | 没有就保持注释，后续到后台添加 |
| `moreplay-data:/data` | 保存配置、账号、缓存和索引 | 建议保持不变 |
| `./strm-library:/strm` | 保存 STRM、NFO、图片以及相关文件 | 需要给媒体库读取时可替换为实际宿主机目录 |

> 已使用旧版 Compose 的用户，请留意旧示例可能使用 `catpaw-open-bridge` 和 `catpaw-data`。**升级旧部署时不要直接改数据卷名称**，否则会出现看似“配置丢失”的情况。迁移前先备份并确认原挂载。

### 3. 启动容器

点击 NAS 项目页面的 **部署 / 启动**；或在 `compose.yaml` 所在目录执行：

```bash
docker compose up -d
```

第一次运行会拉取镜像。容器状态显示 **运行中** 后，在浏览器打开：

```text
http://你的NAS地址:2333/bridge
```

例如：

```text
http://192.168.1.10:2333/bridge
```

如果改过左侧端口，就把地址中的 `2333` 换成你设置的新端口。

### 4. 登录管理台

输入 Compose 中设置的 `CATPAW_SETTINGS_PASSWORD`。**管理台密码不是 Emby 播放账号密码。**

如果没有配置该环境变量，全新安装版本可能使用默认密码 `88888888`。

## 二、首次登录与配置站源

### 1. 添加或检查输入源

打开 **「站源管理」**（`/bridge/sites`），添加你自己的可信输入源。可以使用在线猫源地址、兼容的影视仓 / TVBox HTTP 输入，单站 T4、直播输入以及本地四文件猫源。

#### 推荐猫源（第三方维护）

以下地址由第三方作者提供，更多配置和可用性以站源实际情况为准。

| 名称 | 猫源地址 |
| --- | --- |
| 牛源 | `https://9280.kstore.vip/cat/index.js.md5` |
| 豆源 | `https://woleigedouer:woleigedouer@catpaw.douer.me/index.js.md5` |

> **提示：** 豆源地址包含用户名和密码，请复制完整地址；请勿把它误认为 MorePlay 的管理账号。

如果站源需要网盘账号、Cookie 或其他参数，进入**该站源自己的设置页面**填写。不同猫源的配置分别保存，无需在每个模块重复登录。

> **新手建议：** 第一次只添加一个源，依次检查分类、搜索、详情和播放。源越多，搜索与预热可能越慢。部分 TVBox 站点依赖额外的 Spider 环境，不能保证仅粘贴 URL 就能使用。

### 2. 启用站点，并选择模块范围

先确认站源管理中的站点状态正常，再进入想使用的协议页面选择站点并保存。

**要注意：** 全局可用站点由站源管理统一维护，但 **Emby、Forward / Rex、STRM、T4 各自仍有浏览 / 搜索范围设置**。添加一个站源，不代表它会自动加入所有模块。

### 3. 快速找到管理入口

以下路径均以 `http://你的NAS地址:2333` 为基础；更换对外端口时同步替换。

| 用途 | 路径 |
| --- | --- |
| 管理台首页 | `/bridge` |
| 站源管理 | `/bridge/sites` |
| Forward / Rex 设置 | `/bridge/fw` |
| Emby 设置 | `/bridge/emby` |
| STRM 设置 | `/bridge/strm` |
| T4 设置 | `/bridge/t4` |
| 播放诊断 | `/bridge/playback-diagnostics` |
| 日志 | `/bridge/logs` |

## 三、连接播放器：选择适合自己的功能

| 你的使用场景 | 使用功能 | 需要导入的地址 |
| --- | --- | --- |
| 用 Emby 兼容客户端浏览、播放 | **Emby** | `http://你的NAS地址:2333` |
| 用 Rex / Forward 订阅模块 | **Forward / Rex** | `http://你的NAS地址:2333/bridge/widget.js` |
| 给其他媒体库创建 `.strm` | **STRM** | 在后台生成文件，**不是统一播放器订阅 URL** |
| 用 OK 影视等兼容客户端 | **T4** | `http://你的NAS地址:2333/t4/config.json` |

### A. Emby：适合想用媒体库界面的人

1. 在管理台打开 **Emby**，启用服务并设置**播放账号、密码**。
2. 选择需要浏览的站点并保存；第一次使用可以先等待首页与分类**预热**完成。
3. 打开 Emby 兼容客户端，服务器地址填写：

   ```text
   http://你的NAS地址:2333
   ```

4. 用 **Emby 页面设置的账号和密码** 登录，测试目录、详情和播放。

**容易填错的地方：** Emby 服务器使用根地址，**末尾不要加 `/bridge`**。预热只是预先准备目录与封面，不是下载电影。

更多可选设置：

- **TMDB 海报与简介：** 在 Emby 对应页面配置连接、测试、选择刮削方式。可补充作品简介、海报、演员、分集等内容；缺失资料会回退到可用的猫源信息。首页和分类图通常仍优先使用猫源封面。
- **视频参数探针：** 对应版本提供探针开关与并发配置，探测成功后，兼容客户端可显示分辨率、帧率、色彩及音轨等信息。
- **目录刷新：** 如果播放器仍显示旧列表，可以按首页、站点或分类范围刷新，避免重复扫描全部内容。

### B. Forward / Rex：适合使用模块订阅的人

1. 进入 **Forward / Rex** 页面，启用模块并设置浏览站点、资源匹配范围。
2. 在兼容的 Rex / Forward 播放器中导入：

   ```text
   http://你的NAS地址:2333/bridge/widget.js
   ```

3. 在播放器里测试分类、搜索、匹配线路和播放。

该模块站点范围与 Emby、STRM 独立。

### C. STRM：适合已有媒体库的人

1. 在 **STRM** 页面启用服务，选择**入库展示站点**与**搜索发现站点**。
2. 配置 **对外访问地址**，填入真正负责读取 / 播放 `.strm` 的设备能访问的 MorePlay 地址，例如 `http://192.168.1.10:2333`。
3. 按需要使用**分类同步、手动入库、影视订阅**或动态 STRM 功能。
4. 将 `./strm-library` 对应的宿主机文件夹加入你的媒体库扫描目录。

STRM 可生成 `.strm`、`.nfo`、图片等文件。**`.strm` 是播放指针，不是视频本体**：正常生成文件不会自动下载整部电影；后台视频下载是另一项按需启用的能力。

支持的版本还提供自动补集订阅、动态 STRM 下载，以及与 MoviePilot 等工具配合的服务器订阅。具体账号、目录与订阅设置请以 STRM 页面为准。

### D. T4 / TVBox：适合 OK 影视等兼容播放器

1. 在 **T4** 页面启用服务、选择允许的站点并保存。
2. 在兼容客户端导入：

   ```text
   http://你的NAS地址:2333/t4/config.json
   ```

3. 测试是否可以读取站点、浏览分类、搜索和播放。

T4 按原站点组织浏览与分集线路，支持 VOD 接口的 VidPlay 提供专用订阅地址，实际地址请从 **T4 管理页面复制**，不要直接用上面的通用地址替代。

## 四、按需使用的进阶功能

### 直连 / 302 与 NAS 转发怎么选？

- **直连 / 302：** 播放器客户端直接访问最终视频链接，通常可减少 NAS 的流量和带宽占用，但播放器客户端必须支持线路所需的 URL、跳转、请求头等。
- **NAS 转发：** MorePlay 在 NAS 上取流，再提供给播放器；部分客户端不支持直连或站源需要特殊请求头时更适合。

建议先按默认设置测试稳定播放，再根据客户端兼容性切换。**不是所有 Emby 客户端都能对所有网盘线路实现直连。** STRM 的自动 / 转发设置在自己的播放设置中调整。

### 站源代理、播放诊断与日志

对需要特殊网络环境的输入源，可在站源管理中为相应输入单独配置代理。若出现封面加载失败、搜索超时、无线路或无法播放，可以先查看 **播放诊断** 和 **日志**，并核对该站源的账号、代理及实际线路状态。

## 五、更新与备份

### 更新镜像

使用 NAS 的 Compose 项目执行**拉取镜像并重新部署**；命令行可在项目目录运行：

```bash
docker compose pull
docker compose up -d
```

**升级不能随意删除卷或修改挂载名称；尤其不要运行 `docker compose down -v`。**

### 必须保留的数据

| 容器路径 | 内容 | 建议 |
| --- | --- | --- |
| `/data` | 设置、账号、缓存、索引 | **必须备份** |
| `/strm` | STRM / NFO / 图片及已下载的媒体 | 使用 STRM 时备份 |
| `/source` | 本地猫源文件（若启用） | 使用本地猫源时备份 |

本示例的 `/data` 保存于 Docker 命名卷 `moreplay-data`，`/strm` 保存于项目目录下的 `strm-library`。**命名卷和普通文件夹不是一回事**：迁移设备或更换 Compose 项目名时，建议先查清实际挂载位置并备份。

## 六、常见问题

**1. 浏览器打不开后台？** 先确认容器正在运行、NAS IP 与宿主机端口正确、设备在可访问的网络中，完整管理地址的结尾为 `/bridge`。如果端口被占用，修改映射左侧，如 `2233:2333`。

**2. 容器已运行，但一部影片都没有？** MorePlay 本身不带影视资源。检查站源是否添加、扫描成功并启用，然后检查目标模块有没有勾选对应站点。

**3. 管理台密码错误？** 检查 `CATPAW_SETTINGS_PASSWORD`。已有 `/data` 时可能保留之前的管理设置；Emby 播放账号与管理台密码相互独立。

**4. 猫源加载异常或完全无搜索结果？** 检查源 URL、源站状态、必要的登录信息以及代理配置。

**5. 有线路却不能播放、图片加载失败？** 先看 `/bridge/playback-diagnostics` 与 `/bridge/logs`，排查站点有效性、账号过期、视频地址有效期、播放模式和客户端兼容性。

**6. Emby 目录没有及时变化？** 客户端可能缓存了旧列表。在 Emby 管理页按首页、站点或分类范围刷新，再重新打开播放器。

**7. STRM 在 NAS 内能播放，在其他设备打不开？** 检查 `.strm` 中的对外地址是否能从真正执行播放 / 取流的设备访问；外网入口变化时可更换对外访问地址。

**8. 换了端口，原来的播放器地址还能用吗？** 不行。若把 `2333:2333` 改成 `2233:2333`，管理台应改为 `http://你的NAS地址:2233/bridge`，各播放器订阅地址中的端口也要同步修改。

## 七、版本更新说明

### 3.0

- 修复 STRM 部分分类未正确扫描。
- 修复 T4 会话过期后未自动重连。
- 修复模块聚合搜索刷新未覆盖旧结果。
- 修复 Emby 在 Rex 中站源名与线路名显示异常。
- 新增 Emby 使用 TMDB 刮削海报的能力。

### 2.9.4

- 修复 Emby 需要手动清理播放器缓存才会更新封面的逻辑。
- 修复 AfuseKt 播放器连接问题。

## 八、问题反馈与安全提醒

遇到问题可在 [MorePlay 论坛反馈主题](https://club.ugnas.com/forum.php?mod=viewthread&tid=6122) 中说明：**MorePlay 版本、NAS 系统、播放器、站源名称、操作步骤、报错截图或日志**。分享前务必删除密码、Cookie、Token 和私人分享链接等敏感信息。

- 仅添加可信作者提供的猫源；猫源可能包含可执行代码。
- 不要把管理端口直接暴露到不可信的公网；远程访问建议使用受控网络和 HTTPS。
- MorePlay 不提供影视内容，请自行遵守适用法律、版权规定和第三方服务条款。

**相关链接：** [GitHub 仓库](https://github.com/mobai12312/moreplay) · [Docker Hub 镜像与标签](https://hub.docker.com/r/mobai1231/catpawopen-fw/tags) · [问题反馈](https://club.ugnas.com/forum.php?mod=viewthread&tid=6122)
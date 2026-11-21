# EECS 570 · Dental & Kellogg 室内地图

线上入口：**https://map.solvaneon.com/**

源码仓库：https://github.com/Photo-king/ENTR550OnlineMap

纯静态 HTML / CSS / JavaScript 项目，无构建步骤、后端、数据库或地图 API Key。

## 当前功能

- 默认主页为 **Overview**，展示组员提供的原始总览 SVG；随后依次是 Ground、First、Second。
- **四个页面均支持双指缩放和放大后的单指拖动**，电脑支持滚轮缩放。G/1/2 另有缩放及全图按钮。
- 初始与最小缩放完整适配当前设备的图纸，缩到下限后恢复居中全图。切换楼层会恢复该层初始视图。
- 图例及注释为独立、可折叠面板，不随地图缩放。三个楼层统一使用 First 的符号/颜色图例，目的地说明各自保留。
- 打开网页自动请求位置权限，包括 Overview 主页；定位点只显示在 G/1/2。楼层由用户选择，GPS 不判断楼层。
- 楼外但仍在当前视口中的定位点也显示；失去新定位时保留灰色的最后已知位置。位置数据只保存在浏览器内存中，不上传服务器。
- 公开页面不显示经纬度、精度、时间或测试入口，也没有 Enable/Stop location 按钮。权限被拒绝后需通过浏览器设置恢复。
- 原始素材保存在 `Mapsv1.0/`。室外位图未启用；单层地图移除了原图标题、版式边框和标签色块。

## 本地开发

```sh
python3 -m http.server 5700
```

打开 http://localhost:5700/ 。手机定位需要 HTTPS；手机自己的 localhost 不是这台开发电脑。

| 链接参数 | 页面 |
| --- | --- |
| 无参数或 `?level=union` | Overview |
| `?level=ground` | Ground |
| `?level=first` | First |
| `?level=second` | Second |

**本地编辑不会自动同步线上。** 保存文件、Git 提交、推送 GitHub、部署服务器是不同步骤，完成情况必须分别核对。

## 代码与数据

| 文件 | 用途 |
| --- | --- |
| `index.html`、`style.css`、`app.js` | 页面、样式、定位与地图交互 |
| `gestures.mjs` | 各页面共用的双指缩放与拖动逻辑 |
| `map-data.js` | 楼层、原始五点 WGS84 坐标和各图对应点 |
| `registration.mjs`、`positioning.mjs` | 经纬度映射、楼层对齐、视图计算 |
| `scripts/prepare-maps.py` | 从原始 SVG 生成交互地图、注释和图例 |
| `map-footprints.js`、`public/maps/` | 生成文件，随源码一起提交 |
| `scripts/check-browser.cjs` | 手机多点触摸、定位和公开界面的浏览器验收 |
| `scripts/release.py`、`deploy/release_remote.py` | 版本校验、发布、回滚工具 |
| `deploy/school-map.conf` | 仅本地图站点的 nginx 配置 |

更新素材后运行并检查生成结果，再将源文件和生成文件一起提交：

```sh
python3 scripts/prepare-maps.py
node --check app.js
node --test tests/positioning.test.mjs tests/gestures.test.mjs
```

三层地图已更新为 2026-11-19 提供的 Team 2D corridor draft SVG，导入时统一使用原有文件名。生成过程保留源图墙线、房间文字和图标，去除标题、边框及标签底色，并消除 Second 的重复文字；一楼南北入口文字完整保留。白色室内填充与浅绿画布分开处理，足迹多边形不替代或裁切原始墙线。三个单层图不额外旋转或剪切，原定位校准保持不变。参见[当前校准说明](docs/calibration.md)。

Ground 于 2026-11-21 更新为 `Dental_Kellogg_Ground_2D_Drive.svg`，补充教室及房间名称。导入时将打印版式和内部缩放平移还原到原 Ground 坐标，保留已通过实地验证的定位校准。

开发调试功能保留在 `app.js`，由 `ENABLE_LOCATION_DEBUG = false` 关闭。需要时可在本地显式开启；发布前必须恢复 `false`。关闭时面板隐藏、输入禁用、事件处理也会阻止测试模式启用；没有 URL 开关。

## 版本和发布

**完整 Git commit SHA 是版本的唯一依据。** 发布不使用未提交工作区，也不直接编辑服务器上的上线文件。

```sh
# 提交并推送后，才可部署
python3 scripts/release.py deploy

# 核对：本地干净、HEAD = GitHub main = 线上版本，并校验所有静态文件 SHA-256
python3 scripts/release.py check
```

工具依赖本机 Git、Node、Python 3、已配置的 `ssh solvaneon`；服务器使用 Python 3、curl、nginx。仅管理 `/var/www/school-map/`，不自动改 nginx 或其他业务服务。发布使用 GitHub 上指定提交的源归档，校验后新建 release、生成 `version.json`，再原子切换 `current`；HTTP 检查失败会恢复旧链接。

线上版本记录：**https://map.solvaneon.com/version.json**，包含完整提交号、部署时间、源码链接和静态资源校验值。首次启用该工具前的旧版本没有此端点，但可以通过目录、Git 提交及文件哈希核对。

详细发布步骤、验证、版本标签和回滚见[发布与版本管理](docs/releases.md)；变更记录见 [CHANGELOG.md](CHANGELOG.md)。

## 已知限制

GPS 在室内会漂移、过期或不可用；所显示蓝点不是精确导航或楼层识别。五个参照点来自用户的 Google Maps 数据，仍需实地核对。Overview 是原始示意图，支持查看缩放，但不承载地理定位点。

`README-school-map.md` 及 `docs/history/` 为历史交接和探索记录，不能作为当前功能/发布说明；以本 README 和 `docs/releases.md` 为准。

# AGENTS.md · 高考倒计时壁纸（GitHub 发布版）

> 面向 AI / 协作者的仓库说明。当前 **v1.0.0**，基于主项目 `v4-se`。
> 维护者：**ouroooo** · 仓库：https://github.com/ouroooo/gaokao-countdown
> 先读 README 的「改成你的学校」；改坐标是用户最常见的操作。

## 1. 项目是什么

纯静态单页：高考倒计时 + 毛泽东诗词轮播 + 地球定位开场。
无构建、无 npm、无网络请求；`index.html` 可直接双击运行，也可放 GitHub Pages。

## 2. 文件

| 文件 | 作用 | 是否必需 |
|---|---|---|
| `index.html` | 全部应用代码、诗词、陆地数据 | 是 |
| `earth-still.webp` | 默认坐标的预渲染地球静帧（3072²，透明） | 否，性能优化 |
| `preview.png` | README 预览图 | 否 |
| `README.md` | 用户文档，含坐标修改说明 | — |
| `AGENTS.md` | 本文件 | — |
| `LICENSE` | MIT 开源协议（代码） | — |

## 3. 唯一配置源

`index.html` 里搜索 `用户配置`：

```js
var TARGET_LON = 116.694;   // 经度（东经正 / 西经负）
var TARGET_LAT = 24.352;    // 纬度（北纬正 / 南纬负）
var TARGET_NAME = '虎山中学';
var TARGET_SUB = '广东省梅州市大埔县';

var STILL_LON = 116.694;    // earth-still.webp 的渲染中心
var STILL_LAT = 24.352;
```

约束：

1. **坐标顺序是“经度, 纬度”**，不是 Google 的“纬度, 经度”。
2. `TARGET_NAME` / `TARGET_SUB` 同时驱动：
   - canvas 里的定位标签（`drawMarker`）；
   - 静帧状态下的 DOM 标签（`#stillName` / `#stillSub`）。
   不要在别处再写死地点文字。
3. `STILL_*` 只用于判断静帧是否还能用，**不要跟着 `TARGET_*` 一起改**，除非真的重新生成了静帧。

## 4. 坐标变化后的行为（关键）

```js
var CUSTOM_LOCATION = Math.abs(TARGET_LON - STILL_LON) > 0.01 ||
                      Math.abs(TARGET_LAT - STILL_LAT) > 0.01;
if (!CUSTOM_LOCATION) still.src = './earth-still.webp';
```

- **默认坐标**：加载 `earth-still.webp`；约 6.25s 后 `enterStill()` 隐藏 canvas、显示静帧 + DOM 脉冲点。
- **自定义坐标**：不加载静帧；`enterStill()` 检测到 `CUSTOM_LOCATION`，直接 `mode = 'frozen'` 并 `stopLoop()`。
  - 画面是当前实时 canvas 的最后一帧，坐标正确；
  - 空闲同样 0 次重绘；
  - 没有脉冲点和预渲染图的锐度。

**不要“修复”成自定义坐标仍加载旧静帧**——那会把用户带到旧学校。
也**不要删除这个回退**：`earth-still.webp` 被删除 / 加载失败时同样靠它兜底。

## 5. 模式

`mode` 取值：`intro | still | live | frozen`

- `intro`：0–5s 自转 + 缩放；5s 加 `.dim`。
- `still`：默认坐标，6.25s 后显示 `earth-still.webp`，RAF 停止。
- `live`：拖动时恢复实时 canvas（`beginDrag`），松手后惯性最多 550ms。
- `frozen`：速度归零后冻结，RAF 停止。

按钮 `#globeToggle` 在 `.content` 内、目标日期下方；隐藏时切换 `#globeLayer.hidden`，实时模式暂停/恢复 RAF。

## 6. 代码地图（按搜索词，不按行号）

| 搜索 | 内容 |
|---|---|
| `用户配置` | 坐标 / 标签 / STILL 配置 |
| `var LAND =` | Natural Earth 110m 陆地数据 |
| `function updateCountdown` | 倒计时、目标日期滚动 |
| `const poems = [` | 79 首毛泽东诗词 |
| `getSpecialText` | 9/9、12/26 纪念日 |
| `POEM_INTERVAL_MS` | 每句停留时间（7000ms） |
| `function resize` | 视口 / 地球中心 / 半径 |
| `function drawMarker` | 定位点、标签（用 TARGET_NAME/SUB） |
| `function beginDim` | 5s 变浅、按钮出现、6.25s 切静帧 |
| `function enterStill` | 静帧切换 + 自定义坐标回退 |
| `function frame` | 开场 / 拖动 / 冻结 |
| `function startLoop` / `stopLoop` | RAF 生命周期 |

## 7. 常见修改

### 改学校

1. 改 `TARGET_LON` / `TARGET_LAT`（十进制度，经度在前）。
2. 改 `TARGET_NAME` / `TARGET_SUB`（保持短）。
3. **不要改** `STILL_LON` / `STILL_LAT`。
4. 打开页面验证：约 5s 后目标点应位于地球中心，标签正确。

### 换静帧（可选）

只有想保留脉冲点 / 预渲染锐度时才做：

1. 用相同正交投影、目标朝向，在 3072×3072 透明画布上渲染地球（球半径 1300，含光晕留白，不含标记/文字）；
2. 导出 WebP，替换 `earth-still.webp`；
3. 把 `STILL_LON` / `STILL_LAT` 改成对应坐标。

### 改日期 / 诗词

- 日期：`targetYear = 2027` 与 `new Date(targetYear, 5, 7, 9, 0, 0)`，月份 0 基。
- 诗词：`poems` 数组，`{ t: '题', v: ['句1', '句2'] }`；同首内顺序不能乱。
- 纪念日：`getSpecialText`。
- 换诗动画只在诗与诗之间；同首诗逐句直接切换。

## 8. 测试清单（改完必做）

- [ ] 默认坐标：约 6.4s 后 canvas 隐藏、`earth-still.webp` 显示、DOM 脉冲点显示、标签正确。
- [ ] 默认坐标：静帧状态 1 秒内 canvas `clearRect` 计数为 0。
- [ ] 自定义坐标（例如北京 116.4074 / 39.9042）：约 6.4s 后 canvas 可见、静帧隐藏、无 `<img src>` 请求、标签为新地点。
- [ ] 自定义坐标：冻结后 1 秒内 `clearRect` 计数为 0。
- [ ] 拖动：切回实时 canvas，松手后冻结。
- [ ] 隐藏 / 显示：`#globeLayer` 透明度 1 → 0 → 1；按钮文案切换。
- [ ] 横屏 / 竖屏：按钮在目标日期下方，字号 1vw / 1.5vw。
- [ ] 控制台无报错；两个 `<script>` 可被 `vm.Script` 解析。
- [ ] 若改了 `index.html`，同步 README / AGENTS 中相关描述。

## 9. 已知问题（不处理）

- 南极洲多边形环绕极点，正交裁剪时可能生成楔形填充（转到南极附近可见）。默认视角不可见；按需求**保留现状**。
- 定位点是近似坐标，不表示校门精确位置。
- 开场 5s 仍是实时渲染；只有静止后零重绘。

## 10. 发布

- GitHub Pages：Settings → Pages → Deploy from branch → `main` / root；入口是 `index.html`。
- 保持零依赖：不要引入 CDN、外部字体、接口请求。
- 保持“HTML + 可选静帧”的结构；`earth-still.webp` 缺失时页面应能回退。
- 修改坐标逻辑后，务必重新验证「默认坐标」和「自定义坐标」两条路径。

## 11. 许可证

- 代码：**MIT**，见 `LICENSE`，Copyright (c) 2026 **ouroooo**。
- 诗词文本不在 MIT 范围内，版权归原作者 / 权利人。
- Natural Earth 110m land：公有领域。
- 仓库：https://github.com/ouroooo/gaokao-countdown
- Pages：https://ouroooo.github.io/gaokao-countdown/

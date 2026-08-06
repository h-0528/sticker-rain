# Sticker Rain

一个使用 Three.js 实现的透明贴纸下落效果，主要用于一生一芯学习追踪器。贴纸会随机分散下落，并根据登录和学习计时状态自动显示或隐藏。

## 在线预览

- 完整预览：https://h-0528.github.io/sticker-rain/
- 透明贴纸层：https://h-0528.github.io/sticker-rain/overlay.html

`index.html` 用于独立预览完整效果，`overlay.html` 是嵌入原网站的透明贴纸层。

## 功能

- 随机位置、速度和角度下落
- 尽量减少贴纸之间的重叠
- 校徽、ICIL 和一生一芯贴纸使用更大的显示尺寸
- GitHub、Visual Studio Code 和 Linux 编程主题贴纸
- 半透明画布，不拦截鼠标点击和页面操作
- 根据登录和计时状态开始或停止下落
- 页面滚动时贴纸层固定在浏览器视口中

## 项目结构

```text
sticker-rain/
|-- index.html       # 独立预览页面
|-- overlay.html     # 可嵌入其他网站的透明贴纸层
|-- img/             # 贴纸图片
`-- README.md        # 项目说明
```

## 本地运行

项目使用 ES Modules 加载 Three.js，因此建议通过本地 HTTP 服务器打开，不要直接双击 HTML 文件。

在项目目录运行：

```powershell
python -m http.server 8765
```

然后访问：

```text
http://127.0.0.1:8765/
http://127.0.0.1:8765/overlay.html
```

## 嵌入网站

在目标页面的 `</body>` 前加入透明 iframe：

```html
<iframe
  id="stickerOverlay"
  src="https://h-0528.github.io/sticker-rain/overlay.html"
  title="Sticker overlay"
  aria-hidden="true"
  tabindex="-1"
  style="
    position: fixed;
    inset: 0;
    width: 100%;
    height: 100%;
    border: 0;
    background: transparent;
    pointer-events: none;
    z-index: 9999;
  "
></iframe>
```

`pointer-events: none` 可以保证贴纸层不会挡住按钮、输入框或其他页面操作。

## 状态通信

目标网站使用 `postMessage` 向贴纸 iframe 发送状态：

```js
const frame = document.getElementById("stickerOverlay");

frame.contentWindow.postMessage(
  { type: "sticker-timer-start" },
  "https://h-0528.github.io"
);
```

支持的消息如下：

| 消息 | 效果 |
| --- | --- |
| `sticker-login-success` | 登录成功，隐藏贴纸 |
| `sticker-timer-start` | 开始计时，显示并继续下落 |
| `sticker-timer-stop` | 结束计时，停止并隐藏贴纸 |
| `sticker-logout` | 退出登录，重新显示贴纸 |

`overlay.html` 会检查消息来源，目前只接受来自 `https://ysyx.200502.xyz` 的消息。

## 自定义贴纸

把透明背景的 PNG 图片放进 `img/`，然后在 `STICKER_URLS` 中加入路径：

```js
const STICKER_URLS = [
  "img/example.png",
];
```

常用参数位于 `CONFIG` 中：

| 参数 | 作用 |
| --- | --- |
| `poolSize` | 同屏贴纸数量 |
| `spawnWidth` | 横向生成范围 |
| `fallSpeed` | 下落速度 |
| `rotationSpeed` | 旋转速度 |
| `windStrength` | 左右飘动幅度 |
| `scale` | 普通贴纸尺寸 |
| `codingScale` | 编程主题贴纸尺寸 |
| `featuredScale` | 校徽、ICIL 和一生一芯贴纸尺寸 |
| `featuredChance` | 特色贴纸出现概率 |

画布透明度可以修改 `#stage` 的 `opacity`。

## 技术说明

- 使用 Three.js `0.160.0`
- 使用 InstancedMesh 复用贴纸几何体，减少大量贴纸同时下落时的渲染开销
- 使用 Canvas 合成纹理图集，再通过自定义 Shader 选择不同贴纸

## 标识说明

项目中的学校、组织和软件标识仅用于学习与页面效果展示，相关名称、商标和图形的权利归各自权利人所有。

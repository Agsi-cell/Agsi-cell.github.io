# 夜读书桌 · Three.js 窗边夜景交互场景

在线体验：**https://agsi-cell.github.io/**

> github.io 在中国大陆经常严重限速（实测 1.3 MB 页面 120 秒只能传完 212 KB）。
> 国内直连镜像，内容与上面逐字节一致：https://night-desk-ymo54c7rps3.qoder.zone/

一个用 Three.js（WebGL）从零建模的夜读书桌场景：窗边的木质书桌、黄铜台灯与百褶灯罩、
立着的一排旧书、皮质笔记本与钢笔、白瓷花瓶里的枯枝与小白花、几盆绿植，窗外是城市夜景的散景灯火与星空。

整个场景（含 three.js r160 与 OrbitControls）内联在**同一个 `index.html`** 里：
没有外部 CDN 依赖、没有贴图素材、没有构建步骤，离线双击也能打开。

## 交互

| 操作 | 效果 |
| --- | --- |
| 风（持续） | 两片窗帘一直被风吹动，褶皱实时起伏，松手后布料有惯性回落 |
| 拖动窗帘 | 手动开合，窗帘像手风琴一样朝两侧窗帘杆铰链收拢；合拢后会挡住月光 |
| 下拉灯链 / 点灯罩 | 关灯，暖色台灯淡出，冷色月光从窗外进入，在桌面投出窗形光束 |
| 空白处拖动 / 滚轮 | OrbitControls 旋转视角、缩放 |
| 左下面板 | 风力、窗帘开合度滑杆与两个开关按钮 |

## 技术要点

- **窗帘**：`MeshStandardMaterial.onBeforeCompile` 注入 GLSL，顶点着色器里叠加
  褶皱（主谐波 + 二次谐波 + 幅值调制）与三个频率的风动，`hang = (1-v)^1.4` 让下摆摆幅更大；
  法线由参数曲面 `curtainPoint(u,v)` 的有限差分解析求出，避免变形后光照错误；
  开合度通过 `uGather` uniform 驱动，几何铰链放在局部 `x = 0`，褶皱数量按 uv 计算，
  所以收拢时褶皱会自动变密。左右两片共用一份材质与 uniform，右片用对象级 `scale.x = -1` 镜像
  （把镜像烘进几何顶点会让 three.js 的正面绕序判断失效，整片被背面剔除）。
- **褶皱自阴影**：片元里用 `varying float vFoldShade` 调制 `diffuseColor`，让褶谷变暗、褶脊变亮。
- **光**：台灯是暖色 `PointLight` + `SpotLight`（挂在灯组上，随灯移动）；关灯后切到
  冷色 `DirectionalLight` 月光，ACES 色调映射的曝光随灯的开关混合变化。
- **月光被窗帘挡住**：窗帘本身 `castShadow = false`，改用只注册在 layer 1 的
  `MeshBasicMaterial({ colorWrite: false, depthWrite: false })` 遮光代理平面参与月光的阴影相机，
  代理的 `scale.x` 跟随开合度 —— 因此"合上窗帘"在物理上真的会遮住穿窗的光束。
- **纹理**：墙面木纹、布艺纹理、城市天际线与灯火、光斑贴图全部用 Canvas 2D 运行时程序化生成。
- **粒子**：城市散景、星空、空气尘埃是自定义 `ShaderMaterial` 的 `Points`，逐点相位闪烁。

## 文件

```
index.html      单文件站点（场景 + three.js + OrbitControls，全部内联）
og-cover.png    社交分享卡片配图（1500×787）
robots.txt      允许抓取并指向 sitemap
sitemap.xml     站点地图
.nojekyll       关闭 Jekyll 处理，文件原样发布
```

## 本地运行

直接双击 `index.html`，或起一个静态服务器：

```bash
python -m http.server 8080
```

需要支持 WebGL2 的浏览器（Chrome / Edge / Firefox / Safari 16+）。

## 许可

MIT。three.js 遵循其自身的 MIT 许可。

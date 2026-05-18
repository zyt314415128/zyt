# VR看房 项目规范

## 运行方式
```
python -m http.server 8080
```
浏览器打开 `http://localhost:8080`（必须 HTTP，file:// 有 CORS 限制）。

## 部署
- GitHub Pages: **https://zyt314415128.github.io/zyt/**
- 仓库: `https://github.com/zyt314415128/zyt`
- `git push` 后自动部署，约 1 分钟生效

## 技术架构
- 单文件 `index.html`，Three.js r160 CDN
- SphereGeometry(500, 256, 128) + BackSide 渲染 equirectangular 全景图
- `texture.repeat.x = -1` + `wrapS = RepeatWrapping` 修镜像
- 纹理：`generateMipmaps = true`，`anisotropy = renderer.capabilities.getMaxAnisotropy()`
- FOV 范围：10°～100°（滚轮/双指缩放）
- rig (THREE.Group) 包裹 sphere + sprites，rig.rotation.y 控制水平视角
- 路标为 Sprite，挂在 rig 上，radius=480，通过 bearing+elevation 定位

## 输入系统
- 鼠标/触屏拖拽旋转视角（`userYaw -= dx * DRAG_SPEED`，`userPitch += dy * DRAG_SPEED`）
- 滚轮缩放 FOV（10°～100°）
- 触摸双指缩放（10°～100°）
- 陀螺仪 VR（移动端）— 逐帧增量模式，0.3° 死区过滤噪声
  - yaw: `userYaw -= deltaAlpha`
  - pitch: `userPitch += deltaBeta`

## 房间数据（ROOMS）
8 个房间的 x/y/yaw/pitch 和 connections 配在 `index.html` 的 ROOMS 对象中。
全景图放在 `home/` 目录下。两个客厅已重命名为「客厅1」「客厅2」。

## 路标校准
- 校准数据已嵌入代码 `DEFAULT_CALIBRATION` 常量（index.html 第 140 行）
- localStorage key: `vr_hotspot_angles_v2`（优先级高于默认值，用于覆盖）
- 格式: `{ roomId: { targetId: { bearing, elevation } } }`
- bearing: 水平角度(deg)，elevation: 垂直角度(deg)
- **路标位置已全部手动校准完毕，替换图片时无需重新调整**

## 编辑模式操作
| 操作 | 说明 |
|------|------|
| E | 进入/退出编辑模式 |
| 鼠标拖拽路标 | 在画面中自由移动路标 |
| ← → | 微调 bearing ±2° |
| ↑ ↓ | 微调 elevation ±2° |
| 空格 | 将选中路标吸附到视野中央 |
| 数字键 1-9 | 切换选中路标 |
| S | 保存校准到 localStorage |

## 替换全景图的步骤（叫「小鹿」按此流程执行）

1. 检查 `home/` 目录下的新图片文件名，确认是否与 ROOMS 中 `file` 路径一致（无 `_000` 后缀等差异）
2. 更新 `index.html` 中 ROOMS 的 `file` 路径，确保指向新文件名
3. 确认渲染配置正确：
   - `SphereGeometry(500, 256, 128)` — 分段数
   - 纹理加载时设置 `generateMipmaps = true`、`anisotropy = max`
   - FOV 范围 `Math.max(10, Math.min(100, ...))`
4. 如果房间的相机坐标/角度变了，更新 ROOMS 中的 x/y/yaw/pitch
5. `DEFAULT_CALIBRATION` 中的路标位置与房间 ID 绑定，图片更换后无需重新校准
6. 启动 HTTP 服务器验证页面正常加载
7. 提交并推送至 GitHub

## 图片文件命名约定
- 旧文件格式：`home/{房间名}_000.jpg`
- 新文件格式：`home/{房间名}.jpg`（无 `_000` 后缀）
- 文件名中的房间名必须与 ROOMS 中定义的 name 一致

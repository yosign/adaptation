# HomeA 结果页适配策略完整说明

## 核心目标
在不同尺寸的 iPhone 设备上展示 AI 生成的图片，确保最佳的视觉呈现效果。

## 展示区域定义

### 设备屏幕分层结构（从上到下）：
1. **状态栏 / safeArea**（顶部）：显示时间、信号等系统信息
2. **66pt 顶部操作栏**：放置关闭按钮等操作控件
3. **常规展示窗**：理想的图片展示区域（不覆盖顶部操作栏）
4. **图片展示区**：实际图片渲染的位置（根据策略动态调整）
5. **底部 UI 组件面板**：固定高度，包含控制按钮等
6. **底部安全区**：现代 iPhone 的 Home Indicator 区域

### 关键区域计算：
```javascript
panelTop = phoneH - bottomPanelH              // 底部面板起始位置
normalTop = safeArea + 66                     // 常规展示窗起始位置
normalContentTop = normalTop - 10             // 向上借 10pt，避免视觉上压住关闭按钮
normalContentH = panelTop - normalContentTop  // 常规展示窗可用高度
expandedH = panelTop                          // 扩展展示框高度（从屏幕顶到底部面板）
```

## 适配策略分支

### **策略 1：3:4 特例（864×1184）**
- **触发条件**：图片尺寸精确为 864×1184
- **展示方式**：填充模式（Fill）
- **行为**：
  - 图片宽度固定为设备宽度
  - 图片高度填充整个扩展展示框（从屏幕顶到底部面板）
  - 使用 `object-fit: cover` 效果，上下多余部分被裁切
- **理由**：3:4 是常见的竖图比例，优先保证在展示区域内尽量铺满，视觉冲击力强

### **策略 2：常规居中模式（Normal Center）**
- **触发条件**：`fitH ≤ normalContentH`
  - `fitH = 设备宽度 × (图片高度 / 图片宽度)`
  - 即图片按设备宽度缩放后，高度能放入常规展示窗
- **展示方式**：
  - 图片宽度 = 设备宽度
  - 图片高度 = 等比缩放后的高度
  - 在常规展示窗内上下居中
  - 为避免视觉上压住关闭按钮，展示窗顶边上移 10pt
- **适用图片**：1:1（方图）、4:3（横图）、16:9（宽屏横图）等矮图

### **策略 3：填充展示模式（Fill）**
- **触发条件**：`fitH > normalContentH`
  - 图片按设备宽度缩放后，高度超过常规展示窗
- **展示方式**：
  - 图片宽度 = 设备宽度
  - 图片高度 = 扩展展示框高度（panelTop）
  - 位置从屏幕顶部（y=0）开始
  - 使用 `object-fit: cover` 效果，上下多余部分被裁切
- **适用图片**：9:16（手机竖图）、2:3（竖图）等高图

## 算法伪代码

```javascript
function computeLayout(phone, imgW, imgH, strategy) {
  // 计算基础参数
  const ratioHW = imgH / imgW
  const normalTop = phone.safeArea + strategy.closeBar + strategy.gap
  const panelTop = phone.height - phone.bottomPanelH
  const normalH = Math.max(0, panelTop - normalTop)
  const normalHeadInset = strategy.normalHeadInset || 0
  const normalContentTop = normalTop - normalHeadInset
  const normalContentH = Math.max(0, normalH + normalHeadInset)
  const expandedTop = 0
  const expandedH = panelTop - expandedTop
  const fitH = phone.width * ratioHW

  // 策略 1：3:4 特例
  if (imgW === 864 && imgH === 1184) {
    return {
      mode: "fill-3-4",
      imageW: phone.width,
      imageH: expandedH,
      imageX: 0,
      imageY: expandedTop
    }
  }

  // 策略 2：常规居中
  if (fitH <= normalContentH) {
    return {
      mode: "normal",
      imageW: phone.width,
      imageH: fitH,
      imageX: 0,
      imageY: normalContentTop + (normalContentH - fitH) / 2
    }
  }

  // 策略 3：填充展示
  return {
    mode: "fill",
    imageW: phone.width,
    imageH: expandedH,
    imageX: 0,
    imageY: expandedTop
  }
}
```

## 各尺寸应用情况（参考）

### 以 iPhone 14 Pro (393×852) 为例：

| 图片尺寸 | 比例 | 触发策略 | 展示效果 |
|---------|------|---------|---------|
| 864×1184 | 3:4 | 策略 1 (3:4特例) | 填充展示，上下裁切 |
| 1024×1024 | 1:1 | 策略 2 (常规居中) | 完整展示，上下留白 |
| 1184×864 | 4:3 | 策略 2 (常规居中) | 完整展示，上下留白 |
| 864×1536 | 9:16 | 策略 3 (填充展示) | 填充展示，上下裁切 |
| 1536×864 | 16:9 | 策略 2 (常规居中) | 完整展示，上下留白 |
| 1024×1536 | 2:3 | 策略 3 (填充展示) | 填充展示，上下裁切 |

### 不同设备的适配表现：

| 设备型号 | 屏幕尺寸 | safeArea | 常规展示窗高度 | 扩展展示框高度 |
|---------|---------|----------|--------------|--------------|
| iPhone 16/17 Pro Max | 440×956 | 62pt | 525pt | 587pt |
| iPhone 17/17 Pro | 402×874 | 62pt | 485pt | 547pt |
| iPhone 14 Pro/15/16 | 393×852 | 59pt | 466pt | 525pt |
| iPhone 13 Pro Max | 428×926 | 47pt | 510pt | 557pt |
| iPhone 12/13/14 | 390×844 | 47pt | 470pt | 517pt |
| iPhone 12 mini/13 mini | 375×812 | 50pt | 450pt | 500pt |
| iPhone X/XS/11 Pro | 375×812 | 44pt | 456pt | 500pt |
| iPhone 6/7/8 | 375×667 | 20pt | 369pt | 389pt |

## 设计理念

1. **优先完整展示**：能放入常规展示窗的图片，完整居中展示，保证用户看到完整内容
2. **次选填充裁切**：超出常规展示窗的高图，改为填充模式，牺牲上下边缘换取更好的视觉呈现
3. **特例优化**：3:4 作为常见竖图比例，固定走填充分支，保证一致性和最佳视觉效果
4. **宽度固定**：所有模式下图片宽度始终等于设备宽度，充分利用横向空间
5. **避免缩窄**：取消了"缩宽保全图"策略，不再为了显示完整图片而留左右黑边
6. **视觉优先**：在完整性和视觉冲击力之间，优先选择视觉冲击力

## 技术实现细节

### CSS 关键样式：
```css
.image-content {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;  /* 填充模式时裁切上下 */
  z-index: 1;
  transition: filter 0.16s ease;
  pointer-events: none;
}
```

### 图片尺寸规范：
- 所有图片尺寸按 16 对齐（864, 1024, 1184, 1536 等）
- 支持 6 种预设比例：
  - 3:4（864×1184）
  - 1:1（1024×1024）
  - 4:3（1184×864）
  - 9:16（864×1536）
  - 16:9（1536×864）
  - 2:3（1024×1536）

### 支持设备范围：
- 现代全面屏 iPhone（iPhone X ~ iPhone 17 系列）
- 传统 iPhone（iPhone 6 ~ iPhone 8 系列）
- 共 11 种主流屏幕尺寸

### 响应式计算：
```javascript
// 预览比例计算
const scale = CARD_VIEW_WIDTH / phone.width

// 渲染尺寸
shellW = phone.width * scale
shellH = phone.height * scale

// 坐标转换
styleRect = (x, y, w, h) => 
  `left:${x * scale}px;top:${y * scale}px;width:${w * scale}px;height:${h * scale}px;`
```

## 优势总结

✅ **简化逻辑**：从 4 分支（常规/借顶部/缩宽/特例）简化为 2 分支（常规/填充）  
✅ **视觉统一**：所有图片等宽展示，无左右留白  
✅ **高图友好**：超高图不再被压缩，改为裁切保持视觉冲击力  
✅ **响应式**：自动适配不同设备的屏幕尺寸和安全区  
✅ **性能优化**：使用原生 CSS `object-fit: cover`，GPU 加速  
✅ **易于维护**：逻辑清晰，分支简单，易于理解和调试  
✅ **可视化预览**：提供交互式预览工具，支持实时切换不同尺寸和设备  

## 使用方式

1. 打开 `index.html` 预览页面
2. 在顶部控制面板选择图片尺寸
3. 查看不同设备的适配效果
4. 鼠标悬停在各个区域可查看组件名称和高度
5. 底部关系图显示图片裁切情况

## 未来优化方向

- [ ] 支持更多图片比例（如 1:2、5:4 等）
- [ ] 添加横屏模式适配
- [ ] 支持动态调整底部面板高度
- [ ] 添加图片裁切位置微调（顶部对齐/底部对齐/居中）
- [ ] 支持自定义安全区大小
- [ ] 添加暗黑模式/浅色模式切换

## 更新日志

### v2.0 (2026-02-11)
- 🔧 简化适配策略：移除"借顶部"和"缩宽保全"分支
- ✨ 新增填充展示模式：超出常规展示框改为裁切展示
- 📝 更新文档和算法说明
- 🎨 优化视觉呈现效果

### v1.0 (初始版本)
- ✨ 实现 4 分支适配策略
- 🎨 支持 11 种 iPhone 设备
- 📊 添加可视化预览工具

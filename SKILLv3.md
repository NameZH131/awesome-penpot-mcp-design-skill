---
name: penpot-design
description: |
  使用 Penpot MCP 进行 UI/UX 设计。适用于通过 Penpot MCP Plugin 创建、修改、导出 UI 设计的场景。
  
  触发条件：
  - 用户请求创建 UI 设计（网页、移动端、Dashboard、表单、卡片等）
  - 用户要求使用 Penpot 进行设计
  - 用户需要生成界面设计或 UI 组件
  - 用户需要设计系统、布局或视觉界面
  
  核心能力：
  - 创建 Board、Text、Rectangle、Ellipse 等元素
  - 使用 Flex Layout 和 Grid Layout 进行布局
  - 应用 Design Tokens 和颜色样式
  - 上传图片和创建 SVG 图标
  - 添加交互和原型链接
---

# Penpot MCP 设计技能

通过 Penpot MCP 服务器在 Penpot 设计工具中创建 UI 设计。

**前置条件**：用户必须通过 Penpot MCP Plugin 连接 Penpot 设计项目到 MCP 服务器。

---

## 黄金法则（记住这 5 条）

| 法则 | 说明 | 违反后果 |
|------|------|---------|
| **单代码块** | 所有代码在一个 `execute_code` 中完成 | 变量丢失，需要 storage 传递 |
| **禁用裁剪** | 每个 Board 创建后立即 `clipContent = false` | 子元素被裁剪，显示不全 |
| **Flex 优先** | 能用 Flex 就不用手动坐标 | 手动计算 y 坐标容易出错 |
| **实时验证** | 每添加关键元素后 console.log 检查 | 出错时难以定位问题 |
| **语义命名** | Board 用 `xxxSection`，列表用 `xxxList` | 变量重复声明错误 |

---

## 最小可运行模板

```javascript
// === 50行核心模板 ===

// 1. 清理环境
[...penpot.root.children].forEach(x => x.remove());

// 2. 创建页面容器
const page = penpot.createBoard();
page.name = 'Page';
page.resize(1440, 900);
page.clipContent = false;  // ⚠️ 必须禁用裁剪
penpot.root.appendChild(page);

// 3. 添加 Flex 布局
const flex = page.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 40;

// 4. 添加 Header
const header = penpot.createBoard();
header.name = 'HeaderSection';
header.resize(1320, 70);
header.fills = [{ fillColor: '#0A0A0F', fillOpacity: 1 }];
page.appendChild(header);  // Flex 自动定位，无需设置 y

// 5. 添加 Hero
const hero = penpot.createBoard();
hero.name = 'HeroSection';
hero.resize(1320, 400);
hero.clipContent = false;
page.appendChild(hero);

const title = penpot.createText('HELLO WORLD');
title.fontSize = 48;
title.fontWeight = '800';
title.fills = [{ fillColor: '#FFFFFF' }];
hero.appendChild(title);

// 6. 验证
console.log('✅ 完成:', page.children.length, '层');
console.log('层级:', page.children.map(c => c.name).join(' → '));
```

---

## 核心概念

### 坐标系统

| 属性 | 类型 | 说明 |
|------|------|------|
| `setParentXY(shape, x, y)` | 设置相对位置 | `setParentXY(child, 100, 50)` |
| `x/y` | 绝对坐标 | 可写，相对于画布 |
| `parentX/parentY` | 相对坐标 | 只读，相对于父元素 |
| `width/height` | 只读 | 用 `resize(w, h)` 修改 |

### clipContent 裁剪

```javascript
// Board 默认 clipContent = true，会裁剪超出边界的内容
// ⚠️ 创建任何 Board 后立即禁用
const board = penpot.createBoard();
board.clipContent = false;  // 第一件事！
```

### fills 设置

```javascript
// ✅ 正确格式：必须包含 fillOpacity
rect.fills = [{ fillColor: '#FF0000', fillOpacity: 1 }];

// ❌ 常见错误：缺少 fillOpacity
rect.fills = [{ fillColor: '#FF0000' }];  // 图片会显示空白
```

### Z-Order 层级顺序

- `appendChild()` → 添加到末尾 = 视觉顶层
- `insertChild(index, shape)` → 控制指定位置
- 子元素按添加顺序从下到上堆叠

---

## 布局方案速查

### 三选一决策表

| 方案 | 适用场景 | 代码量 | 维护成本 | 推荐度 |
|------|---------|--------|---------|--------|
| **Flex Layout** | 多子元素自动排列 | 少 | 低 | ⭐⭐⭐ |
| **currentY** | 简单垂直堆叠 | 中 | 中 | ⭐⭐ |
| **手动坐标** | 特殊定位需求 | 多 | 高 | ⭐ |

### 方案一：Flex Layout（推荐）

```javascript
const page = penpot.createBoard();
page.resize(1440, 900);
page.clipContent = false;
penpot.root.appendChild(page);

const flex = page.addFlexLayout();
flex.dir = 'column';           // 垂直排列
flex.horizontalPadding = 60;   // 左右边距
flex.verticalPadding = 40;     // 上下边距
flex.rowGap = 20;              // 行间距

// 子元素自动排列，无需设置 y 坐标
[header, hero, gallery, footer].forEach(el => page.appendChild(el));
```

**Flex 常用属性速查**：

| 属性 | 说明 | 常用值 |
|------|------|--------|
| `dir` | 排列方向 | `'column'` / `'row'` |
| `justifyContent` | 主轴对齐 | `'center'` / `'space-between'` |
| `alignItems` | 交叉轴对齐 | `'center'` / `'stretch'` |
| `rowGap` / `columnGap` | 间距 | `20` |
| `horizontalPadding` | 水平内边距 | `60` |

### 方案二：currentY 累积坐标

```javascript
let currentY = 0;  // 从 0 开始

const nav = penpot.createBoard();
nav.resize(1440, 80);
nav.y = currentY;
page.appendChild(nav);
currentY += 80;  // 累积高度

const hero = penpot.createBoard();
hero.resize(1440, 500);
hero.y = currentY;  // 自动为 80
page.appendChild(hero);
currentY += 500;

console.log(`总高度: ${currentY}px`);  // 580
```

### 方案三：手动坐标（不推荐）

```javascript
// ❌ 容易算错，维护困难
nav.y = 0;
hero.y = 80;      // 需要精确计算
gallery.y = 580;  // 80 + 500，容易算错！
footer.y = 980;   // 580 + 400，继续算...
```

---

## penpotUtils 工具速查

| 工具 | 用途 | 示例 |
|------|------|------|
| `setParentXY(shape, x, y)` | 设置相对位置 | `setParentXY(child, 100, 50)` |
| `addFlexLayout(board, dir)` | 添加 Flex（保持顺序）| `addFlexLayout(board, 'column')` |
| `findShape(predicate)` | 查找单个元素 | `findShape(s => s.name === 'Nav')` |
| `findShapes(predicate)` | 查找多个元素 | `findShapes(s => s.name?.includes('Card'))` |
| `shapeStructure(shape, depth)` | 查看结构树 | `shapeStructure(root, 3)` |

### setParentXY 示例

```javascript
// ❌ parentX/parentY 是只读属性
shape.parentX = 100;  // TypeError!

// ✅ 使用工具函数
penpotUtils.setParentXY(shape, 100, 50);
```

### findShape / findShapes 示例

```javascript
// 精确查找
const nav = penpotUtils.findShape(s => s.name === 'Navigation');

// 模糊查找
const allCards = penpotUtils.findShapes(s => s.name?.includes('Card'));

// 按类型查找
const allTexts = penpotUtils.findShapes(s => s.type === 'text');
```

---

## 验证检查

### 实时验证（推荐）

```javascript
// 每添加关键元素后，立即 console.log 检查
const header = penpot.createBoard();
header.resize(1320, 70);
page.appendChild(header);
console.log('✅ Header 添加完成, 当前层数:', page.children.length);

const hero = penpot.createBoard();
hero.resize(1320, 400);
page.appendChild(hero);
console.log('✅ Hero 添加完成, 当前层数:', page.children.length);
```

### 结构检查

```javascript
// 1. 查看层级结构树
const structure = penpotUtils.shapeStructure(penpot.root, 4);
console.log(JSON.stringify(structure, null, 2));

// 2. 检查 Root 元素数量（应该只有 1 个页面容器）
console.log('Root children:', penpot.root.children.length);

// 3. 检查 clipContent 状态（全部应为 false）
page.children.forEach((layer, i) => {
  console.log(`${i}: ${layer.name} (y=${layer.y}, h=${layer.height}, clip=${layer.clipContent})`);
});

// 4. 检查子元素是否超出父容器
const gallery = penpotUtils.findShape(s => s.name === 'GallerySection');
gallery.children.forEach(child => {
  const overflow = child.y + child.height > gallery.height;
  if (overflow) console.log(`⚠️ ${child.name} 超出边界!`);
});
```

### 导出验证

```javascript
// 导出为 PNG 查看效果
// ⚠️ 大尺寸页面可能失败，优先用上面结构检查
export_shape({ shapeId: 'page', format: 'png' });
```

---

## 常见错误速查

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `Identifier 'xxx' has already been declared` | 变量命名冲突 | 使用语义化命名：`xxxSection`, `xxxList` |
| 变量在不同代码块丢失 | 跨调用上下文独立 | 使用单代码块策略或 storage |
| 导出失败 `http error` | 页面尺寸/元素过多 | 使用验证脚本替代导出 |
| 子元素被遮挡/裁剪 | 父Board的 clipContent=true | 创建Board后立即 `clipContent = false` |
| 子元素位置超出父容器 | 坐标计算错误 | 使用 Flex Layout 或 currentY |
| `Cannot add property fill` | Board 创建后立即设置 fills | 使用 Rectangle 替代 |
| Root 有多个元素 | 元素未正确添加到父容器 | 使用 verify() 检查层级 |
| 颜色不生效 | 小写颜色值 | 使用大写 `#FFFFFF` |
| 图片空白 | 缺少 fillOpacity | 添加 `fillOpacity: 1` |

---

## 精简示例：Landing Page

```javascript
// === Landing Page 完整示例（60行）===

// 1. 清理
[...penpot.root.children].forEach(x => x.remove());

// 2. 创建页面
const page = penpot.createBoard();
page.name = 'LandingPage';
page.resize(1440, 900);
page.clipContent = false;
penpot.root.appendChild(page);

const flex = page.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 30;

// 3. Header
const header = penpot.createBoard();
header.name = 'HeaderSection';
header.resize(1320, 70);
header.fills = [{ fillColor: 'rgba(10,10,15,0.95)', fillOpacity: 1 }];
page.appendChild(header);

const headerFlex = header.addFlexLayout();
headerFlex.dir = 'row';
headerFlex.justifyContent = 'space-between';

const logo = penpot.createText('BRAND');
logo.fontSize = 22;
logo.fontWeight = '800';
logo.fills = [{ fillColor: '#FFFFFF' }];
header.appendChild(logo);

['HOME', 'ABOUT', 'CONTACT'].forEach(item => {
  const nav = penpot.createText(item);
  nav.fontSize = 14;
  nav.fills = [{ fillColor: 'rgba(255,255,255,0.7)' }];
  header.appendChild(nav);
});

// 4. Hero
const hero = penpot.createBoard();
hero.name = 'HeroSection';
hero.resize(1320, 400);
hero.clipContent = false;
page.appendChild(hero);

const title = penpot.createText('Welcome to\nOur Platform');
title.fontSize = 56;
title.fontWeight = '800';
title.fills = [{ fillColor: '#FFFFFF' }];
hero.appendChild(title);

// 5. CTA Button
const cta = penpot.createRectangle();
cta.resize(140, 44);
cta.borderRadius = 22;
cta.fills = [{ fillColor: '#6366F1', fillOpacity: 1 }];
cta.y = 200;
hero.appendChild(cta);

// 6. 验证
console.log('✅ Landing Page 完成');
console.log('层级:', page.children.map(c => c.name).join(' → '));
console.log('Header 子元素:', header.children.length);
```

---

## API 查看

遇到不确定的 API 时：

```javascript
// 查看 Board 完整 API
penpot_penpot_api_info({ type: 'Board' })

// 查看 FlexLayout 属性
penpot_penpot_api_info({ type: 'FlexLayout' })

// 查看 GridLayout 属性
penpot_penpot_api_info({ type: 'GridLayout' })
```

---

## 详细文档

完整 API 参考见 Penpot MCP 文档。

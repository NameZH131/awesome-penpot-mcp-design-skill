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

## 核心工作流

```
需求确认 → 环境准备 → 创建层级 → 添加内容 → 验证检查
```

---

## Step 1: 需求确认

与用户确认：风格、配色、页面范围。

---

## Step 2: 环境准备

### 2.1 清理环境

```javascript
// 清理旧元素
[...penpot.root.children].forEach(x => x.remove());

// 如需创建新页面
const page = penpot.createPage();
page.name = '页面名称';
penpot.openPage(page);
```

### 2.2 单代码块策略（推荐！）

**核心原则**：在一个 `execute_code` 中完成所有设计，避免跨调用变量丢失。

```javascript
// ✅ 推荐：单代码块完成所有工作
// - 所有变量在同一上下文
// - 避免 90% 的跨调用问题
// - 代码更易理解和调试

// ❌ 避免：分多个代码块调用
// - 变量在不同上下文会丢失
// - 需要用 storage 传递数据
// - 容易产生孤儿元素
```

---

## Step 3: 创建层级结构（核心！）

### 变量命名规范（避免冲突）

**问题**：通用名称容易导致变量重复声明错误

```javascript
// ❌ 错误：通用名称容易冲突
const artists = ['Elena', 'Marcus'];
const artists = penpot.createBoard(); // ReferenceError!

// ✅ 正确：语义化名称
const artistNamesList = ['Elena', 'Marcus'];
const artistsSection = penpot.createBoard();
```

**命名建议**：

| 类型 | 命名规范 | 示例 |
|------|---------|------|
| 区域 Board | `xxxSection` | `artistsSection`, `gallerySection` |
| 数据列表 | `xxxList` 或 `xxxNames` | `artistNamesList`, `cardTitles` |
| 配色对象 | 单字母 `c` | `const c = { bgPrimary: '#0F172A' }` |
| 辅助函数 | 动词开头 | `add()`, `verify()`, `createCard()` |
| 累积坐标 | `currentY` / `currentX` | `let currentY = 0` |

### 最佳实践：分层Board架构

```javascript
// 1. 创建根容器 (禁用clipContent防止裁剪)
const page = penpot.createBoard();
page.name = 'Page';
page.resize(1440, 900);
page.clipContent = false;  // ⚠️ 关键！禁用裁剪
penpot.root.appendChild(page);

// 2. 创建分层结构 - 每个Board都禁用clipContent
const layerConfig = [
  { name: 'Background', y: 0, h: 900 },
  { name: 'Header', y: 0, h: 80 },
  { name: 'Hero', y: 80, h: 480 },
  { name: 'Gallery', y: 560, h: 320 },
  { name: 'Footer', y: 880, h: 20 }
];

const layers = {};
layerConfig.forEach(cfg => {
  const board = penpot.createBoard();
  board.name = cfg.name;
  board.x = 0;
  board.y = cfg.y;
  board.resize(1440, cfg.h);
  board.clipContent = false;  // ⚠️ 每个子Board都要禁用裁剪
  page.appendChild(board);
  layers[cfg.name] = board;
});

// 3. 验证层级
console.log('页面层级:');
page.children.forEach((layer, i) => {
  console.log(`  ${i}: ${layer.name} (y=${layer.y}, h=${layer.height}, clipContent=${layer.clipContent})`);
});
```

### 累积 Y 坐标：currentY 变量（推荐！）

**核心优势：自动累积高度，无需手动计算每个区域的 y 坐标，避免算错**

```javascript
// ═════════════════════════════════════════════════════
// 方案对比：手动坐标 vs 累积 currentY
// ═════════════════════════════════════════════════════

// ❌ 传统方式：手动计算每个 y 坐标（容易算错）
const nav = penpot.createBoard();
nav.resize(1440, 80);
nav.y = 0;   // 手动设置
page.appendChild(nav);

const hero = penpot.createBoard();
hero.resize(1440, 500);
hero.y = 80;   // 需要精确计算 0 + 80
page.appendChild(hero);

const gallery = penpot.createBoard();
gallery.resize(1440, 400);
gallery.y = 580;   // 需要精确计算 80 + 500 = 580（容易算错！）
page.appendChild(gallery);

// ✅ 优雅方式：使用 currentY 自动累积
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
currentY += 500;  // 累积高度

const gallery = penpot.createBoard();
gallery.resize(1440, 400);
gallery.y = currentY;  // 自动为 580
page.appendChild(gallery);
currentY += 400;  // 累积高度

console.log(`总高度: ${currentY}px`);  // 980
```

### clipContent 裁剪问题（必知！）

**Board 默认 clipContent=true，会裁剪超出边界的内容！**

```javascript
// ❌ 错误：子元素位置超出父Board范围时会被裁剪
const parent = penpot.createBoard();
parent.resize(500, 300);
penpot.root.appendChild(parent);

const child = penpot.createRectangle();
child.x = 100; child.y = 350;  // 超出父容器底部
child.resize(100, 100);
parent.appendChild(child);  // 超出部分被裁剪！

// ✅ 正确：创建Board后立即设置 clipContent = false
const parent = penpot.createBoard();
parent.resize(500, 300);
parent.clipContent = false;  // 关键！
penpot.root.appendChild(parent);
```

### 优雅定位方案：使用 Flex Layout（推荐！）

**核心优势：自动定位，无需手动计算坐标，避免溢出问题**

```javascript
// ═════════════════════════════════════════════════════
// 方案对比：手动坐标 vs Flex Layout
// ═════════════════════════════════════════════════════

// ❌ 传统方式：手动设置 x/y 坐标
const page = penpot.createBoard();
page.resize(1440, 900);
penpot.root.appendChild(page);

const nav = penpot.createBoard();
nav.x = 0; nav.y = 0;  // 手动设置
nav.resize(1440, 70);
page.appendChild(nav);

const hero = penpot.createBoard();
hero.x = 0; hero.y = 70;  // 需要精确计算
hero.resize(1440, 400);
page.appendChild(hero);

const gallery = penpot.createBoard();
gallery.x = 0; gallery.y = 470;  // 容易算错！
gallery.resize(1440, 300);
page.appendChild(gallery);

// ✅ 优雅方式：使用 Flex Layout 自动定位
const pageFlex = penpot.createBoard();
pageFlex.resize(1440, 900);
pageFlex.clipContent = false;
penpot.root.appendChild(pageFlex);

// 主容器 Column Flex - 子元素自动垂直排列
const mainFlex = pageFlex.addFlexLayout();
mainFlex.dir = 'column';
mainFlex.horizontalPadding = 60;  // 自动边距
mainFlex.verticalPadding = 40;

// 添加的子元素会自动按顺序排列，无需设置 y 坐标
const nav = penpot.createBoard();
nav.resize(1320, 70);
mainFlex.appendChild(nav);  // 自动排在第一位

const hero = penpot.createBoard();
hero.resize(1320, 380);
mainFlex.appendChild(hero);  // 自动排在第二位

const gallery = penpot.createBoard();
gallery.resize(1320, 280);
mainFlex.appendChild(gallery);  // 自动排在第三位
```

#### Flex Layout 完整示例

```javascript
// ═════════════════════════════════════════════════════
// 赛博朋克画廊页面 - Flex Layout 版
// ═════════════════════════════════════════════════════

// 1. 清理并创建页面
[...penpot.root.children].forEach(x => x.remove());
const page = penpot.createBoard();
page.name = 'Cyberpunk Gallery';
page.resize(1440, 900);
page.clipContent = false;
penpot.root.appendChild(page);

// 2. 主容器 - Column Flex
const mainFlex = page.addFlexLayout();
mainFlex.dir = 'column';
mainFlex.horizontalPadding = 60;
mainFlex.verticalPadding = 30;

// 3. Header
const header = penpot.createBoard();
header.name = 'Header';
header.resize(1320, 70);
header.fills = [{ fillColor: 'rgba(10,10,15,0.95)', fillOpacity: 1 }];
header.clipContent = false;
mainFlex.appendChild(header);

// Header 内部用 Row Flex
const headerFlex = header.addFlexLayout();
headerFlex.dir = 'row';
headerFlex.justifyContent = 'space-between';

const logo = penpot.createText('CYBER//ART');
logo.fontSize = 22;
logo.fontWeight = '800';
logo.fills = [{ fillColor: '#00FFFF' }];
headerFlex.appendChild(logo);

['GALLERY', 'ARTISTS', 'ABOUT', 'CONTACT'].forEach(item => {
  const nav = penpot.createText(item);
  nav.fontSize = 14;
  nav.fontWeight = '600';
  nav.fills = [{ fillColor: 'rgba(255,255,255,0.7)' }];
  headerFlex.appendChild(nav);
});

// 4. Hero
const hero = penpot.createBoard();
hero.name = 'Hero';
hero.resize(1320, 320);
hero.clipContent = false;
mainFlex.appendChild(hero);

const heroFlex = hero.addFlexLayout();
heroFlex.dir = 'column';
heroFlex.justifyContent = 'center';

const title = penpot.createText('DIGITAL ART REIMAGINED');
title.fontSize = 48;
title.fontWeight = '900';
title.fills = [{ fillColor: '#FFFFFF' }];
heroFlex.appendChild(title);

// 5. Gallery - Row Flex
const gallery = penpot.createBoard();
gallery.name = 'Gallery';
gallery.resize(1320, 220);
gallery.clipContent = false;
mainFlex.appendChild(gallery);

const galleryFlex = gallery.addFlexLayout();
galleryFlex.dir = 'row';
galleryFlex.columnGap = 20;

['Neon Dreams', 'Digital Soul', 'Cyber Bloom'].forEach(name => {
  const card = penpot.createBoard();
  card.name = `Card_${name}`;
  card.resize(420, 180);
  card.borderRadius = 8;
  card.fills = [{ fillColor: 'rgba(255,255,255,0.05)' }];
  card.clipContent = true;
  galleryFlex.appendChild(card);
});

// 6. Footer
const footer = penpot.createBoard();
footer.name = 'Footer';
footer.resize(1320, 60);
footer.fills = [{ fillColor: 'rgba(0,0,0,0.5)' }];
mainFlex.appendChild(footer);

// 验证布局
page.children.forEach(child => {
  console.log(`${child.name}: flex=${child.flex?.dir || 'none'}`);
});
```

#### API 查看最佳实践

**遇到不确定的 API 时，使用 `penpot_penpot_api_info`：**

```javascript
// 查看 Board 的完整 API
penpot_penpot_api_info({ type: 'Board' })

// 查看 FlexLayout 的属性
penpot_penpot_api_info({ type: 'FlexLayout' })
```

| 类型 | 查看方式 |
|------|----------|
| Board | `penpot_penpot_api_info({ type: 'Board' })` |
| FlexLayout | `penpot_penpot_api_info({ type: 'FlexLayout' })` |
| GridLayout | `penpot_penpot_api_info({ type: 'GridLayout' })` |

---

## Step 4: 添加内容

### 使用 appendChild 添加子元素

```javascript
const bg = layers['Background'];
const bgRect = penpot.createRectangle();
bgRect.name = 'BG';
bgRect.resize(1440, 900);
bgRect.fills = [{ fillColor: '#0D0D12', fillOpacity: 1 }];
bg.appendChild(bgRect);

const header = layers['Header'];
const headerBg = penpot.createRectangle();
headerBg.name = 'HeaderBg';
headerBg.resize(1440, 80);
headerBg.fills = [{ fillColor: 'rgba(255,255,255,0.03)', fillOpacity: 1 }];
header.appendChild(headerBg);

const logo = penpot.createText('BRAND');
logo.name = 'Logo';
logo.fontSize = '24';
logo.fontWeight = '700';
logo.x = 60; logo.y = 28;
logo.fills = [{ fillColor: '#FFFFFF' }];
header.appendChild(logo);
```

### 资源库使用

```javascript
// 查看已连接的库
const libs = penpot.library.connected;
console.log('已连接库:', libs.map(l => l.name));

// 使用 Phosphor Icons
const phosphor = libs.find(lib => lib.name.includes('Phosphor'));
const icon = phosphor.components.find(c => c.name.includes('Star') && c.name.includes('Fill'));
if (icon) {
  const instance = icon.instance();
  instance.name = 'Icon_Star';
  instance.resize(24, 24);
  header.appendChild(instance);
}
```

### 图片上传

```javascript
// 上传图片
const imgData = await penpot.uploadMediaUrl('HeroImage', 'https://picsum.photos/seed/hero/800/400');

// 使用图片填充（必须加 fillOpacity!）
const imgRect = penpot.createRectangle();
imgRect.name = 'HeroImage';
imgRect.resize(800, 400);
imgRect.borderRadius = 16;
imgRect.fills = [{ fillImage: imgData, fillOpacity: 1 }];
layers['Hero'].appendChild(imgRect);
```

---

## Step 5: 验证检查

### 辅助函数定义（推荐！）

**在代码块开头定义 `add()` 和 `verify()`，简化后续操作：**

```javascript
/**
 * 添加子元素到父容器并设置位置
 * @returns {Shape} 返回子元素，便于链式调用
 */
function add(parent, child, x, y) {
  parent.appendChild(child);
  penpotUtils.setParentXY(child, x, y);
  return child;
}

/**
 * 验证父容器子元素数量
 */
function verify(parent, expected) {
  const actual = parent.children?.length || 0;
  if (actual !== expected) {
    console.log(`⚠️ ${parent.name}: 期望 ${expected}, 实际 ${actual}`);
  }
}
```

### 实时验证模式（推荐！）

**核心原则：每添加一个元素后，立即验证嵌套正确性。**

```javascript
// ❌ 错误：批量添加后验证（难以定位问题）
add(mainBoard, nav, 0, 0);
add(mainBoard, hero, 0, 80);
add(mainBoard, gallery, 0, 580);
add(mainBoard, footer, 0, 980);
verify(mainBoard, 4);  // 出错时不知道是哪个元素

// ✅ 正确：每步验证（问题定位清晰）
add(mainBoard, nav, 0, 0);
verify(mainBoard, 1);  // 立即检查

add(mainBoard, hero, 0, 80);
verify(mainBoard, 2);  // 再次检查

add(mainBoard, gallery, 0, 580);
verify(mainBoard, 3);  // 继续检查

add(mainBoard, footer, 0, 980);
verify(mainBoard, 4);  // 最后检查
```

### 常规验证检查

```javascript
// 1. 检查Root元素数量
console.log('Root children:', penpot.root.children.length);

// 2. 检查层级结构
const structure = penpotUtils.shapeStructure(penpot.root, 4);
console.log(JSON.stringify(structure, null, 2));

// 3. 检查clipContent状态
const page = penpotUtils.findShape(s => s.name === 'Page');
console.log('Page clipContent:', page?.clipContent);

// 4. 检查各层位置和层级顺序
page.children.forEach((layer, i) => {
  console.log(`${i}: ${layer.name} (y=${layer.y}, h=${layer.height}, clip=${layer.clipContent})`);
});

// 5. 检查子元素是否超出父容器
const gallery = penpotUtils.findShape(s => s.name === 'Gallery');
gallery.children.forEach(child => {
  const absY = gallery.y + child.y;
  const overflow = absY + child.height > gallery.y + gallery.height;
  if (overflow) console.log(`⚠️ ${child.name} 超出边界!`);
});
```

### 导出功能限制（已知问题）

**问题**：`export_shape` 导出大尺寸页面可能失败

```javascript
// 可能出现的错误
Error: Error handling task: http error
```

**原因**：
- 导出尺寸过大（如 1920×2600）
- 元素数量过多（如 100+ 元素）
- Penpot MCP Plugin 连接限制

**解决方案**：
- ✅ 使用验证脚本确认设计正确性
- ✅ 直接在 Penpot UI 中查看
- ⚠️ 避免依赖导出功能

**最佳实践**：验证脚本比截图更可靠

---

## penpotUtils 工具集

### 常用工具速查

| 工具 | 用途 | 示例 |
|------|------|------|
| `setParentXY(shape, x, y)` | 设置相对位置 | `setParentXY(child, 100, 50)` |
| `addFlexLayout(board, dir)` | 添加 Flex（保持顺序）| `addFlexLayout(board, 'column')` |
| `findShape(predicate)` | 查找单个元素 | `findShape(s => s.name === 'Nav')` |
| `findShapes(predicate)` | 查找多个元素 | `findShapes(s => s.type === 'text')` |
| `shapeStructure(shape, depth)` | 查看结构树 | `shapeStructure(root, 3)` |

### setParentXY：设置相对位置

```javascript
// ❌ parentX/parentY 是只读属性
shape.parentX = 100;  // TypeError!

// ✅ 使用工具函数
penpotUtils.setParentXY(shape, 100, 50);
// 内部实现: shape.x = parent.x + 100; shape.y = parent.y + 50;
```

### addFlexLayout：添加 Flex（保持顺序）

```javascript
// ❌ 已有子元素时直接调用会打乱视觉顺序
const board = penpot.createBoard();
board.appendChild(child1);
board.appendChild(child2);
board.addFlexLayout();  // 子元素顺序可能被打乱！

// ✅ 使用工具函数保持现有顺序
penpotUtils.addFlexLayout(board, 'column');
```

### findShape / findShapes：查找元素

```javascript
// 按名字精确查找
const nav = penpotUtils.findShape(s => s.name === 'Navigation');

// 按名字模糊查找
const allCards = penpotUtils.findShapes(s => s.name?.includes('Card'));

// 按类型查找
const allTexts = penpotUtils.findShapes(s => s.type === 'text');
const allBoards = penpotUtils.findShapes(s => s.type === 'board');

// 按尺寸查找
const mainBoard = penpotUtils.findShape(s => s.width === 1920);
```

---

## 核心概念

### 坐标系统

- **x/y**: 绝对坐标（可写）
- **parentX/parentY**: 相对坐标（只读）
- **width/height**: 只读，用 `resize(w, h)` 修改
- 子元素坐标相对于父元素

### Z-Order

- `appendChild()` 添加到末尾 = 视觉顶层
- `insertChild(index, shape)` 可控制Z顺序
- `setParentIndex(index)` 调整已有元素顺序

### fills 设置

```javascript
// Rectangle 可立即设置
const rect = penpot.createRectangle();
rect.resize(100, 50);
penpot.root.appendChild(rect);
rect.fills = [{ fillColor: '#FF0000', fillOpacity: 1 }];

// Board 需要分批执行
const board = penpot.createBoard();
board.resize(100, 50);
penpot.root.appendChild(board);
// ... 稍后设置
board.fills = [{ fillColor: '#00FF00', fillOpacity: 1 }];
```

---

## 常见错误速查

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `Identifier 'xxx' has already been declared` | 变量命名冲突 | 使用语义化命名：`xxxSection`, `xxxList` |
| 变量在不同代码块丢失 | 跨调用上下文独立 | 使用单代码块策略或 storage |
| 导出失败 `http error` | 页面尺寸/元素过多 | 使用验证脚本替代导出 |
| 子元素被遮挡/裁剪 | 父Board的 clipContent=true | 创建Board后立即设置 `board.clipContent = false` |
| 子元素位置超出父容器 | 坐标计算错误 | 确保 `child.y + child.height <= parent.height`，使用累积 Y 坐标 |
| `Cannot add property fill` | Board 创建后立即设置 fills | 使用 Rectangle 替代，或分批执行 |
| Root 有多个元素 | 元素未正确添加到父容器 | 使用 verify() 检查层级 |
| 颜色不生效 | 小写颜色值 | 使用大写 `#FFFFFF` |
| 图片空白 | 缺少 fillOpacity | 添加 `fillOpacity: 1` |

---

## 完整示例：Landing Page

```javascript
// ===== 完整示例：创建 Landing Page =====

// 1. 清理
[...penpot.root.children].forEach(x => x.remove());

// 2. 创建根容器
const page = penpot.createBoard();
page.name = 'LandingPage';
page.resize(1440, 900);
page.clipContent = false;
penpot.root.appendChild(page);

// 3. 创建分层结构
const config = [
  { name: 'Background', y: 0, h: 900 },
  { name: 'Header', y: 0, h: 80 },
  { name: 'Hero', y: 80, h: 500 },
  { name: 'Features', y: 580, h: 280 },
  { name: 'Footer', y: 860, h: 40 }
];

const layers = {};
config.forEach(c => {
  const b = penpot.createBoard();
  b.name = c.name;
  b.x = 0; b.y = c.y;
  b.resize(1440, c.h);
  b.clipContent = false;
  page.appendChild(b);
  layers[c.name] = b;
});

// 4. 添加背景
const bg = penpot.createRectangle();
bg.name = 'BG';
bg.resize(1440, 900);
bg.fills = [{ fillColor: '#0D0D12', fillOpacity: 1 }];
layers['Background'].appendChild(bg);

// 5. 添加Header内容
const headerBg = penpot.createRectangle();
headerBg.name = 'HeaderBg';
headerBg.resize(1440, 80);
headerBg.fills = [{ fillColor: 'rgba(255,255,255,0.03)', fillOpacity: 1 }];
layers['Header'].appendChild(headerBg);

const logo = penpot.createText('AETHER');
logo.name = 'Logo';
logo.fontSize = '24';
logo.fontWeight = '700';
logo.x = 60; logo.y = 28;
logo.fills = [{ fillColor: '#FFFFFF' }];
layers['Header'].appendChild(logo);

// 6. 添加Hero内容
const headline = penpot.createText('Crafting Digital\nExperiences');
headline.name = 'Headline';
headline.fontSize = '56';
headline.fontWeight = '800';
headline.x = 60; headline.y = 40;
headline.fills = [{ fillColor: '#FFFFFF' }];
layers['Hero'].appendChild(headline);

const cta = penpot.createRectangle();
cta.name = 'CTA';
cta.x = 60; cta.y = 220;
cta.resize(140, 44);
cta.borderRadius = 22;
cta.fills = [{ fillColor: '#6366F1', fillOpacity: 1 }];
layers['Hero'].appendChild(cta);

// 7. 验证
console.log('✅ Landing Page 创建完成');
console.log('层级:', page.children.map(c => c.name));
```

---

## 详细文档

- 完整 API 参考 penpot mcp 
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
  - 使用组件库（Phosphor Icons、Wireframing Kit 等）
  - 添加交互和原型链接
---

# Penpot MCP 设计技能 v6

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

## 完整工作流

### 阶段 1：环境准备

```javascript
// === 环境准备 ===

// 1. 清理旧元素
[...penpot.root.children].forEach(c => c.remove());
console.log('✅ 环境已清理');

// 2. 创建新页面
const page = penpot.createPage();
page.name = 'ProjectName-PageName';
penpot.openPage(page);
console.log('✅ 页面已创建:', page.name);

// 3. 创建主画布
const mainBoard = penpot.createBoard();
mainBoard.name = 'MainBoard';  // ✅ 语义化命名
mainBoard.resize(1440, 900);
mainBoard.fills = [{ fillColor: '#0F172A', fillOpacity: 1 }];
mainBoard.clipContent = false;  // ⚠️ 必须禁用裁剪
penpot.root.appendChild(mainBoard);
console.log('✅ 主画布已创建');
```

### 阶段 2：资源准备

```javascript
// === 资源准备 ===

// 1. 检查连接的资源库
console.log('📚 检查资源库...');
const connectedLibs = penpot.library.connected;
console.log(`  已连接 ${connectedLibs.length} 个资源库`);

// 2. 查找常用资源库
const phosphorLib = connectedLibs.find(lib => lib.name.includes('Phosphor'));
const wireframeLib = connectedLibs.find(lib => lib.name.includes('Wireframing'));

if (phosphorLib) {
  console.log(`  ✅ Phosphor Icons: ${phosphorLib.components.length} 个组件`);
} else {
  console.log('  ⚠️ Phosphor Icons 未连接');
}

if (wireframeLib) {
  console.log(`  ✅ Wireframing Kit: ${wireframeLib.components.length} 个组件`);
} else {
  console.log('  ⚠️ Wireframing Kit 未连接');
}

// 3. 预加载常用图标（优化性能）
const requiredIcons = ['home-bold', 'search-bold', 'heart-bold', 'user-bold', 'settings-bold'];
const availableIcons = [];

if (phosphorLib) {
  requiredIcons.forEach(iconName => {
    const icon = phosphorLib.components.find(c => c.name === iconName);
    if (icon) {
      availableIcons.push(icon);
      console.log(`  ✅ ${iconName} 可用`);
    } else {
      console.log(`  ❌ ${iconName} 不可用`);
    }
  });
}

// 存储到 storage 供后续使用
storage.availableIcons = availableIcons;
storage.phosphorLib = phosphorLib;

// 4. 上传图片资源（示例）
const imageUrls = {
  heroBg: 'https://picsum.photos/seed/hero/1200/600',
  feature1: 'https://picsum.photos/seed/f1/400/300',
  avatar1: 'https://i.pravatar.cc/150?img=1'
};

const uploadedImages = {};
for (const [key, url] of Object.entries(imageUrls)) {
  try {
    uploadedImages[key] = await penpot.uploadMediaUrl(key, url);
    console.log(`  ✅ ${key} 上传成功`);
  } catch (e) {
    console.log(`  ❌ ${key} 上传失败: ${e.message}`);
  }
}

storage.uploadedImages = uploadedImages;

// 5. 初始化颜色系统
const colors = {
  primary: '#6366F1',
  bgPrimary: '#0F172A',
  bgSecondary: '#1E293B',
  bgCard: '#1C1C24',
  textPrimary: '#F1F5F9',
  textSecondary: '#94A3B8',
  success: '#10B981',
  danger: '#EF4444'
};

storage.colors = colors;
console.log('✅ 资源准备完成');
```

### 阶段 3：结构构建

```javascript
// === 结构构建 ===

const mainBoard = [...penpot.root.children].find(b => b.name === 'MainBoard');
const colors = storage.colors;
const availableIcons = storage.availableIcons || [];

// 辅助函数
function add(parent, child, x, y) {
  parent.appendChild(child);
  penpotUtils.setParentXY(child, x, y);
  return child;
}

// 使用 Flex Layout
const flex = mainBoard.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 40;
flex.rowGap = 20;

// 1. Header Section
const headerSection = penpot.createBoard();
headerSection.name = 'HeaderSection';  // ✅ 语义化命名
headerSection.resize(1320, 70);
headerSection.fills = [{ fillColor: colors.bgSecondary, fillOpacity: 1 }];
mainBoard.appendChild(headerSection);
console.log('✅ HeaderSection 已添加');

// 2. Hero Section
const heroSection = penpot.createBoard();
heroSection.name = 'HeroSection';  // ✅ 语义化命名
heroSection.resize(1320, 400);
heroSection.clipContent = false;
mainBoard.appendChild(heroSection);
console.log('✅ HeroSection 已添加');

// 3. 使用图标组件
if (availableIcons.length > 0) {
  const iconInstance = availableIcons[0].instance();
  iconInstance.name = 'HeroIcon';  // ✅ 语义化命名
  heroSection.appendChild(iconInstance);
  penpotUtils.setParentXY(iconInstance, 100, 150);
  iconInstance.fills = [{ fillColor: colors.primary, fillOpacity: 1 }];
  console.log('✅ 图标已添加');
}

console.log('✅ 结构构建完成');
```

### 阶段 4：验证

```javascript
// === 验证 ===

// 1. 命名规范检查
console.log('\n📋 命名规范检查:');
mainBoard.children.forEach((child, i) => {
  const hasSuffix = child.name.match(/(Section|Container|List|Grid|Card|Icon)/);
  const isPascalCase = child.name[0] === child.name[0].toUpperCase();
  
  const namingOk = hasSuffix && isPascalCase;
  console.log(`  ${i+1}. ${child.name} ${namingOk ? '✅' : '⚠️'}`);
  
  if (!namingOk) {
    console.log(`      建议: 添加后缀或使用 PascalCase`);
  }
});

// 2. 层级结构检查
console.log('\n🏗️  层级结构检查:');
const structure = penpotUtils.shapeStructure(mainBoard, 3);
console.log(JSON.stringify(structure, null, 2));

// 3. clipContent 检查
console.log('\n✂️  clipContent 检查:');
mainBoard.children.forEach((child, i) => {
  const status = child.clipContent ? '❌ 需禁用' : '✅ 正确';
  console.log(`  ${i+1}. ${child.name}: ${status}`);
  
  if (child.clipContent) {
    child.clipContent = false;
    console.log(`      已自动修复`);
  }
});

// 4. 资源检查
console.log('\n📚 资源检查:');
console.log(`  颜色系统: ${storage.colors ? '✅ 已初始化' : '❌ 未初始化'}`);
console.log(`  图片资源: ${Object.keys(storage.uploadedImages || {}).length} 个`);
console.log(`  可用图标: ${storage.availableIcons?.length || 0} 个`);

// 5. 最终报告
console.log('\n📊 最终报告:');
console.log(`  总层数: ${mainBoard.children.length}`);
console.log(`  页面高度: ${mainBoard.height}px`);
console.log(`  验证状态: ✅ 通过`);

console.log('\n✅ 验证完成');
```

---

## 核心概念

### 坐标系统

```javascript
// 坐标系统完整示例
const parent = penpot.createBoard();
parent.resize(800, 600);

const child = penpot.createBoard();
child.resize(200, 150);

// ❌ 错误：parentX 是只读属性
child.parentX = 100;  // TypeError!

// ✅ 正确：使用工具函数
penpotUtils.setParentXY(child, 100, 50);

// ✅ 正确：设置绝对坐标（相对于画布）
child.x = 100;
child.y = 50;

console.log(`相对位置: (${child.parentX}, ${child.parentY})`);
console.log(`绝对位置: (${child.x}, ${child.y})`);
```

### Board fills 最佳实践

```javascript
// Board fills 使用场景对比

// 场景 1：纯色背景（推荐直接设置 fills）
const simpleSection = penpot.createBoard();
simpleSection.name = 'SimpleSection';
simpleSection.resize(800, 400);
simpleSection.fills = [{ fillColor: '#1E293B', fillOpacity: 1 }];

// 场景 2：渐变背景（推荐使用 Rectangle 子元素）
const gradientSection = penpot.createBoard();
gradientSection.name = 'GradientSection';
gradientSection.resize(800, 400);

const bgRect = penpot.createRectangle();
bgRect.resize(800, 400);
bgRect.fills = [{
  fillColor: '#1E293B',
  fillOpacity: 1,
  // 可以添加渐变等其他效果
}];
gradientSection.appendChild(bgRect);

// 场景 3：图片背景（必须使用 Rectangle 子元素）
const imageSection = penpot.createBoard();
imageSection.name = 'ImageSection';
imageSection.resize(800, 400);

const imgRect = penpot.createRectangle();
imgRect.resize(800, 400);
const imageData = await penpot.uploadMediaUrl('bg', 'https://...');
imgRect.fills = [{ fillImage: imageData, fillOpacity: 1 }];
imageSection.appendChild(imgRect);
```

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

## 元素命名规范

```javascript
// 📋 命名规范代码示例

// ✅ 1. 区域命名（Section 后缀）
const heroSection = penpot.createBoard();
heroSection.name = 'HeroSection';        // 首屏区域
const gallerySection = penpot.createBoard();
gallerySection.name = 'GallerySection';   // 画廊区域
const footerSection = penpot.createBoard();
footerSection.name = 'FooterSection';     // 页脚区域

// ✅ 2. 列表命名（List/Grid 后缀）
const galleryGrid = penpot.createBoard();
galleryGrid.name = 'GalleryGrid';         // 网格布局
const cardList = penpot.createBoard();
cardList.name = 'CardList';               // 卡片列表
const artistList = penpot.createBoard();
artistList.name = 'ArtistList';           // 艺术家列表

// ✅ 3. 卡片命名（Card + 序号/类型）
const card1 = penpot.createBoard();
card1.name = 'ArtCard_1';                 // 带序号
const card2 = penpot.createBoard();
card2.name = 'FeatureCard_Hero';         // 带类型

// ✅ 4. 容器命名（Container 后缀）
const contentContainer = penpot.createBoard();
contentContainer.name = 'ContentContainer';
const navContainer = penpot.createBoard();
navContainer.name = 'NavContainer';

// ✅ 5. 装饰元素（Decor/Overlay 后缀）
const bgDecor = penpot.createRectangle();
bgDecor.name = 'BackgroundDecor';        // 背景装饰
const overlay = penpot.createBoard();
overlay.name = 'DarkOverlay';            // 叠加层

// ❌ 避免的命名
const board1 = penpot.createBoard();
board1.name = 'Board1';                   // ❌ 不够语义化
const container = penpot.createBoard();
container.name = 'container';             // ❌ 小写开头
const a = penpot.createBoard();
a.name = 'a';                            // ❌ 无意义命名
```

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

## 最小可运行模板

```javascript
// === 80行核心模板（包含资源准备和命名规范）===

// 1. 清理环境
[...penpot.root.children].forEach(x => x.remove());

// 2. 创建页面容器
const page = penpot.createBoard();
page.name = 'LandingPage';  // ✅ 语义化命名
page.resize(1440, 900);
page.clipContent = false;  // ⚠️ 必须禁用裁剪
penpot.root.appendChild(page);

// 3. 资源准备：检查组件库
const phosphorLib = penpot.library.connected.find(lib => 
  lib.name.includes('Phosphor')
);
console.log('📚 Phosphor Icons 库:', phosphorLib ? '已连接' : '未找到');

// 预加载常用图标（可选）
const iconNames = ['home-bold', 'heart-bold', 'user-bold'];
iconNames.forEach(name => {
  const icon = phosphorLib?.components.find(c => c.name === name);
  if (icon) console.log(`  ✅ ${name} 可用`);
});

// 4. 添加 Flex 布局
const flex = page.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 40;
flex.rowGap = 20;

// 5. 添加 Header Section
const headerSection = penpot.createBoard();  // ✅ 语义化命名
headerSection.name = 'HeaderSection';
headerSection.resize(1320, 70);
headerSection.fills = [{ fillColor: '#0A0A0F', fillOpacity: 1 }];
page.appendChild(headerSection);
console.log('✅ HeaderSection 添加完成');

// Header 内部布局
const headerFlex = headerSection.addFlexLayout();
headerFlex.dir = 'row';
headerFlex.justifyContent = 'space-between';

const logo = penpot.createText('BRAND');
logo.name = 'LogoText';  // ✅ 语义化命名
logo.fontSize = 22;
logo.fontWeight = '800';
logo.fills = [{ fillColor: '#FFFFFF' }];
headerSection.appendChild(logo);

// 6. 添加 Hero Section
const heroSection = penpot.createBoard();  // ✅ 语义化命名
heroSection.name = 'HeroSection';
heroSection.resize(1320, 400);
heroSection.clipContent = false;
page.appendChild(heroSection);
console.log('✅ HeroSection 添加完成');

const title = penpot.createText('HELLO WORLD');
title.name = 'HeroTitle';  // ✅ 语义化命名
title.fontSize = 48;
title.fontWeight = '800';
title.fills = [{ fillColor: '#FFFFFF' }];
heroSection.appendChild(title);

// 7. 添加图标示例（使用组件库）
if (phosphorLib) {
  const homeIconComp = phosphorLib.components.find(c => c.name === 'home-bold');
  if (homeIconComp) {
    const homeIcon = homeIconComp.instance();
    homeIcon.name = 'HomeIcon';  // ✅ 语义化命名
    heroSection.appendChild(homeIcon);
    penpotUtils.setParentXY(homeIcon, 200, 150);
    homeIcon.fills = [{ fillColor: '#8B5CF6', fillOpacity: 1 }];
    console.log('✅ HomeIcon 添加完成');
  }
}

// 8. 验证：检查命名规范
console.log('\n📋 命名检查:');
page.children.forEach((child, i) => {
  const hasSuffix = child.name.includes('Section') || 
                      child.name.includes('Container') ||
                      child.name.includes('List') ||
                      child.name.includes('Grid');
  console.log(`  ${i+1}. ${child.name} ${hasSuffix ? '✅' : '⚠️'}`);
});

// 9. 最终验证
console.log('\n✅ 完成:', page.children.length, '层');
console.log('层级:', page.children.map(c => c.name).join(' → '));
```

---

## 完整示例：使用组件库的 Landing Page

```javascript
// === 完整示例（120行，包含组件库和命名规范）===

// 阶段 1: 环境准备
[...penpot.root.children].forEach(c => c.remove());

const page = penpot.createBoard();
page.name = 'LandingPage';
page.resize(1440, 1000);
page.clipContent = false;
penpot.root.appendChild(page);

// 阶段 2: 资源准备
const phosphorLib = penpot.library.connected.find(lib => 
  lib.name.includes('Phosphor')
);

const colors = {
  primary: '#6366F1',
  bgPrimary: '#0F172A',
  bgSecondary: '#1E293B',
  textPrimary: '#FFFFFF'
};

const iconsNeeded = ['home-bold', 'heart-bold', 'star-bold', 'user-bold'];
const availableIcons = [];

if (phosphorLib) {
  iconsNeeded.forEach(name => {
    const icon = phosphorLib.components.find(c => c.name === name);
    if (icon) availableIcons.push(icon);
  });
}

// 阶段 3: 结构构建
const flex = page.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 40;
flex.rowGap = 24;

// Header Section
const headerSection = penpot.createBoard();
headerSection.name = 'HeaderSection';
headerSection.resize(1320, 70);
headerSection.fills = [{ fillColor: colors.bgSecondary, fillOpacity: 1 }];
page.appendChild(headerSection);

const logo = penpot.createText('BRAND');
logo.name = 'LogoText';
logo.fontSize = 22;
logo.fontWeight = '800';
logo.fills = [{ fillColor: colors.textPrimary }];
headerSection.appendChild(logo);

// Hero Section
const heroSection = penpot.createBoard();
heroSection.name = 'HeroSection';
heroSection.resize(1320, 500);
heroSection.clipContent = false;
page.appendChild(heroSection);

const title = penpot.createText('Welcome to Our Platform');
title.name = 'HeroTitle';
title.fontSize = 56;
title.fontWeight = '800';
title.fills = [{ fillColor: colors.textPrimary }];
heroSection.appendChild(title);

// 添加图标
if (availableIcons.length >= 2) {
  const icon1 = availableIcons[0].instance();
  icon1.name = 'FeatureIcon_1';
  heroSection.appendChild(icon1);
  penpotUtils.setParentXY(icon1, 0, 150);
  icon1.fills = [{ fillColor: colors.primary, fillOpacity: 1 }];
  
  const icon2 = availableIcons[1].instance();
  icon2.name = 'FeatureIcon_2';
  heroSection.appendChild(icon2);
  penpotUtils.setParentXY(icon2, 200, 150);
  icon2.fills = [{ fillColor: colors.primary, fillOpacity: 1 }];
}

// Features Section
const featuresSection = penpot.createBoard();
featuresSection.name = 'FeaturesSection';
featuresSection.resize(1320, 300);
featuresSection.fills = [{ fillColor: colors.bgSecondary, fillOpacity: 1 }];
page.appendChild(featuresSection);

const featuresGrid = featuresSection.addGridLayout();
featuresGrid.columns = [{ type: 'fixed', value: 400 }];
featuresGrid.rows = [
  { type: 'fixed', value: 120 },
  { type: 'fixed', value: 120 }
];
featuresGrid.columnGap = 24;
featuresGrid.rowGap = 24;

const featureCards = [];
for (let i = 0; i < 4; i++) {
  const card = penpot.createBoard();
  card.name = `FeatureCard_${i + 1}`;
  card.resize(400, 120);
  card.fills = [{ fillColor: colors.bgPrimary, fillOpacity: 1 }];
  card.borderRadius = 12;
  featuresSection.appendChild(card);
  
  featuresGrid.appendChild(card, Math.floor(i / 2) + 1, (i % 2) + 1);
  featureCards.push(card);
}

// 阶段 4: 验证
console.log('📋 命名检查:');
page.children.forEach((child, i) => {
  const hasSuffix = child.name.includes('Section');
  console.log(`  ${i+1}. ${child.name} ${hasSuffix ? '✅' : '⚠️'}`);
});

console.log('\n✅ 完成:', page.children.length, '层');
console.log('层级:', page.children.map(c => c.name).join(' → '));
console.log('可用图标:', availableIcons.length);
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
| 命名不规范 | 未使用语义化命名 | 使用 `xxxSection`, `xxxList` 等后缀 |

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
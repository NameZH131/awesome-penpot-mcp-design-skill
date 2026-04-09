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
  - 使用组件库创建可复用组件
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
| **语义命名** | **所有元素必须命名！** | 无法维护、修复或查找元素 |

---

## 最小可运行模板（50 行核心代码）

```javascript
// ═══════════════════════════════════════════════════════════════
// 1. 清理环境
// ═══════════════════════════════════════════════════════════════
[...penpot.root.children].forEach(x => x.remove());

// ═══════════════════════════════════════════════════════════════
// 2. 创建页面容器
// ═══════════════════════════════════════════════════════════════
const page = penpot.createBoard();
page.name = 'LandingPage';           // ⚠️ 必须命名
page.resize(1440, 900);
page.clipContent = false;            // ⚠️ 必须禁用裁剪
penpot.root.appendChild(page);

// ═══════════════════════════════════════════════════════════════
// 3. 添加 Flex 布局
// ═══════════════════════════════════════════════════════════════
const flex = page.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 40;

// ═══════════════════════════════════════════════════════════════
// 4. 添加 Header
// ═══════════════════════════════════════════════════════════════
const header = penpot.createBoard();
header.name = 'HeaderSection';
header.resize(1320, 70);
header.fills = [{ fillColor: '#0A0A0F', fillOpacity: 1 }];
page.appendChild(header);            // Flex 自动定位，无需设置 y

// Header 内部使用 Flex 水平布局
const headerFlex = header.addFlexLayout();
headerFlex.dir = 'row';
headerFlex.justifyContent = 'space-between';

const logo = penpot.createText('BRAND');
logo.name = 'LogoText';
logo.fontSize = 22;
logo.fontWeight = '800';
logo.fills = [{ fillColor: '#FFFFFF' }];
header.appendChild(logo);

['Home', 'About', 'Contact'].forEach((item, i) => {
  const nav = penpot.createText(item);
  nav.name = `NavItem-${i + 1}`;
  nav.fontSize = 14;
  nav.fills = [{ fillColor: 'rgba(255,255,255,0.7)' }];
  header.appendChild(nav);
});

// ═══════════════════════════════════════════════════════════════
// 5. 添加 Hero 区域
// ═══════════════════════════════════════════════════════════════
const hero = penpot.createBoard();
hero.name = 'HeroSection';
hero.resize(1320, 400);
hero.clipContent = false;
page.appendChild(hero);

const title = penpot.createText('Welcome to\nOur Platform');
title.name = 'HeroTitle';
title.fontSize = 56;
title.fontWeight = '800';
title.fills = [{ fillColor: '#FFFFFF' }];
hero.appendChild(title);

const subtitle = penpot.createText('Discover amazing features');
subtitle.name = 'HeroSubtitle';
subtitle.fontSize = 18;
subtitle.fills = [{ fillColor: 'rgba(255,255,255,0.7)' }];
hero.appendChild(subtitle);

const cta = penpot.createRectangle();
cta.name = 'PrimaryBtn';
cta.resize(140, 44);
cta.borderRadius = 22;
cta.fills = [{ fillColor: '#6366F1', fillOpacity: 1 }];
cta.y = 200;
hero.appendChild(cta);

const ctaText = penpot.createText('Get Started');
ctaText.name = 'PrimaryBtnText';
ctaText.fontSize = 14;
ctaText.fontWeight = '600';
ctaText.fills = [{ fillColor: '#FFFFFF' }];
cta.appendChild(ctaText);

// ═══════════════════════════════════════════════════════════════
// 6. 验证
// ═══════════════════════════════════════════════════════════════
console.log('✅ Landing Page 完成');
console.log('层级:', page.children.map(c => c.name).join(' → '));
console.log('Header 子元素:', header.children.map(c => c.name).join(', '));
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
board.name = 'MyBoard';
board.clipContent = false;           // 第一件事！
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

### 方案一：Flex Layout（推荐）

```javascript
const page = penpot.createBoard();
page.name = 'Page';
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

### 方案二：currentY 累积坐标（备用）

```javascript
let currentY = 0;  // 从 0 开始

const nav = penpot.createBoard();
nav.name = 'NavSection';
nav.resize(1440, 80);
nav.y = currentY;
page.appendChild(nav);
currentY += 80;  // 累积高度

const hero = penpot.createBoard();
hero.name = 'HeroSection';
hero.resize(1440, 500);
hero.y = currentY;
page.appendChild(hero);
currentY += 500;

console.log(`总高度: ${currentY}px`);  // 580
```

---

## 工具函数（penpotUtils）

| 工具 | 用途 | 示例 |
|------|------|------|
| `setParentXY(shape, x, y)` | 设置相对位置 | `setParentXY(child, 100, 50)` |
| `addFlexLayout(board, dir)` | 添加 Flex（保持顺序）| `addFlexLayout(board, 'column')` |
| `findShape(predicate)` | 查找单个元素 | `findShape(s => s.name === 'Nav')` |
| `findShapes(predicate)` | 查找多个元素 | `findShapes(s => s.name?.includes('Card'))` |
| `shapeStructure(shape, depth)` | 查看结构树 | `shapeStructure(root, 3)` |

### 使用示例

```javascript
// ❌ parentX/parentY 是只读属性
shape.parentX = 100;  // TypeError!

// ✅ 使用工具函数
penpotUtils.setParentXY(shape, 100, 50);

// 精确查找
const nav = penpotUtils.findShape(s => s.name === 'HeaderSection');

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
header.name = 'HeaderSection';
header.resize(1320, 70);
page.appendChild(header);
console.log('✅ Header 添加完成, 当前层数:', page.children.length);
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
if (gallery) {
  gallery.children.forEach(child => {
    const overflow = child.y + child.height > gallery.height;
    if (overflow) console.log(`⚠️ ${child.name} 超出边界!`);
  });
}

// 5. 检查所有元素是否都有命名
const allShapes = penpotUtils.findShapes(s => true);
const unnamed = allShapes.filter(s => !s.name || s.name === s.type);
if (unnamed.length > 0) {
  console.log(`⚠️ ${unnamed.length} 个元素未命名:`, unnamed.map(s => s.id));
} else {
  console.log('✅ 所有元素都已命名');
}
```

---

## 扩展模块（高级功能）

<details>
<summary><b>📦 命名规范详解（点击展开）</b></summary>

### 为什么命名很重要？

1. **优雅修复**：Root 有孤立元素时，可根据名字智能归类
2. **可维护性**：便于后续查找和修改
3. **团队协作**：其他设计师能理解结构

### 命名规范速查表

| 元素类型 | 命名规则 | 示例 |
|---------|---------|------|
| **页面容器** | `XxxPage` | `LandingPage`, `DashboardPage` |
| **区域 Board** | `XxxSection` / `XxxArea` | `HeroSection`, `SidebarArea` |
| **列表容器** | `XxxList` / `XxxGrid` | `CardList`, `NavList`, `GalleryGrid` |
| **卡片** | `XxxCard-序号` | `ProductCard-1`, `ArtistCard-2` |
| **标题文本** | `XxxTitle` | `HeroTitle`, `SectionTitle` |
| **副标题** | `XxxSubtitle` | `HeroSubtitle`, `CardSubtitle` |
| **标签/说明** | `XxxLabel` / `XxxDesc` | `PriceLabel`, `CardDesc` |
| **按钮** | `XxxBtn` | `PrimaryBtn`, `SubmitBtn`, `HeartBtn` |
| **图标** | `XxxIcon` | `SearchIcon`, `HeartIcon`, `LogoIcon` |
| **图片** | `XxxImage` / `XxxPhoto` | `HeroImage`, `GalleryImage-1` |
| **头像** | `XxxAvatar` | `UserAvatar`, `ArtistAvatar` |
| **装饰元素** | `XxxGlow` / `XxxDeco` | `HeroGlow`, `BgPattern` |

### 命名示例

```javascript
// ✅ 正确：所有元素都有语义化命名
const page = penpot.createBoard();
page.name = 'LandingPage';

const hero = penpot.createBoard();
hero.name = 'HeroSection';

const title = penpot.createText('Welcome');
title.name = 'HeroTitle';

const ctaBtn = penpot.createRectangle();
ctaBtn.name = 'PrimaryBtn';

const heartIcon = penpot.createShapeFromSvg(svg);
heartIcon.name = 'HeartIcon';
```

</details>

<details>
<summary><b>🧩 组件库使用（Penpot Library）</b></summary>

### 获取组件库

```javascript
// 1. 获取所有已连接的组件库
const libs = penpot.library.connected;
console.log('已连接库:', libs.map(l => l.name).join(', '));

// 2. 查找特定库
const pencilLib = libs.find(l => l.name.includes('Pencil'));
const phosphorLib = libs.find(l => l.name.includes('Phosphor'));

// 3. 查看库中的组件
if (pencilLib) {
  console.log('组件列表:', pencilLib.components.map(c => c.name).join(', '));
}
```

### 使用组件

```javascript
// 1. 查找组件
const button = pencilLib.components.find(c => c.name.includes('Button'));
const card = pencilLib.components.find(c => c.name.includes('Card'));

// 2. 创建实例
const buttonInstance = button.instance();
buttonInstance.name = 'PrimaryBtn';  // ⚠️ 必须命名！

// 3. 添加到页面
parent.appendChild(buttonInstance);

// 4. 如果需要修改组件内部，先 detach
buttonInstance.detach();  // 解除与主组件的关联
```

### 本地库：创建自己的组件

```javascript
const localLib = penpot.library.local;

// 创建颜色资源
const brandColor = localLib.createColor();
brandColor.name = 'Brand Primary';
brandColor.color = '#6366F1';

// 创建排版样式
const headingStyle = localLib.createTypography();
headingStyle.name = 'Heading Large';
headingStyle.fontFamily = 'Inter';
headingStyle.fontWeight = '700';
headingStyle.fontSize = '32';

// 创建组件（从已有 shapes）
const shapesToComponent = [headerSection, logo, navItems];
const navComponent = localLib.createComponent(shapesToComponent);
navComponent.name = 'NavigationBar';
```

### 组件库速查表

| 操作 | 代码 |
|------|------|
| 获取连接库 | `penpot.library.connected` |
| 获取本地库 | `penpot.library.local` |
| 查找组件 | `lib.components.find(c => c.name.includes('Button'))` |
| 创建实例 | `component.instance()` |
| 解除关联 | `instance.detach()` |
| 创建颜色 | `localLib.createColor()` |
| 创建排版 | `localLib.createTypography()` |
| 创建组件 | `localLib.createComponent(shapes)` |

</details>

<details>
<summary><b>🛠️ 优雅修复孤立元素</b></summary>

### 问题背景

Root 下出现孤立元素的原因：
1. 跨调用时变量丢失
2. `appendChild` 未成功执行
3. 元素被创建但未添加到正确父容器

### 根据名字智能归类

```javascript
function elegantlyFixOrphans() {
  const root = penpot.root;
  const rootChildren = [...root.children];
  
  // Step 1: 找到主画布
  const mainBoard = rootChildren.find(c => 
    c.type === 'board' && c.width >= 1400
  );
  
  if (!mainBoard) {
    console.log('❌ 未找到主画布');
    return { error: 'No main board' };
  }
  
  console.log(`📦 主画布: ${mainBoard.name}`);
  console.log(`   子元素: ${mainBoard.children?.length || 0} 个`);
  console.log(`   孤立元素: ${rootChildren.length - 1} 个\n`);
  
  // Step 2: 建立父子映射（根据命名规范）
  const parentMapping = {
    // 侧边栏元素
    'LogoArea': 'SidebarSection',
    'NavList': 'SidebarSection',
    'UserArea': 'SidebarSection',
    // Hero 区域元素
    'HeroImage': 'HeroSection',
    'HeroTitle': 'HeroSection',
    'HeroSubtitle': 'HeroSection',
    'PrimaryBtn': 'HeroSection',
    // Gallery 区域元素
    'GalleryTitle': 'GallerySection',
    'GalleryGrid': 'GallerySection',
    'GalleryCard-1': 'GalleryGrid',
    'GalleryCard-2': 'GalleryGrid',
    // ... 根据实际命名添加更多
  };
  
  // Step 3: 遍历孤立元素，智能归类
  const results = { moved: [], cannotRecognize: [] };
  
  rootChildren.forEach(child => {
    if (child === mainBoard) return;
    
    const targetParentName = parentMapping[child.name];
    
    if (targetParentName) {
      const targetParent = penpotUtils.findShape(s => s.name === targetParentName);
      if (targetParent) {
        targetParent.appendChild(child);
        penpotUtils.setParentXY(child, 0, 0);
        console.log(`✅ ${child.name} → ${targetParentName}`);
        results.moved.push(child.name);
      } else {
        console.log(`⚠️ 目标未找到: ${targetParentName}`);
        results.cannotRecognize.push(child.name);
      }
    } else {
      results.cannotRecognize.push(child.name);
    }
  });
  
  // Step 4: 最终清理（只删除无法识别的空元素）
  const finalOrphans = [...root.children].filter(c => c !== mainBoard);
  finalOrphans.forEach(c => {
    if (results.cannotRecognize.includes(c.name)) {
      c.remove();
      console.log(`🗑️ 删除孤立元素: ${c.name}`);
    }
  });
  
  console.log(`✅ 移动: ${results.moved.length} 个, 删除: ${results.cannotRecognize.length} 个`);
  return results;
}

// 使用
elegantlyFixOrphans();
```

### 根据位置自动推断（备选方案）

如果命名不规范，可以用位置和尺寸推断：

```javascript
function smartFixByPosition() {
  const root = penpot.root;
  const mainBoard = root.children.find(c => c.type === 'board' && c.width >= 1400);
  if (!mainBoard) return;
  
  // 获取主画布的已知区域
  const sections = {
    sidebar: penpotUtils.findShape(s => s.name?.includes('Sidebar')),
    hero: penpotUtils.findShape(s => s.name?.includes('Hero')),
    gallery: penpotUtils.findShape(s => s.name?.includes('Gallery')),
    footer: penpotUtils.findShape(s => s.name?.includes('Footer')),
  };
  
  root.children.forEach(child => {
    if (child === mainBoard) return;
    const { x, y, width, height } = child;
    let target = null;
    
    if (x < 300 && width < 300) target = sections.sidebar;
    else if (y < 800 && height > 200) target = sections.hero;
    else if (y >= 700 && y < 1500) target = sections.gallery;
    else if (y >= 2000) target = sections.footer;
    
    if (target) {
      target.appendChild(child);
      penpotUtils.setParentXY(child, 0, 0);
      console.log(`✅ ${child.name} → ${target.name}`);
    }
  });
}
```

</details>

---

## 常见错误速查

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `Identifier 'xxx' has already been declared` | 变量命名冲突 | 使用语义化命名：`xxxSection`, `xxxList` |
| 变量在不同代码块丢失 | 跨调用上下文独立 | 使用单代码块策略 |
| 导出失败 `http error` | 页面尺寸/元素过多 | 使用验证脚本替代导出 |
| 子元素被遮挡/裁剪 | 父Board的 clipContent=true | 创建Board后立即 `clipContent = false` |
| 子元素位置超出父容器 | 坐标计算错误 | 使用 Flex Layout 或 currentY |
| `Cannot add property fill` | Board 创建后立即设置 fills | 使用 Rectangle 替代 |
| Root 有多个元素 | 元素未正确添加到父容器 | 使用 `elegantlyFixOrphans()` |
| 颜色不生效 | 小写颜色值 | 使用大写 `#FFFFFF` |
| 图片空白 | 缺少 fillOpacity | 添加 `fillOpacity: 1` |
| 无法找到元素 | 元素未命名 | 所有元素必须设置 `name` |

---

## API 查看

遇到不确定的 API 时：

```javascript
// 查看 Board 完整 API
penpot_api_info({ type: 'Board' })

// 查看 FlexLayout 属性
penpot_api_info({ type: 'FlexLayout' })

// 查看 GridLayout 属性
penpot_api_info({ type: 'GridLayout' })

// 查看 Library 组件库 API
penpot_api_info({ type: 'Library' })

// 查看 LibraryComponent API
penpot_api_info({ type: 'LibraryComponent' })
```
---

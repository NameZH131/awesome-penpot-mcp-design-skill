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

# Penpot MCP 设计技能 v4

通过 Penpot MCP 服务器在 Penpot 设计工具中创建 UI 设计。

**前置条件**：用户必须通过 Penpot MCP Plugin 连接 Penpot 设计项目到 MCP 服务器。

---

## 黄金法则（记住这 6 条）

| 法则 | 说明 | 违反后果 |
|------|------|---------|
| **单代码块** | 所有代码在一个 `execute_code` 中完成 | 变量丢失，需要 storage 传递 |
| **禁用裁剪** | 每个 Board 创建后立即 `clipContent = false` | 子元素被裁剪，显示不全 |
| **Flex 优先** | 能用 Flex 就不用手动坐标 | 手动计算 y 坐标容易出错 |
| **实时验证** | 每添加关键元素后 console.log 检查 | 出错时难以定位问题 |
| **语义命名** | **所有元素必须命名！** | 无法优雅修复孤立元素 |
| **优雅修复** | 孤立元素按名字归类，不直接删除 | 误删有效元素 |

---

## 元素命名规范（v4 新增）

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
| **图片** | `XxxImage` / `XxxPhoto` | `HeroImage`, `GalleryImage-1`, `AvatarPhoto` |
| **头像** | `XxxAvatar` | `UserAvatar`, `ArtistAvatar` |
| **装饰元素** | `XxxGlow` / `XxxDeco` | `HeroGlow`, `BgPattern` |

### 命名示例代码

```javascript
// ✅ 正确：所有元素都有语义化命名
const page = penpot.createBoard();
page.name = 'LandingPage';           // 页面

const hero = penpot.createBoard();
hero.name = 'HeroSection';           // 区域

const title = penpot.createText('Welcome');
title.name = 'HeroTitle';            // 标题

const subtitle = penpot.createText('Discover amazing art');
subtitle.name = 'HeroSubtitle';      // 副标题

const heroImage = penpot.createRectangle();
heroImage.name = 'HeroImage';        // 图片

const ctaBtn = penpot.createRectangle();
ctaBtn.name = 'PrimaryBtn';          // 按钮

const heartIcon = penpot.createShapeFromSvg(svg);
heartIcon.name = 'HeartIcon';        // 图标

const heroGlow = penpot.createEllipse();
heroGlow.name = 'HeroGlow';          // 装饰
```

---

## 组件库使用（v4 新增）

### 获取组件库

```javascript
// ═══════════════════════════════════════════════════════════════
// 组件库使用完整流程
// ═══════════════════════════════════════════════════════════════

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
// ═══════════════════════════════════════════════════════════════
// 创建组件实例
// ═══════════════════════════════════════════════════════════════

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
// 现在可以自由修改内部元素了
```

### 本地库：创建自己的组件

```javascript
// ═══════════════════════════════════════════════════════════════
// 本地库操作
// ═══════════════════════════════════════════════════════════════

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

| 操作 | 代码 | 说明 |
|------|------|------|
| 获取连接库 | `penpot.library.connected` | 返回 Library[] |
| 获取本地库 | `penpot.library.local` | 当前文件的库 |
| 查找组件 | `lib.components.find(c => c.name.includes('Button'))` | 按名称查找 |
| 创建实例 | `component.instance()` | 返回 Shape |
| 获取主实例 | `component.mainInstance()` | 获取主组件 |
| 解除关联 | `instance.detach()` | 允许修改内部 |
| 创建颜色 | `localLib.createColor()` | 返回 LibraryColor |
| 创建排版 | `localLib.createTypography()` | 返回 LibraryTypography |
| 创建组件 | `localLib.createComponent(shapes)` | 从 shapes 创建 |

---

## 优雅修复孤立元素（v4 新增）

### 问题背景

Root 下出现孤立元素的原因：
1. 跨调用时变量丢失
2. `appendChild` 未成功执行
3. 元素被创建但未添加到正确父容器

### ❌ 错误做法：直接删除

```javascript
// ❌ 会误删有效元素！
const rootChildren = [...penpot.root.children];
rootChildren.forEach(child => {
  if (child !== mainBoard) {
    child.remove();  // 危险！
  }
});
```

### ✅ 正确做法：根据名字智能归类

```javascript
// ═══════════════════════════════════════════════════════════════
// 优雅修复孤立元素
// ═══════════════════════════════════════════════════════════════

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
  
  // Step 2: 根据命名规范建立父子映射
  const parentMapping = buildParentMapping();
  
  // Step 3: 遍历孤立元素，智能归类
  const results = { moved: [], cannotRecognize: [], deleted: [] };
  
  rootChildren.forEach(child => {
    if (child === mainBoard) return;
    
    console.log(`🔍 处理: ${child.name}`);
    
    // 根据名字查找目标父元素
    const targetParentName = parentMapping[child.name];
    
    if (targetParentName) {
      const targetParent = penpotUtils.findShape(s => s.name === targetParentName);
      
      if (targetParent) {
        // 移动到正确的父元素
        targetParent.appendChild(child);
        penpotUtils.setParentXY(child, 0, 0);
        console.log(`   ✅ → ${targetParentName}`);
        results.moved.push({ name: child.name, to: targetParentName });
      } else {
        console.log(`   ⚠️ 目标父元素未找到: ${targetParentName}`);
        results.cannotRecognize.push(child.name);
      }
    } else {
      // 无法识别的元素
      console.log(`   ❓ 无法识别，保留`);
      results.cannotRecognize.push(child.name);
    }
  });
  
  // Step 4: 最终清理 - 只删除确认无用的元素
  const finalCheck = [...root.children].filter(c => c !== mainBoard);
  if (finalCheck.length > 0 && results.cannotRecognize.length === 0) {
    finalCheck.forEach(c => {
      c.remove();
      results.deleted.push(c.name);
    });
  }
  
  // 验证结果
  console.log('\n═══════════════════════════════════════════');
  console.log('        修复结果');
  console.log('═══════════════════════════════════════════');
  console.log(`移动: ${results.moved.length} 个`);
  console.log(`无法识别: ${results.cannotRecognize.length} 个`);
  console.log(`删除: ${results.deleted.length} 个`);
  console.log(`Root 子元素: ${[...root.children].length} (应为 1)`);
  
  return results;
}

// 构建父子元素名字映射
function buildParentMapping() {
  return {
    // ═══ 侧边栏元素 ═══
    'LogoArea': 'SidebarSection',
    'NavList': 'SidebarSection',
    'UserArea': 'SidebarSection',
    'NavItem-Home': 'NavList',
    'NavItem-Gallery': 'NavList',
    'NavItem-Artists': 'NavList',
    'NavItem-Settings': 'NavList',
    
    // ═══ Hero 区域元素 ═══
    'HeroImage': 'HeroSection',
    'HeroOverlay': 'HeroSection',
    'HeroTitle': 'HeroSection',
    'HeroSubtitle': 'HeroSection',
    'HeroContent': 'HeroSection',
    'HeroGlow': 'HeroSection',
    'PrimaryBtn': 'HeroSection',
    'SecondaryBtn': 'HeroSection',
    
    // ═══ Gallery 区域元素 ═══
    'GalleryTitle': 'GallerySection',
    'GallerySubtitle': 'GallerySection',
    'GalleryGrid': 'GallerySection',
    'GalleryCard-1': 'GalleryGrid',
    'GalleryCard-2': 'GalleryGrid',
    'GalleryCard-3': 'GalleryGrid',
    'GalleryCard-4': 'GalleryGrid',
    'GalleryCard-5': 'GalleryGrid',
    'GalleryCard-6': 'GalleryGrid',
    'GalleryImage-1': 'GalleryCard-1',
    'GalleryImage-2': 'GalleryCard-2',
    'CardTitle-1': 'GalleryCard-1',
    'CardTitle-2': 'GalleryCard-2',
    'ArtistLabel-1': 'GalleryCard-1',
    'PriceLabel-1': 'GalleryCard-1',
    'HeartBtn-1': 'GalleryCard-1',
    
    // ═══ Artists 区域元素 ═══
    'ArtistsTitle': 'ArtistsSection',
    'ArtistsSubtitle': 'ArtistsSection',
    'ArtistsGrid': 'ArtistsSection',
    'ArtistCard-1': 'ArtistsGrid',
    'ArtistCard-2': 'ArtistsGrid',
    'ArtistCard-3': 'ArtistsGrid',
    'ArtistCard-4': 'ArtistsGrid',
    'ArtistAvatar-1': 'ArtistCard-1',
    'ArtistName-1': 'ArtistCard-1',
    'ArtistStyle-1': 'ArtistCard-1',
    'FollowBtn-1': 'ArtistCard-1',
    
    // ═══ Footer 区域元素 ═══
    'FooterContent': 'FooterSection',
    'BrandArea': 'FooterContent',
    'FooterLogo': 'BrandArea',
    'FooterDesc': 'BrandArea',
    'SocialIcons': 'BrandArea',
    'LinkGroup-1': 'FooterContent',
    'LinkGroup-2': 'FooterContent',
    'LinkGroup-3': 'FooterContent',
    'CopyrightText': 'FooterContent',
  };
}
```

### 智能修复：根据位置推断

如果元素命名不规范，可以用位置和尺寸推断：

```javascript
// ═══════════════════════════════════════════════════════════════
// 智能修复 - 根据元素特征自动归类
// ═══════════════════════════════════════════════════════════════

function smartFixOrphans() {
  const root = penpot.root;
  const rootChildren = [...root.children];
  
  const mainBoard = rootChildren.find(c => c.type === 'board' && c.width >= 1400);
  if (!mainBoard) return { error: 'No main board' };
  
  // 获取主画布的已知区域
  const sections = {
    sidebar: penpotUtils.findShape(s => 
      s.name?.includes('Sidebar') || s.name?.includes('侧边')
    ),
    hero: penpotUtils.findShape(s => s.name?.includes('Hero')),
    gallery: penpotUtils.findShape(s => s.name?.includes('Gallery')),
    artists: penpotUtils.findShape(s => s.name?.includes('Artist')),
    footer: penpotUtils.findShape(s => s.name?.includes('Footer')),
    content: penpotUtils.findShape(s => s.name?.includes('Content')),
  };
  
  console.log('找到区域:', Object.entries(sections)
    .filter(([k, v]) => v)
    .map(([k, v]) => `${k}: ${v.name}`)
    .join(', ')
  );
  
  const results = { moved: [], toMain: [], deleted: [] };
  
  rootChildren.forEach(child => {
    if (child === mainBoard) return;
    
    const { x, y, width, height } = child;
    let targetSection = null;
    let reason = '';
    
    // 根据位置和尺寸推断归属
    if (x < 300 && width < 300) {
      targetSection = sections.sidebar;
      reason = '左侧窄元素 → Sidebar';
    } 
    else if (y < 800 && height > 200) {
      targetSection = sections.hero;
      reason = '顶部大元素 → Hero';
    }
    else if (y >= 700 && y < 1500) {
      targetSection = sections.gallery;
      reason = '中部位置 → Gallery';
    }
    else if (y >= 1500 && y < 2000) {
      targetSection = sections.artists;
      reason = '下部位置 → Artists';
    }
    else if (y >= 2000) {
      targetSection = sections.footer;
      reason = '底部位置 → Footer';
    }
    
    if (targetSection) {
      targetSection.appendChild(child);
      penpotUtils.setParentXY(child, 0, 0);
      console.log(`✅ ${child.name}: ${reason}`);
      results.moved.push({ name: child.name, to: targetSection.name });
    } else {
      // 备选：添加到内容区域或主画布
      const fallback = sections.content || mainBoard;
      fallback.appendChild(child);
      console.log(`⚠️ ${child.name} → ${fallback.name}（无法推断）`);
      results.toMain.push(child.name);
    }
  });
  
  // 最终清理
  [...root.children]
    .filter(c => c !== mainBoard)
    .forEach(c => {
      c.remove();
      results.deleted.push(c.name);
    });
  
  return results;
}
```

---

## 最小可运行模板

```javascript
// === 50行核心模板 ===

// 1. 清理环境
[...penpot.root.children].forEach(x => x.remove());

// 2. 创建页面容器
const page = penpot.createBoard();
page.name = 'LandingPage';           // ⚠️ 命名
page.resize(1440, 900);
page.clipContent = false;            // ⚠️ 禁用裁剪
penpot.root.appendChild(page);

// 3. 添加 Flex 布局
const flex = page.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 40;

// 4. 添加 Header
const header = penpot.createBoard();
header.name = 'HeaderSection';       // ⚠️ 命名
header.resize(1320, 70);
header.fills = [{ fillColor: '#0A0A0F', fillOpacity: 1 }];
page.appendChild(header);

// 5. 添加 Hero
const hero = penpot.createBoard();
hero.name = 'HeroSection';           // ⚠️ 命名
hero.resize(1320, 400);
hero.clipContent = false;
page.appendChild(hero);

const title = penpot.createText('HELLO WORLD');
title.name = 'HeroTitle';            // ⚠️ 文字命名
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
board.name = 'MyBoard';              // 命名
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

### 三选一决策表

| 方案 | 适用场景 | 代码量 | 维护成本 | 推荐度 |
|------|---------|--------|---------|--------|
| **Flex Layout** | 多子元素自动排列 | 少 | 低 | ⭐⭐⭐ |
| **currentY** | 简单垂直堆叠 | 中 | 中 | ⭐⭐ |
| **手动坐标** | 特殊定位需求 | 多 | 高 | ⭐ |

### 方案一：Flex Layout（推荐）

```javascript
const page = penpot.createBoard();
page.name = 'Page';                  // 命名
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
nav.name = 'NavSection';            // 命名
nav.resize(1440, 80);
nav.y = currentY;
page.appendChild(nav);
currentY += 80;  // 累积高度

const hero = penpot.createBoard();
hero.name = 'HeroSection';          // 命名
hero.resize(1440, 500);
hero.y = currentY;  // 自动为 80
page.appendChild(hero);
currentY += 500;

console.log(`总高度: ${currentY}px`);  // 580
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

const hero = penpot.createBoard();
hero.name = 'HeroSection';
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

// 5. 检查所有元素是否都有命名
const allShapes = penpotUtils.findShapes(s => true);
const unnamed = allShapes.filter(s => !s.name || s.name === s.type);
if (unnamed.length > 0) {
  console.log(`⚠️ ${unnamed.length} 个元素未命名:`, unnamed.map(s => s.id));
}
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
| Root 有多个元素 | 元素未正确添加到父容器 | 使用 `elegantlyFixOrphans()` |
| 颜色不生效 | 小写颜色值 | 使用大写 `#FFFFFF` |
| 图片空白 | 缺少 fillOpacity | 添加 `fillOpacity: 1` |
| 无法找到元素 | 元素未命名 | 所有元素必须设置 `name` |

---

## 完整示例：Landing Page（带完整命名）

```javascript
// === Landing Page 完整示例（80行）===

// 1. 清理
[...penpot.root.children].forEach(x => x.remove());

// 2. 创建页面
const page = penpot.createBoard();
page.name = 'LandingPage';           // ✅ 命名
page.resize(1440, 900);
page.clipContent = false;
penpot.root.appendChild(page);

const flex = page.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 30;

// 3. Header
const header = penpot.createBoard();
header.name = 'HeaderSection';       // ✅ 命名
header.resize(1320, 70);
header.fills = [{ fillColor: 'rgba(10,10,15,0.95)', fillOpacity: 1 }];
page.appendChild(header);

const headerFlex = header.addFlexLayout();
headerFlex.dir = 'row';
headerFlex.justifyContent = 'space-between';

const logo = penpot.createText('BRAND');
logo.name = 'LogoText';              // ✅ 命名
logo.fontSize = 22;
logo.fontWeight = '800';
logo.fills = [{ fillColor: '#FFFFFF' }];
header.appendChild(logo);

['Home', 'About', 'Contact'].forEach((item, i) => {
  const nav = penpot.createText(item);
  nav.name = `NavItem-${i + 1}`;     // ✅ 命名
  nav.fontSize = 14;
  nav.fills = [{ fillColor: 'rgba(255,255,255,0.7)' }];
  header.appendChild(nav);
});

// 4. Hero
const hero = penpot.createBoard();
hero.name = 'HeroSection';           // ✅ 命名
hero.resize(1320, 400);
hero.clipContent = false;
page.appendChild(hero);

const title = penpot.createText('Welcome to\nOur Platform');
title.name = 'HeroTitle';            // ✅ 命名
title.fontSize = 56;
title.fontWeight = '800';
title.fills = [{ fillColor: '#FFFFFF' }];
hero.appendChild(title);

const subtitle = penpot.createText('Discover amazing features');
subtitle.name = 'HeroSubtitle';      // ✅ 命名
subtitle.fontSize = 18;
subtitle.fills = [{ fillColor: 'rgba(255,255,255,0.7)' }];
hero.appendChild(subtitle);

// 5. CTA Button
const cta = penpot.createRectangle();
cta.name = 'PrimaryBtn';             // ✅ 命名
cta.resize(140, 44);
cta.borderRadius = 22;
cta.fills = [{ fillColor: '#6366F1', fillOpacity: 1 }];
cta.y = 200;
hero.appendChild(cta);

const ctaText = penpot.createText('Get Started');
ctaText.name = 'PrimaryBtnText';     // ✅ 命名
ctaText.fontSize = 14;
ctaText.fontWeight = '600';
ctaText.fills = [{ fillColor: '#FFFFFF' }];
cta.appendChild(ctaText);

// 6. 验证
console.log('✅ Landing Page 完成');
console.log('层级:', page.children.map(c => c.name).join(' → '));
console.log('Header 子元素:', header.children.map(c => c.name).join(', '));

// 7. 检查命名完整性
const allShapes = penpotUtils.findShapes(s => true);
const unnamed = allShapes.filter(s => !s.name || s.name === s.type);
console.log(`命名检查: ${unnamed.length === 0 ? '✅ 全部命名' : `⚠️ ${unnamed.length} 个未命名`}`);
```

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

## v4 更新日志

| 更新内容 | 说明 |
|---------|------|
| **元素命名规范** | 所有元素（文字、图片、图标等）必须设置 `name` |
| **组件库使用** | 新增完整的组件库使用指南 |
| **优雅修复孤立元素** | 根据元素名字智能归类，不直接删除 |
| **黄金法则 6 条** | 新增「语义命名」和「优雅修复」法则 |
| **验证增强** | 新增命名完整性检查 |

---

## 详细文档

完整 API 参考见 Penpot MCP 文档。

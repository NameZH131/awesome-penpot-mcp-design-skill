---
name: penpot-design-multi-board
description: |
  使用 Penpot MCP 创建和管理多个独立 Board 的 UI 设计。核心能力：Board 管理、自动布局、Storage 持久化、验证机制。适用于 App 多页面设计、设计系统、响应式设计。
---

# Penpot MCP 多 Board 设计

## 核心类：BoardManager

```javascript
/**
 * Board 管理器 - 用于管理 Penpot 中的多个 Board
 * 
 * 核心功能：
 * 1. 创建/获取 Board（避免重复）
 * 2. 自动计算布局位置
 * 3. Storage 持久化
 * 4. 验证和清理
 */
class BoardManager {
  constructor() {
    this.currentPage = penpot.currentPage;
    this.boardConfig = new Map();
    this.storageKey = 'board-manager-v7';
    this.loadFromStorage();
  }
  
  loadFromStorage() {
    try {
      const saved = this.currentPage.getPluginData(this.storageKey);
      if (saved) {
        this.boardConfig = new Map(JSON.parse(saved));
        console.log(`📦 加载了 ${this.boardConfig.size} 个 Board 配置`);
      }
    } catch (e) {
      console.warn('⚠️ 从 storage 加载失败:', e.message);
    }
  }
  
  saveToStorage() {
    try {
      const data = JSON.stringify(Array.from(this.boardConfig));
      this.currentPage.setPluginData(this.storageKey, data);
      console.log(`💾 已保存 ${this.boardConfig.size} 个 Board 配置`);
    } catch (e) {
      console.warn('⚠️ 保存到 storage 失败:', e.message);
    }
  }
  
  getAllBoards() {
    return this.currentPage.findShapes({ type: 'board' })
      .filter(b => b.name !== 'Root Frame');
  }
  
  getBoardByName(name) {
    const boards = this.currentPage.findShapes({ type: 'board', name });
    return boards.length > 0 ? boards[0] : null;
  }
  
  calculateNextX(gap = 475) {
    const boards = this.getAllBoards();
    let maxX = 0;
    boards.forEach(b => maxX = Math.max(maxX, b.x + b.width));
    return maxX + gap;
  }
  
  createOrGetBoard(config) {
    const { name, width = 375, height = 812, x, y = 409 } = config;
    let board = this.getBoardByName(name);
    
    if (!board) {
      board = penpot.createBoard();
      board.name = name;
      board.resize(width, height);
      board.y = y;
      board.x = x !== undefined ? x : this.calculateNextX();
      board.clipContent = false;
      
      this.boardConfig.set(name, { width, height, x: board.x, y: board.y });
      this.saveToStorage();
      console.log(`✅ 创建: ${name} (${board.x.toFixed(0)}, ${board.y.toFixed(0)})`);
    } else {
      console.log(`📋 复用: ${name}`);
    }
    
    return board;
  }
  
  validateBoard(board) {
    const children = board.children || [];
    const hasText = children.some(c => c.type === 'text');
    const hasShapes = children.length > 20;
    const hasBackground = children.some(c => 
      c.type === 'rectangle' && c.x === 0 && c.y === 0
    );
    
    return {
      isValid: hasText && hasShapes && hasBackground,
      childrenCount: children.length,
      hasText,
      hasShapes,
      hasBackground
    };
  }
  
  getBoardStatus() {
    const boards = this.getAllBoards();
    return boards.map(board => {
      const validation = this.validateBoard(board);
      return {
        name: board.name,
        id: board.id,
        x: board.x,
        y: board.y,
        width: board.width,
        height: board.height,
        ...validation
      };
    });
  }
  
  cleanupDuplicates() {
    const boards = this.getAllBoards();
    const nameMap = new Map();
    const toRemove = [];
    
    boards.forEach(board => {
      const existing = nameMap.get(board.name);
      if (existing) {
        const bc = board.children?.length || 0;
        const ec = existing.children?.length || 0;
        if (bc > ec) {
          toRemove.push(existing);
          nameMap.set(board.name, board);
        } else {
          toRemove.push(board);
        }
      } else {
        nameMap.set(board.name, board);
      }
    });
    
    toRemove.forEach(board => {
      try {
        board.remove();
        if (this.boardConfig.has(board.name)) {
          this.boardConfig.delete(board.name);
          this.saveToStorage();
        }
      } catch (e) {}
    });
    
    return { removed: toRemove.length, kept: nameMap.size };
  }
  
  getStorageConfig() {
    return Object.fromEntries(this.boardConfig);
  }
  
  validateStorage() {
    const storageBoards = Array.from(this.boardConfig.keys());
    const actualBoards = this.getAllBoards().map(b => b.name);
    
    return {
      storageCount: storageBoards.length,
      actualCount: actualBoards.length,
      missingInStorage: actualBoards.filter(n => !storageBoards.includes(n)),
      missingInActual: storageBoards.filter(n => !actualBoards.includes(n)),
      isConsistent: storageBoards.length === actualBoards.length
    };
  }
  
  syncStorage() {
    const validation = this.validateStorage();
    
    validation.missingInActual.forEach(name => {
      this.boardConfig.delete(name);
    });
    
    validation.missingInStorage.forEach(name => {
      const board = this.getBoardByName(name);
      if (board) {
        this.boardConfig.set(name, {
          width: board.width,
          height: board.height,
          x: board.x,
          y: board.y
        });
      }
    });
    
    this.saveToStorage();
    return { synced: true, validation: this.validateStorage() };
  }
}
```

## 工作流模板 1：基础多 Board 创建

```javascript
// === 工作流 1：基础多 Board 创建 ===

// 1. 初始化
const manager = new BoardManager();
console.log('📦 BoardManager 已初始化');

// 2. 清理重复
const cleanup = manager.cleanupDuplicates();
console.log(`清理: 移除 ${cleanup.removed}, 保留 ${cleanup.kept}`);

// 3. 创建 Board 1
const musicBoard = manager.createOrGetBoard({
  name: 'Music App',
  width: 375,
  height: 812,
  x: 455,
  y: 409
});

const bg1 = penpot.createRectangle();
bg1.resize(375, 812);
bg1.fills = [{ fillColor: '#667eea', fillOpacity: 1 }];
musicBoard.appendChild(bg1);

// 4. 创建 Board 2
const profileBoard = manager.createOrGetBoard({
  name: 'Profile',
  width: 375,
  height: 812,
  x: 930,
  y: 409
});

const bg2 = penpot.createRectangle();
bg2.resize(375, 812);
bg2.fills = [{ fillColor: '#00c6ff', fillOpacity: 1 }];
profileBoard.appendChild(bg2);

// 5. 创建 Board 3
const searchBoard = manager.createOrGetBoard({
  name: 'Search',
  width: 375,
  height: 812,
  x: 1405,
  y: 409
});

const bg3 = penpot.createRectangle();
bg3.resize(375, 812);
bg3.fills = [{ fillColor: '#f093fb', fillOpacity: 1 }];
searchBoard.appendChild(bg3);

// 6. 验证
const status = manager.getBoardStatus();
console.log('\n📊 状态:', status.map(b => 
  `${b.name} (${b.children} 元素 ${b.isValid ? '✅' : '❌'})`
).join(', '));

const storageConfig = manager.getStorageConfig();
console.log('💾 Storage:', Object.keys(storageConfig).join(', '));
```

## 工作流模板 2：带 Storage 验证

```javascript
// === 工作流 2：带 Storage 验证 ===

// 1. 初始化
const manager = new BoardManager();

// 2. 验证 Storage 一致性
console.log('🔍 验证 Storage...');
const validation = manager.validateStorage();
console.log(`  Storage: ${validation.storageCount}, 实际: ${validation.actualCount}`);
console.log(`  一致性: ${validation.isConsistent ? '✅' : '❌'}`);

if (!validation.isConsistent) {
  console.log('⚠️ 不同步，正在同步...');
  const syncResult = manager.syncStorage();
  console.log(`  同步完成: ${syncResult.validation.isConsistent ? '✅' : '❌'}`);
}

// 3. 创建 Board
const board = manager.createOrGetBoard({
  name: 'Music App',
  width: 375,
  height: 812,
  x: 455,
  y: 409
});

// 4. 添加内容
const bg = penpot.createRectangle();
bg.resize(375, 812);
bg.fills = [{ fillColor: '#667eea', fillOpacity: 1 }];
board.appendChild(bg);

// 5. 显示 Storage 状态
const storageConfig = manager.getStorageConfig();
console.log('\n💾 Storage 配置:');
Object.entries(storageConfig).forEach(([name, config]) => {
  console.log(`  ${name}: (${config.x.toFixed(0)}, ${config.y.toFixed(0)})`);
});
```

## 工作流模板 3：任务函数模式

```javascript
// === 工作流 3：任务函数模式 ===

// 1. 定义任务函数
function createMusicTask(board, colors) {
  const bg = penpot.createRectangle();
  bg.resize(375, 812);
  bg.fills = [{ fillColor: colors.purple, fillOpacity: 1 }];
  board.appendChild(bg);
  
  const title = penpot.createText('Music');
  title.fontSize = 24;
  title.fontWeight = '700';
  title.fills = [{ fillColor: '#ffffff' }];
  title.x = 20;
  title.y = 50;
  board.appendChild(title);
  
  return { hasBackground: true, children: board.children.length };
}

function createProfileTask(board, colors) {
  const bg = penpot.createRectangle();
  bg.resize(375, 812);
  bg.fills = [{ fillColor: colors.blue, fillOpacity: 1 }];
  board.appendChild(bg);
  
  const title = penpot.createText('Profile');
  title.fontSize = 24;
  title.fontWeight = '700';
  title.fills = [{ fillColor: '#ffffff' }];
  title.x = 20;
  title.y = 50;
  board.appendChild(title);
  
  return { hasBackground: true, children: board.children.length };
}

// 2. 准备资源
const colors = {
  purple: '#667eea',
  blue: '#00c6ff',
  pink: '#f093fb'
};

// 3. 创建 Board 并执行任务
const manager = new BoardManager();

const musicBoard = manager.createOrGetBoard({
  name: 'Music App',
  x: 455
});
createMusicTask(musicBoard, colors);

const profileBoard = manager.createOrGetBoard({
  name: 'Profile',
  x: 930
});
createProfileTask(profileBoard, colors);

// 4. 验证
const status = manager.getBoardStatus();
console.log('📊 状态:', status.map(b => 
  `${b.name}: ${b.children} 元素`
).join(', '));
```

## 工作流模板 4：玻璃拟态设计

```javascript
// === 工作流 4：玻璃拟态设计 ===

// 1. 辅助函数
function createGlassCard(board, x, y, width, height, radius) {
  const card = penpot.createRectangle();
  card.resize(width, height);
  card.x = x;
  card.y = y;
  card.borderRadius = radius;
  card.fills = [{
    type: 'solid',
    fillColor: '#ffffff',
    fillOpacity: 0.12
  }];
  card.strokes = [{
    type: 'solid',
    strokeColor: '#ffffff',
    strokeOpacity: 0.25,
    strokeWidth: 1
  }];
  card.shadows = [{
    type: 'drop-shadow',
    color: '#000000',
    opacity: 0.15,
    blur: 25,
    offsetX: 0,
    offsetY: 8
  }];
  board.appendChild(card);
  return card;
}

function createGradientBoard(board, colors) {
  const bg = penpot.createRectangle();
  bg.resize(board.width, board.height);
  bg.fills = [{
    type: 'gradient',
    gradientType: 'linear',
    stops: colors.map((c, i) => ({
      position: i / (colors.length - 1),
      color: c
    })),
    x1: 0, y1: 0,
    x2: board.width,
    y2: board.height
  }];
  board.appendChild(bg);
  return bg;
}

// 2. 创建玻璃拟态 Board
const manager = new BoardManager();

const glassBoard = manager.createOrGetBoard({
  name: 'Glassmorphism Music',
  width: 375,
  height: 812,
  x: 455,
  y: 409
});

// 3. 添加内容
createGradientBoard(glassBoard, ['#667eea', '#f093fb', '#ffecd2']);
createGlassCard(glassBoard, 27, 120, 321, 321, 20);
createGlassCard(glassBoard, 27, 470, 321, 200, 20);

// 4. 添加文本
const title = penpot.createText('Now Playing');
title.fontSize = 28;
title.fontWeight = '700';
title.fills = [{ fillColor: '#ffffff' }];
title.x = 20;
title.y = 50;
glassBoard.appendChild(title);

// 5. 验证
const status = manager.getBoardStatus();
console.log('📊 玻璃拟态:', status.map(b => 
  `${b.name}: ${b.children} 元素 ${b.isValid ? '✅' : '❌'}`
).join(', '));
```

## 工作流模板 5：响应式设计

```javascript
// === 工作流 5：响应式设计 ===

// 1. 响应式配置
const responsiveConfig = [
  { name: 'Desktop', width: 1440, height: 900, x: 0 },
  { name: 'Tablet', width: 768, height: 1024, x: 1440 },
  { name: 'Mobile', width: 375, height: 812, x: 2220 }
];

// 2. 创建任务函数
function createResponsiveLayout(board, size) {
  const bg = penpot.createRectangle();
  bg.resize(board.width, board.height);
  bg.fills = [{ fillColor: '#0F172A', fillOpacity: 1 }];
  board.appendChild(bg);
  
  const title = penpot.createText(`Layout for ${size}`);
  title.fontSize = size === 'Mobile' ? 24 : 48;
  title.fontWeight = '700';
  title.fills = [{ fillColor: '#FFFFFF' }];
  title.x = 20;
  title.y = 50;
  board.appendChild(title);
  
  const info = penpot.createText(`${board.width}x${board.height}`);
  info.fontSize = 16;
  info.fontWeight = '500';
  info.fills = [{ fillColor: '#94A3B8', fillOpacity: 0.7 }];
  info.x = 20;
  info.y = size === 'Mobile' ? 80 : 110;
  board.appendChild(info);
  
  return { size, children: board.children.length };
}

// 3. 创建所有尺寸
const manager = new BoardManager();

responsiveConfig.forEach(config => {
  const board = manager.createOrGetBoard(config);
  createResponsiveLayout(board, config.name);
});

// 4. 验证
const status = manager.getBoardStatus();
console.log('📊 响应式:');
status.forEach((b, i) => {
  console.log(`  ${i+1}. ${b.name} (${b.width}x${b.height}): ${b.children} 元素`);
});
```

## 工作流模板 6：设计系统

```javascript
// === 工作流 6：设计系统 ===

// 1. 组件定义
const components = [
  { name: 'Buttons', width: 1200, height: 600 },
  { name: 'Inputs', width: 1200, height: 600 },
  { name: 'Cards', width: 1200, height: 600 },
  { name: 'Modals', width: 1200, height: 600 }
];

// 2. 创建组件展示函数
function createComponentShowcase(board, componentName) {
  const bg = penpot.createRectangle();
  bg.resize(board.width, board.height);
  bg.fills = [{ fillColor: '#0F172A', fillOpacity: 1 }];
  board.appendChild(bg);
  
  const title = penpot.createText(`Component: ${componentName}`);
  title.fontSize = 32;
  title.fontWeight = '700';
  title.fills = [{ fillColor: '#FFFFFF' }];
  title.x = 40;
  title.y = 40;
  board.appendChild(title);
  
  // 添加组件示例
  const card = penpot.createRectangle();
  card.resize(200, 100);
  card.x = 40;
  card.y = 100;
  card.borderRadius = 12;
  card.fills = [{ fillColor: '#1E293B', fillOpacity: 1 }];
  board.appendChild(card);
  
  return { component: componentName, children: board.children.length };
}

// 3. 创建所有组件 Board
const manager = new BoardManager();

components.forEach(comp => {
  const board = manager.createOrGetBoard({
    name: `Component_${comp.name}`,
    width: comp.width,
    height: comp.height,
    x: components.indexOf(comp) * 1300,
    y: 0
  });
  
  createComponentShowcase(board, comp.name);
});

// 4. 验证
const status = manager.getBoardStatus();
console.log('📊 设计系统:');
status.forEach((b, i) => {
  console.log(`  ${i+1}. ${b.name}: ${b.children} 元素`);
});
```

## 验证工具箱

```javascript
// === 验证工具箱 ===

// 1. 检查 Board 唯一性
function checkBoardUniqueness(manager) {
  const boards = manager.getAllBoards();
  const nameCount = {};
  boards.forEach(b => {
    nameCount[b.name] = (nameCount[b.name] || 0) + 1;
  });
  
  const duplicates = Object.entries(nameCount)
    .filter(([_, count]) => count > 1)
    .map(([name]) => name);
  
  console.log(`🔍 Board 唯一性: ${duplicates.length === 0 ? '✅' : '❌'}`);
  if (duplicates.length > 0) {
    console.log(`  重复: ${duplicates.join(', ')}`);
  }
  
  return { isUnique: duplicates.length === 0, duplicates };
}

// 2. 检查位置重叠
function checkPositionOverlap(manager) {
  const boards = manager.getAllBoards();
  const positions = boards.map(b => b.x).sort((a, b) => a - b);
  const gaps = [];
  
  for (let i = 1; i < positions.length; i++) {
    gaps.push(positions[i] - positions[i-1]);
  }
  
  const hasOverlap = gaps.some(g => g < 100);
  console.log(`🔍 位置重叠: ${hasOverlap ? '❌' : '✅'}`);
  console.log(`  间隔: ${gaps.map(g => g.toFixed(0)).join('px, ')}px`);
  
  return { hasOverlap, gaps, positions };
}

// 3. 检查内容完整性
function checkContentCompleteness(manager) {
  const boards = manager.getAllBoards();
  const allValid = boards.every(b => {
    const validation = manager.validateBoard(b);
    return validation.isValid;
  });
  
  console.log(`🔍 内容完整性: ${allValid ? '✅' : '❌'}`);
  
  return { allValid, boards: boards.map(b => manager.validateBoard(b)) };
}

// 4. 完整验证流程
function runFullValidation(manager) {
  console.log('🔍 开始完整验证...\n');
  
  const uniqueness = checkBoardUniqueness(manager);
  const overlap = checkPositionOverlap(manager);
  const content = checkContentCompleteness(manager);
  
  console.log('\n📊 验证结果:');
  console.log(`  Board 唯一性: ${uniqueness.isUnique ? '✅' : '❌'}`);
  console.log(`  位置无重叠: ${!overlap.hasOverlap ? '✅' : '❌'}`);
  console.log(`  内容完整: ${content.allValid ? '✅' : '❌'}`);
  
  return {
    overallValid: uniqueness.isUnique && !overlap.hasOverlap && content.allValid,
    uniqueness,
    overlap,
    content
  };
}

// 使用验证
const manager = new BoardManager();
const validation = runFullValidation(manager);
console.log(`\n✅ 总体验证: ${validation.overallValid ? '通过' : '失败'}`);
```

## 快速诊断工具

```javascript
// === 快速诊断工具 ===

// 1. 诊断重复 Board
function diagnoseDuplicates(manager) {
  const validation = manager.validateStorage();
  
  console.log('🔍 诊断重复 Board:');
  console.log(`  Storage 中: ${validation.storageCount}`);
  console.log(`  实际 Board: ${validation.actualCount}`);
  console.log(`  缺失 Storage: ${validation.missingInStorage.join(', ')}`);
  console.log(`  缺失实际: ${validation.missingInActual.join(', ')}`);
  
  if (!validation.isConsistent) {
    console.log('⚠️ 发现不一致，建议运行 manager.syncStorage()');
  }
  
  return validation;
}

// 2. 诊断 Board 状态
function diagnoseBoardStatus(manager) {
  const status = manager.getBoardStatus();
  
  console.log('🔍 诊断 Board 状态:');
  status.forEach((b, i) => {
    console.log(`  ${i+1}. ${b.name}:`);
    console.log(`     位置: (${b.x.toFixed(0)}, ${b.y.toFixed(0)})`);
    console.log(`     元素: ${b.childrenCount} 个`);
    console.log(`     有效性: ${b.isValid ? '✅' : '❌'}`);
    console.log(`     文本: ${b.hasText ? '有' : '无'}, 形状: ${b.hasShapes ? '有' : '无'}, 背景: ${b.hasBackground ? '有' : '无'}`);
  });
  
  return status;
}

// 3. 诊断 Storage 状态
function diagnoseStorageStatus(manager) {
  const storageConfig = manager.getStorageConfig();
  
  console.log('🔍 诊断 Storage 状态:');
  console.log(`  配置数量: ${Object.keys(storageConfig).length}`);
  
  Object.entries(storageConfig).forEach(([name, config]) => {
    console.log(`  ${name}:`);
    console.log(`    尺寸: ${config.width}x${config.height}`);
    console.log(`    位置: (${config.x.toFixed(0)}, ${config.y.toFixed(0)})`);
  });
  
  return storageConfig;
}

// 使用诊断
const manager = new BoardManager();

console.log('=== 诊断报告 ===\n');
diagnoseDuplicates(manager);
console.log('\n');
diagnoseBoardStatus(manager);
console.log('\n');
diagnoseStorageStatus(manager);
```

## Penpot MCP 验证工具

### MCP 验证检查清单

```javascript
// === MCP 验证检查清单 ===

// 1. clipContent 检查（Penpot MCP 关键）
function checkClipContent(manager) {
  const boards = manager.getAllBoards();
  console.log('✂️  clipContent 检查:');
  
  boards.forEach((board, i) => {
    const status = board.clipContent ? '❌ 需禁用' : '✅ 正确';
    console.log(`  ${i+1}. ${board.name}: ${status}`);
    
    // 自动修复
    if (board.clipContent) {
      board.clipContent = false;
      console.log(`      已自动修复`);
    }
  });
  
  return boards;
}

// 2. 命名规范检查（Penpot MCP 最佳实践）
function checkNamingConventions(manager) {
  const boards = manager.getAllBoards();
  console.log('\n📋 命名规范检查:');
  
  boards.forEach((board, i) => => {
    const hasSuffix = board.name.match(/(Section|Page|Board|App|Screen)/);
    const isPascalCase = board.name[0] === board.name[0].toUpperCase();
    
    const namingOk = hasSuffix && isPascalCase;
    console.log(`  ${i+1}. ${board.name} ${namingOk ? '✅' : '⚠️'}`);
    
    if (!namingOk) {
      console.log(`      建议: 添加 Page/Board/App 后缀或使用 PascalCase`);
    }
  });
  
  return boards;
}

// 3. 层级结构检查（Penpot MCP 工具）
function checkLayerStructure(manager) {
  const boards = manager.getAllBoards();
  console.log('\n🏗️  层级结构检查:');
  
  boards.forEach((board, i) => {
    const structure = penpotUtils.shapeStructure(board, 3);
    console.log(`  ${i+1}. ${board.name}:`);
    console.log(JSON.stringify(structure, null, 2));
  });
  
  return boards;
}

// 4. 资源检查（Penpot MCP 资源）
function checkResources() {
  console.log('\n📚 资源检查:');
  
  // 检查组件库
  const connectedLibs = penpot.library.connected;
  console.log(`  已连接 ${connectedLibs.length} 个资源库`);
  
  const phosphorLib = connectedLibs.find(lib => lib.name.includes('Phosphor'));
  if (phosphorLib) {
    console.log(`  ✅ Phosphor Icons: ${phosphorLib.components.length} 个组件`);
  } else {
    console.log(`  ⚠️ Phosphor Icons 未连接`);
  }
  
  const wireframeLib = connectedLibs.find(lib => lib.name.includes('Wireframing'));
  if (wireframeLib) {
    console.log(`  ✅ Wireframing Kit: ${wireframeLib.components.length} 个组件`);
  } else {
    console.log(`  ⚠️ Wireframing Kit 未连接`);
  }
  
  return { connectedLibs, phosphorLib, wireframeLib };
}

// 5. 填充格式检查（Penpot MCP 常见错误）
function checkFillFormat(manager) {
  const boards = manager.getAllBoards();
  console.log('\n🎨 填充格式检查:');
  
  let issues = 0;
  boards.forEach(board => {
    board.children.forEach(child => {
      if (child.fills) {
        child.fills.forEach((fill, j) => {
          // 检查是否缺少 fillOpacity
          if (fill.type === 'solid' && !fill.fillOpacity) {
            console.log(`  ⚠️ ${board.name} → ${child.name}: 缺少 fillOpacity`);
            issues++;
          }
        });
      }
    });
  });
  
  if (issues === 0) {
    console.log('  ✅ 所有填充格式正确');
  }
  
  return issues;
}

// 6. 完整 MCP 验证流程
function runMCPValidation(manager) {
  console.log('🔍 Penpot MCP 完整验证...\n');
  
  // 1. clipContent 检查
  checkClipContent(manager);
  
  // 2. 命名规范检查
  checkNamingConventions(manager);
  
  // 3. 层级结构检查
  checkLayerStructure(manager);
  
  // 4. 资源检查
  const resources = checkResources();
  
  // 5. 填充格式检查
  const fillIssues = checkFillFormat(manager);
  
  // 6. BoardManager 验证
  const status = manager.getBoardStatus();
  const validation = manager.validateStorage();
  
  // 7. 最终报告
  console.log('\n📊 MCP 验证报告:');
  console.log(`  总 Board 数: ${status.length}`);
  console.log(`  有效 Board: ${status.filter(b => b.isValid).length}/${status.length}`);
  console.log(`  Storage 一致性: ${validation.isConsistent ? '✅' : '❌'}`);
  console.log(`  填充问题: ${fillIssues} 个`);
  
  const overallValid = status.every(b => b.isValid) && 
                        validation.isConsistent && 
                        fillIssues === 0;
  
  console.log(`\n✅ MCP 验证完成: ${overallValid ? '通过' : '失败'}`);
  
  return { overallValid, status, validation, fillIssues, resources };
}

// 使用 MCP 验证
const manager = new BoardManager();
const mcpValidation = runMCPValidation(manager);
console.log(`\n🎯 验证结果: ${mcpValidation.overallValid ? '✅ 通过' : '❌ 失败'}`);
```

## MCP 专用检查

### 坐标系统检查

```javascript
// === 坐标系统检查 ===

function checkCoordinateSystem(manager) {
  const boards = manager.getAllBoards();
  console.log('📍 坐标系统检查:');
  
  boards.forEach((board, i) => {
    const children = board.children || [];
    console.log(`  ${i+1}. ${board.name}:`);
    
    // 检查子元素的相对位置是否正确
    children.forEach((child, j) => {
      const hasCorrectPosition = child.parentX >= 0 && 
                             child.parentY >= 0 &&
                             child.parentX <= board.width &&
                             child.parentY <= board.height;
      
      console.log(`    ${j+1}. ${child.name}: 位置 ${hasCorrectPosition ? '✅' : '❌'}`);
    });
  });
}

// 使用示例
const manager = new BoardManager();
checkCoordinateSystem(manager);
```

### fills 格式验证

```javascript
// === fills 格式验证 ===

function validateFills(element, elementName) {
  if (!element.fills || element.fills.length === 0) {
    console.log(`⚠️ ${elementName}: 无 fills`);
    return false;
  }
  
  let isValid = true;
  element.fills.forEach((fill, i) => {
    if (fill.type === 'solid' && !fill.fillOpacity) {
      console.log(`❌ ${elementName}: fill[${i}] 缺少 fillOpacity`);
      isValid = false;
    }
    
    if (fill.type === 'solid' && typeof fill.fillColor !== 'string') {
      console.log(`❌ ${elementName}: fill[${i}] fillColor 不是字符串`);
      isValid = false;
    }
    
    if (fill.type === 'gradient' && (!fill.stops || fill.stops.length === 0)) {
      console.log(`❌ ${elementName}: gradient 缺少 stops`);
      isValid = false;
    }
  });
  
  if (isValid) {
    console.log(`✅ ${elementName}: fills 格式正确`);
  }
  
  return isValid;
}

// 使用示例
const manager = new BoardManager();
const board = manager.createOrGetBoard({ name: 'Test' });
const rect = penpot.createRectangle();
rect.fills = [{ fillColor: '#FFFFFF', fillOpacity: 1 }];
board.appendChild(rect);

validateFills(rect, 'Test Rectangle');
```

### strokes 格式验证

```javascript
// === strokes 格式验证 ===

function validateStrokes(element, elementName) {
  if (!element.strokes || element.strokes.length === 0) {
    console.log(`ℹ️ ${elementName}: 无 strokes`);
    return true;
  }
  
  let isValid = true;
  element.strokes.forEach((stroke, i) => {
    if (!stroke.strokeOpacity && stroke.strokeType !== 'none') {
      console.log(`❌ ${elementName}: stroke[${i}] 缺少 strokeOpacity`);
      isValid = false;
    }
    
    if (!stroke.strokeWidth && stroke.strokeType !== 'none') {
      console.log(`⚠️ ${elementName}: stroke[${i}] 缺少 strokeWidth`);
    }
  });
  
  if (isValid) {
    console.log(`✅ ${elementName}: strokes 格式正确`);
  }
  
  return isValid;
}

// 使用示例
const manager = new BoardManager();
const board = manager.createOrGetBoard({ name: 'Test' });
const rect = penpot.createRectangle();
rect.strokes = [{
  strokeColor: '#FFFFFF',
  strokeOpacity: 1,
  strokeWidth: 2,
  strokeStyle: 'solid',
  strokeAlignment: 'outer'
}];
board.appendChild(rect);

validateStrokes(rect, 'Test Rectangle');
```

## 快速诊断

```javascript
// === 快速诊断 ===

// 1. 诊断 Board 配置
function diagnoseBoardConfig(manager) {
  const status = manager.getBoardStatus();
  console.log('🔍 Board 配置诊断:');
  
  status.forEach((b, i) => {
    console.log(`  ${i+1}. ${b.name}:`);
    console.log(`     位置: (${b.x.toFixed(0)}, ${b.y.toFixed(0)})`);
    console.log(`     尺寸: ${b.width}x${b.height}`);
    console.log(`     元素: ${b.childrenCount} 个`);
    console.log(`     有效性: ${b.isValid ? '✅' : '❌'}`);
    console.log(`     文本: ${b.hasText ? '有' : '无'}, 形状: ${b.hasShapes ? '有' : '无'}, 背景: ${b.hasBackground ? '有' : '无'}`);
  });
  
  return status;
}

// 2. 诊断 Storage 状态
function diagnoseStorageStatus(manager) {
  const validation = manager.validateStorage();
  const storageConfig = manager.getStorageConfig();
  
  console.log('🔍 Storage 状态诊断:');
  console.log(`  Storage 配置数: ${validation.storageCount}`);
  console.log(`  实际 Board 数: ${validation.actualCount}`);
  console.log(`  一致性: ${validation.isConsistent ? '✅' : '❌'}`);
  
  if (!validation.isConsistent) {
    console.log(`  缺失 Storage: ${validation.missingInStorage.join(', ')}`);
    console.log(`  缺失实际: ${validation.missingInActual.join(', ')}`);
  }
  
  console.log('\n💾 Storage 配置详情:');
  Object.entries(storageConfig).forEach(([name, config]) => {
    console.log(`  ${name}:`);
    console.log(`    尺寸: ${config.width}x${config.height}`);
    console.log(`    位置: (${config.x.toFixed(0)}, ${config.y.toFixed(0)})`);
  });
  
  return { validation, storageConfig };
}

// 使用诊断
const manager = new BoardManager();
diagnoseBoardConfig(manager);
console.log('\n');
diagnoseStorageStatus(manager);
```

## 最佳实践

```javascript
// === Penpot MCP 最佳实践 ===

// 1. 每个创建 Board 后立即禁用裁剪
const board = penpot.createBoard();
board.clipContent = false;  // ✅ 第一件事！

// 2. 使用正确的 fills 格式
rect.fills = [{ 
  fillColor: '#FFFFFF', 
  fillOpacity: 1  // ✅ 必须包含
}];

// 3. 使用 strokes 时指定完整属性
rect.strokes = [{
  strokeColor: '#FFFFFF',
  strokeOpacity: 1,
  strokeWidth: 2,
  strokeStyle: 'solid',
  strokeAlignment: 'outer'
}];

// 4. 使用 penpotUtils 设置相对位置
penpotUtils.setParentXY(child, 100, 50);  // ✅ 正确
// child.parentX = 100;  // ❌ parentX 是只读的

// 5. 定期运行 MCP 验证
const manager = new BoardManager();
const mcpValidation = runMCPValidation(manager);
if (!mcpValidation.overallValid) {
  console.log('⚠️ 发现问题，请修复');
}

// 6. 使用 penpotUtils 检查结构
const structure = penpotUtils.shapeStructure(board, 3);
console.log(JSON.stringify(structure, null, 2));
```
---
name: penpot-design
description: |
  Use Penpot MCP for UI/UX design. This is suitable for scenarios where UI designs are created, modified, or exported using the Penpot MCP Plugin.
  
  Trigger Conditions:
  - The user requests to create a UI design (web, mobile, dashboard, form, card, etc.)
  - The user requests to use Penpot for design
  - The user needs to generate interface designs or UI components
  - The user needs to design systems, layouts, or visual interfaces
  
  Core Capabilities:
  - Create elements such as Boards, Text, Rectangles, and Ellipses
  - Use Flex Layout and Grid Layout for layout
  - Apply Design Tokens and color styles
  - Upload images and create SVG icons
  - Use component libraries (Phosphor Icons, Wireframing Kit, etc.)
  - Add interactions and prototype links
---

# Penpot MCP Design Skill

Create UI designs in Penpot design tool via Penpot MCP Server.

**Prerequisite**: User must connect a Penpot design project to the MCP server via the Penpot MCP Plugin.

---

## Golden Rules (Remember These 5)

| Rule | Description | Consequence of Violation |
|------|-------------|-------------------------|
| **Single Code Block** | All code in one `execute_code` call | Variables lost, need storage to pass |
| **Disable Clipping** | Set `clipContent = false` immediately after creating each Board | Child elements clipped, display incomplete |
| **Flex First** | Use Flex instead of manual coordinates | Manual y-coordinate calculation error-prone |
| **Real-time Validation** | `console.log` after adding each key element | Hard to locate errors when they occur |
| **Semantic Naming** | Use `xxxSection` for Boards, `xxxList` for lists | Variable redeclaration errors |

---

## Complete Workflow

### Phase 1: Environment Setup

```javascript
// === Environment Setup ===

// 1. Clean up old elements
[...penpot.root.children].forEach(c => c.remove());
console.log('✅ Environment cleaned');

// 2. Create new page
const page = penpot.createPage();
page.name = 'ProjectName-PageName';
penpot.openPage(page);
console.log('✅ Page created:', page.name);

// 3. Create main canvas
const mainBoard = penpot.createBoard();
mainBoard.name = 'MainBoard';  // ✅ Semantic naming
mainBoard.resize(1440, 900);
mainBoard.fills = [{ fillColor: '#0F172A', fillOpacity: 1 }];
mainBoard.clipContent = false;  // ⚠️ Must disable clipping
penpot.root.appendChild(mainBoard);
console.log('✅ Main canvas created');
```

### Phase 2: Resource Preparation

```javascript
// === Resource Preparation ===

// 1. Check connected libraries
console.log('📚 Checking libraries...');
const connectedLibs = penpot.library.connected;
console.log(`  ${connectedLibs.length} libraries connected`);

// 2. Find common libraries
const phosphorLib = connectedLibs.find(lib => lib.name.includes('Phosphor'));
const wireframeLib = connectedLibs.find(lib => lib.name.includes('Wireframing'));

if (phosphorLib) {
  console.log(`  ✅ Phosphor Icons: ${phosphorLib.components.length} components`);
} else {
  console.log('  ⚠️ Phosphor Icons not connected');
}

if (wireframeLib) {
  console.log(`  ✅ Wireframing Kit: ${wireframeLib.components.length} components`);
} else {
  console.log('  ⚠️ Wireframing Kit not connected');
}

// 3. Preload common icons (performance optimization)
const requiredIcons = ['home-bold', 'search-bold', 'heart-bold', 'user-bold', 'settings-bold'];
const availableIcons = [];

if (phosphorLib) {
  requiredIcons.forEach(iconName => {
    const icon = phosphorLib.components.find(c => c.name === iconName);
    if (icon) {
      availableIcons.push(icon);
      console.log(`  ✅ ${iconName} available`);
    } else {
      console.log(`  ❌ ${iconName} not available`);
    }
  });
}

// Store in storage for later use
storage.availableIcons = availableIcons;
storage.phosphorLib = phosphorLib;

// 4. Upload image resources (example)
const imageUrls = {
  heroBg: 'https://picsum.photos/seed/hero/1200/600',
  feature1: 'https://picsum.photos/seed/f1/400/300',
  avatar1: 'https://i.pravatar.cc/150?img=1'
};

const uploadedImages = {};
for (const [key, url] of Object.entries(imageUrls)) {
  try {
    uploadedImages[key] = await penpot.uploadMediaUrl(key, url);
    console.log(`  ✅ ${key} uploaded successfully`);
  } catch (e) {
    console.log(`  ❌ ${key} upload failed: ${e.message}`);
  }
}

storage.uploadedImages = uploadedImages;

// 5. Initialize color system
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
console.log('✅ Resource preparation complete');
```

### Phase 3: Structure Building

```javascript
// === Structure Building ===

const mainBoard = [...penpot.root.children].find(b => b.name === 'MainBoard');
const colors = storage.colors;
const availableIcons = storage.availableIcons || [];

// Helper function
function add(parent, child, x, y) {
  parent.appendChild(child);
  penpotUtils.setParentXY(child, x, y);
  return child;
}

// Using Flex Layout
const flex = mainBoard.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 40;
flex.rowGap = 20;

// 1. Header Section
const headerSection = penpot.createBoard();
headerSection.name = 'HeaderSection';  // ✅ Semantic naming
headerSection.resize(1320, 70);
headerSection.fills = [{ fillColor: colors.bgSecondary, fillOpacity: 1 }];
mainBoard.appendChild(headerSection);
console.log('✅ HeaderSection added');

// 2. Hero Section
const heroSection = penpot.createBoard();
heroSection.name = 'HeroSection';  // ✅ Semantic naming
heroSection.resize(1320, 400);
heroSection.clipContent = false;
mainBoard.appendChild(heroSection);
console.log('✅ HeroSection added');

// 3. Using icon components
if (availableIcons.length > 0) {
  const iconInstance = availableIcons[0].instance();
  iconInstance.name = 'HeroIcon';  // ✅ Semantic naming
  heroSection.appendChild(iconInstance);
  penpotUtils.setParentXY(iconInstance, 100, 150);
  iconInstance.fills = [{ fillColor: colors.primary, fillOpacity: 1 }];
  console.log('✅ Icon added');
}

console.log('✅ Structure building complete');
```

### Phase 4: Validation

```javascript
// === Validation ===

// 1. Naming convention check
console.log('\n📋 Naming convention check:');
mainBoard.children.forEach((child, i) => {
  const hasSuffix = child.name.match(/(Section|Container|List|Grid|Card|Icon)/);
  const isPascalCase = child.name[0] === child.name[0].toUpperCase();
  
  const namingOk = hasSuffix && isPascalCase;
  console.log(`  ${i+1}. ${child.name} ${namingOk ? '✅' : '⚠️'}`);
  
  if (!namingOk) {
    console.log(`      Suggestion: Add suffix or use PascalCase`);
  }
});

// 2. Hierarchy structure check
console.log('\n🏗️  Hierarchy structure check:');
const structure = penpotUtils.shapeStructure(mainBoard, 3);
console.log(JSON.stringify(structure, null, 2));

// 3. clipContent check
console.log('\n✂️  clipContent check:');
mainBoard.children.forEach((child, i) => {
  const status = child.clipContent ? '❌ Needs disable' : '✅ Correct';
  console.log(`  ${i+1}. ${child.name}: ${status}`);
  
  if (child.clipContent) {
    child.clipContent = false;
    console.log(`      Auto-fixed`);
  }
});

// 4. Resource check
console.log('\n📚 Resource check:');
console.log(`  Color system: ${storage.colors ? '✅ Initialized' : '❌ Not initialized'}`);
console.log(`  Image resources: ${Object.keys(storage.uploadedImages || {}).length}`);
console.log(`  Available icons: ${storage.availableIcons?.length || 0}`);

// 5. Final report
console.log('\n📊 Final report:');
console.log(`  Total layers: ${mainBoard.children.length}`);
console.log(`  Page height: ${mainBoard.height}px`);
console.log(`  Validation status: ✅ Passed`);

console.log('\n✅ Validation complete');
```

---

## Core Concepts

### Coordinate System

```javascript
// Complete coordinate system example
const parent = penpot.createBoard();
parent.resize(800, 600);

const child = penpot.createBoard();
child.resize(200, 150);

// ❌ Wrong: parentX is read-only
child.parentX = 100;  // TypeError!

// ✅ Correct: Use utility function
penpotUtils.setParentXY(child, 100, 50);

// ✅ Correct: Set absolute coordinates (relative to canvas)
child.x = 100;
child.y = 50;

console.log(`Relative position: (${child.parentX}, ${child.parentY})`);
console.log(`Absolute position: (${child.x}, ${child.y})`);
```

### Board Fills Best Practices

```javascript
// Board fills usage scenarios comparison

// Scenario 1: Solid color background (recommended to set fills directly)
const simpleSection = penpot.createBoard();
simpleSection.name = 'SimpleSection';
simpleSection.resize(800, 400);
simpleSection.fills = [{ fillColor: '#1E293B', fillOpacity: 1 }];

// Scenario 2: Gradient background (recommended to use Rectangle child)
const gradientSection = penpot.createBoard();
gradientSection.name = 'GradientSection';
gradientSection.resize(800, 400);

const bgRect = penpot.createRectangle();
bgRect.resize(800, 400);
bgRect.fills = [{
  fillColor: '#1E293B',
  fillOpacity: 1,
  // Can add gradient and other effects here
}];
gradientSection.appendChild(bgRect);

// Scenario 3: Image background (must use Rectangle child)
const imageSection = penpot.createBoard();
imageSection.name = 'ImageSection';
imageSection.resize(800, 400);

const imgRect = penpot.createRectangle();
imgRect.resize(800, 400);
const imageData = await penpot.uploadMediaUrl('bg', 'https://...');
imgRect.fills = [{ fillImage: imageData, fillOpacity: 1 }];
imageSection.appendChild(imgRect);
```

### clipContent Clipping

```javascript
// Board defaults clipContent = true, which clips content beyond boundaries
// ⚠️ Disable immediately after creating any Board
const board = penpot.createBoard();
board.clipContent = false;  // First thing!
```

### fills Setting

```javascript
// ✅ Correct format: must include fillOpacity
rect.fills = [{ fillColor: '#FF0000', fillOpacity: 1 }];

// ❌ Common mistake: missing fillOpacity
rect.fills = [{ fillColor: '#FF0000' }];  // Image will show blank
```

### Z-Order Layer Order

- `appendChild()` → Adds to end = visual top layer
- `insertChild(index, shape)` → Control position
- Child elements stack bottom to top in order of addition

---

## Element Naming Convention

```javascript
// 📋 Naming convention code examples

// ✅ 1. Section naming (Section suffix)
const heroSection = penpot.createBoard();
heroSection.name = 'HeroSection';        // Hero section
const gallerySection = penpot.createBoard();
gallerySection.name = 'GallerySection';   // Gallery section
const footerSection = penpot.createBoard();
footerSection.name = 'FooterSection';     // Footer section

// ✅ 2. List naming (List/Grid suffix)
const galleryGrid = penpot.createBoard();
galleryGrid.name = 'GalleryGrid';         // Grid layout
const cardList = penpot.createBoard();
cardList.name = 'CardList';               // Card list
const artistList = penpot.createBoard();
artistList.name = 'ArtistList';           // Artist list

// ✅ 3. Card naming (Card + number/type)
const card1 = penpot.createBoard();
card1.name = 'ArtCard_1';                 // With number
const card2 = penpot.createBoard();
card2.name = 'FeatureCard_Hero';         // With type

// ✅ 4. Container naming (Container suffix)
const contentContainer = penpot.createBoard();
contentContainer.name = 'ContentContainer';
const navContainer = penpot.createBoard();
navContainer.name = 'NavContainer';

// ✅ 5. Decorative elements (Decor/Overlay suffix)
const bgDecor = penpot.createRectangle();
bgDecor.name = 'BackgroundDecor';        // Background decoration
const overlay = penpot.createBoard();
overlay.name = 'DarkOverlay';            // Overlay layer

// ❌ Names to avoid
const board1 = penpot.createBoard();
board1.name = 'Board1';                   // ❌ Not semantic enough
const container = penpot.createBoard();
container.name = 'container';             // ❌ Lowercase start
const a = penpot.createBoard();
a.name = 'a';                            // ❌ Meaningless naming
```

---

## Layout Solutions Quick Reference

### Three Options Decision Table

| Solution | Use Case | Code Volume | Maintenance Cost | Recommendation |
|----------|----------|-------------|------------------|----------------|
| **Flex Layout** | Multiple child elements auto-arranging | Low | Low | ⭐⭐⭐ |
| **currentY** | Simple vertical stacking | Medium | Medium | ⭐⭐ |
| **Manual Coordinates** | Special positioning needs | High | High | ⭐ |

### Solution 1: Flex Layout (Recommended)

```javascript
const page = penpot.createBoard();
page.resize(1440, 900);
page.clipContent = false;
penpot.root.appendChild(page);

const flex = page.addFlexLayout();
flex.dir = 'column';           // Vertical arrangement
flex.horizontalPadding = 60;   // Left/right padding
flex.verticalPadding = 40;     // Top/bottom padding
flex.rowGap = 20;              // Row gap

// Child elements auto-arrange, no need to set y coordinates
[header, hero, gallery, footer].forEach(el => page.appendChild(el));
```

**Flex Common Properties Quick Reference**:

| Property | Description | Common Values |
|----------|-------------|----------------|
| `dir` | Arrangement direction | `'column'` / `'row'` |
| `justifyContent` | Main axis alignment | `'center'` / `'space-between'` |
| `alignItems` | Cross axis alignment | `'center'` / `'stretch'` |
| `rowGap` / `columnGap` | Gaps | `20` |
| `horizontalPadding` | Horizontal padding | `60` |

### Solution 2: currentY Accumulated Coordinates

```javascript
let currentY = 0;  // Start from 0

const nav = penpot.createBoard();
nav.resize(1440, 80);
nav.y = currentY;
page.appendChild(nav);
currentY += 80;  // Accumulate height

const hero = penpot.createBoard();
hero.resize(1440, 500);
hero.y = currentY;  // Automatically 80
page.appendChild(hero);
currentY += 500;

console.log(`Total height: ${currentY}px`);  // 580
```

### Solution 3: Manual Coordinates (Not Recommended)

```javascript
// ❌ Prone to calculation errors, difficult maintenance
nav.y = 0;
hero.y = 80;      // Need precise calculation
gallery.y = 580;  // 80 + 500, error-prone!
footer.y = 980;   // 580 + 400, more calculations...
```

---

## penpotUtils Tools Quick Reference

| Tool | Purpose | Example |
|------|---------|---------|
| `setParentXY(shape, x, y)` | Set relative position | `setParentXY(child, 100, 50)` |
| `addFlexLayout(board, dir)` | Add Flex (maintains order) | `addFlexLayout(board, 'column')` |
| `findShape(predicate)` | Find single element | `findShape(s => s.name === 'Nav')` |
| `findShapes(predicate)` | Find multiple elements | `findShapes(s => s.name?.includes('Card'))` |
| `shapeStructure(shape, depth)` | View structure tree | `shapeStructure(root, 3)` |

### setParentXY Example

```javascript
// ❌ parentX/parentY are read-only properties
shape.parentX = 100;  // TypeError!

// ✅ Use utility function
penpotUtils.setParentXY(shape, 100, 50);
```

### findShape / findShapes Example

```javascript
// Exact search
const nav = penpotUtils.findShape(s => s.name === 'Navigation');

// Fuzzy search
const allCards = penpotUtils.findShapes(s => s.name?.includes('Card'));

// Search by type
const allTexts = penpotUtils.findShapes(s => s.type === 'text');
```

---

## Minimal Runnable Template

```javascript
// === 80-line core template (includes resource preparation and naming convention) ===

// 1. Clean environment
[...penpot.root.children].forEach(x => x.remove());

// 2. Create page container
const page = penpot.createBoard();
page.name = 'LandingPage';  // ✅ Semantic naming
page.resize(1440, 900);
page.clipContent = false;  // ⚠️ Must disable clipping
penpot.root.appendChild(page);

// 3. Resource preparation: Check component libraries
const phosphorLib = penpot.library.connected.find(lib => 
  lib.name.includes('Phosphor')
);
console.log('📚 Phosphor Icons library:', phosphorLib ? 'Connected' : 'Not found');

// Preload common icons (optional)
const iconNames = ['home-bold', 'heart-bold', 'user-bold'];
iconNames.forEach(name => {
  const icon = phosphorLib?.components.find(c => c.name === name);
  if (icon) console.log(`  ✅ ${name} available`);
});

// 4. Add Flex layout
const flex = page.addFlexLayout();
flex.dir = 'column';
flex.horizontalPadding = 60;
flex.verticalPadding = 40;
flex.rowGap = 20;

// 5. Add Header Section
const headerSection = penpot.createBoard();  // ✅ Semantic naming
headerSection.name = 'HeaderSection';
headerSection.resize(1320, 70);
headerSection.fills = [{ fillColor: '#0A0A0F', fillOpacity: 1 }];
page.appendChild(headerSection);
console.log('✅ HeaderSection added');

// Header internal layout
const headerFlex = headerSection.addFlexLayout();
headerFlex.dir = 'row';
headerFlex.justifyContent = 'space-between';

const logo = penpot.createText('BRAND');
logo.name = 'LogoText';  // ✅ Semantic naming
logo.fontSize = 22;
logo.fontWeight = '800';
logo.fills = [{ fillColor: '#FFFFFF' }];
headerSection.appendChild(logo);

// 6. Add Hero Section
const heroSection = penpot.createBoard();  // ✅ Semantic naming
heroSection.name = 'HeroSection';
heroSection.resize(1320, 400);
heroSection.clipContent = false;
page.appendChild(heroSection);
console.log('✅ HeroSection added');

const title = penpot.createText('HELLO WORLD');
title.name = 'HeroTitle';  // ✅ Semantic naming
title.fontSize = 48;
title.fontWeight = '800';
title.fills = [{ fillColor: '#FFFFFF' }];
heroSection.appendChild(title);

// 7. Add icon example (using component library)
if (phosphorLib) {
  const homeIconComp = phosphorLib.components.find(c => c.name === 'home-bold');
  if (homeIconComp) {
    const homeIcon = homeIconComp.instance();
    homeIcon.name = 'HomeIcon';  // ✅ Semantic naming
    heroSection.appendChild(homeIcon);
    penpotUtils.setParentXY(homeIcon, 200, 150);
    homeIcon.fills = [{ fillColor: '#8B5CF6', fillOpacity: 1 }];
    console.log('✅ HomeIcon added');
  }
}

// 8. Validation: Check naming convention
console.log('\n📋 Naming check:');
page.children.forEach((child, i) => {
  const hasSuffix = child.name.includes('Section') || 
                      child.name.includes('Container') ||
                      child.name.includes('List') ||
                      child.name.includes('Grid');
  console.log(`  ${i+1}. ${child.name} ${hasSuffix ? '✅' : '⚠️'}`);
});

// 9. Final validation
console.log('\n✅ Complete:', page.children.length, 'layers');
console.log('Hierarchy:', page.children.map(c => c.name).join(' → '));
```

---

## Complete Example: Landing Page Using Component Library

```javascript
// === Complete example (120 lines, includes component library and naming convention) ===

// Phase 1: Environment Setup
[...penpot.root.children].forEach(c => c.remove());

const page = penpot.createBoard();
page.name = 'LandingPage';
page.resize(1440, 1000);
page.clipContent = false;
penpot.root.appendChild(page);

// Phase 2: Resource Preparation
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

// Phase 3: Structure Building
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

// Add icons
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

// Phase 4: Validation
console.log('📋 Naming check:');
page.children.forEach((child, i) => {
  const hasSuffix = child.name.includes('Section');
  console.log(`  ${i+1}. ${child.name} ${hasSuffix ? '✅' : '⚠️'}`);
});

console.log('\n✅ Complete:', page.children.length, 'layers');
console.log('Hierarchy:', page.children.map(c => c.name).join(' → '));
console.log('Available icons:', availableIcons.length);
```

---

## Common Errors Quick Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `Identifier 'xxx' has already been declared` | Variable naming conflict | Use semantic naming: `xxxSection`, `xxxList` |
| Variables lost across code blocks | Independent call contexts | Use single code block strategy or storage |
| Export fails `http error` | Page size/element count too large | Use validation script instead of export |
| Child elements clipped/occluded | Parent Board has clipContent=true | Set `clipContent = false` immediately after creating Board |
| Child element position exceeds parent container | Coordinate calculation error | Use Flex Layout or currentY |
| `Cannot add property fill` | Setting fills immediately after Board creation | Use Rectangle instead |
| Root has multiple elements | Elements not properly added to parent | Use verify() to check hierarchy |
| Colors not working | Lowercase color values | Use uppercase `#FFFFFF` |
| Image blank | Missing fillOpacity | Add `fillOpacity: 1` |
| Naming not standardized | Semantic naming not used | Use suffixes like `xxxSection`, `xxxList` |

---

## API Reference

When encountering unfamiliar APIs:

```javascript
// View Board complete API
penpot_penpot_api_info({ type: 'Board' })

// View FlexLayout properties
penpot_penpot_api_info({ type: 'FlexLayout' })

// View GridLayout properties
penpot_penpot_api_info({ type: 'GridLayout' })
```

---

## Detailed Documentation

See Penpot MCP documentation for complete API reference.

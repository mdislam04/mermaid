# Portfolio Tracker Design System Reference

This document details the design system for the portfolio tracker application to ensure visual consistency when making modifications.

## Aesthetic Direction

**Financial Terminal Theme** - Inspired by Bloomberg terminals and professional trading platforms

### Core Principles
1. **Data-First Design** - Information hierarchy prioritizes numerical data
2. **High Contrast** - Dark backgrounds with bright accents for readability
3. **Monospace Typography** - Consistent character width for numbers alignment
4. **Minimal Ornamentation** - Focus on function over decoration
5. **Professional Polish** - Refined details and smooth interactions

## Color System

### Base Colors
```css
/* Primary Backgrounds */
--bg-primary: #0a0e1a;      /* Deep navy - main page background */
--bg-secondary: #151b2e;     /* Darker blue - cards and containers */
--bg-tertiary: #1e2840;      /* Medium blue - inputs and hover states */

/* Accent Colors */
--accent-cyan: #00f5ff;      /* Electric cyan - primary actions, highlights */
--accent-green: #00ff88;     /* Bright green - positive values, success */
--accent-red: #ff3366;       /* Vibrant red - negative values, danger */

/* Text Colors */
--text-primary: #e0e6f0;     /* Light blue-grey - main text */
--text-secondary: #8b96b0;   /* Muted blue-grey - secondary text, labels */

/* Borders and Dividers */
--border: #2a3752;           /* Subtle blue-grey - borders, dividers */
--shadow: rgba(0, 245, 255, 0.1);  /* Cyan glow - shadows on hover */
```

### Color Usage Rules

**DO:**
- Use `--accent-cyan` for primary buttons, links, and interactive elements
- Use `--accent-green` exclusively for positive numbers and success states
- Use `--accent-red` exclusively for negative numbers and danger actions
- Use `--text-secondary` for labels, hints, and less important text
- Apply subtle cyan shadows (`--shadow`) on hover for depth

**DON'T:**
- Mix warm colors (orange, yellow) with the cool palette
- Use pure white (#ffffff) - always use `--text-primary` instead
- Use pure black (#000000) - always use `--bg-primary` or darker
- Apply accent colors to large backgrounds (keep them as highlights)

## Typography

### Font Families

```css
/* Primary - JetBrains Mono */
font-family: 'JetBrains Mono', monospace;

/* Alternative - Space Mono */
font-family: 'Space Mono', monospace;
```

**JetBrains Mono Usage:**
- All body text
- Table content
- Form inputs
- Numbers and data
- Code-like elements

**Space Mono Usage:**
- Alternative for variety if needed
- Display headings (optional)

### Type Scale

```css
/* Display/Heading */
font-size: clamp(24px, 4vw, 36px);
font-weight: 700;
letter-spacing: -0.5px;

/* Body Large */
font-size: 14px;
font-weight: 600;

/* Body Regular */
font-size: 13px;
font-weight: 400;

/* Body Small */
font-size: 11px;
font-weight: 400;
letter-spacing: 0.5px;

/* Micro Text (labels) */
font-size: 10px;
font-weight: 400;
text-transform: uppercase;
letter-spacing: 1px;
```

### Typography Rules

**DO:**
- Use uppercase + letter-spacing for labels and section headers
- Apply negative letter-spacing (-0.5px) to large headings
- Keep line-height around 1.5 for readability
- Use font-weight 700 for emphasis, 600 for semi-bold, 400 for regular

**DON'T:**
- Use font-sizes smaller than 10px
- Mix serif or sans-serif fonts with monospace
- Use italic (doesn't fit terminal aesthetic)
- Over-use bold - reserve for important data

## Spacing System

### Base Unit: 4px

```css
/* Scale */
xs:  4px   (0.25rem)
sm:  8px   (0.5rem)
md:  12px  (0.75rem)
base: 16px  (1rem)
lg:  20px  (1.25rem)
xl:  24px  (1.5rem)
2xl: 32px  (2rem)
3xl: 48px  (3rem)
```

### Usage Guidelines

```css
/* Padding for cells */
padding: 16px;  /* base unit */

/* Gap between elements */
gap: 15px;      /* Close to base */

/* Margins between sections */
margin-bottom: 30px;  /* ~2x base */

/* Container padding */
padding: 20px;  /* lg unit */
```

## Layout Patterns

### Container Widths

```css
/* Main container */
max-width: 1600px;
margin: 0 auto;
padding: 20px;

/* Card/Panel */
border-radius: 8px;
padding: 20px;
background: var(--bg-secondary);
```

### Grid System

```css
/* Flex-based layouts */
display: flex;
gap: 15px;
flex-wrap: wrap;

/* Table remains traditional table layout */
/* Don't convert to CSS grid - keep semantic HTML */
```

## Component Patterns

### Buttons

```css
/* Base Button */
.btn {
    background: var(--bg-secondary);
    border: 1px solid var(--border);
    padding: 10px 20px;
    border-radius: 6px;
    font-size: 13px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    transition: all 0.2s;
}

.btn:hover {
    background: var(--bg-tertiary);
    border-color: var(--accent-cyan);
    transform: translateY(-1px);
    box-shadow: 0 4px 12px var(--shadow);
}

/* Primary Button */
.btn-primary {
    background: var(--accent-cyan);
    color: var(--bg-primary);
    border-color: var(--accent-cyan);
}

/* Danger Button */
.btn-danger {
    background: var(--accent-red);
    border-color: var(--accent-red);
    color: white;
}
```

### Form Inputs

```css
input, select, textarea {
    background: var(--bg-tertiary);
    color: var(--text-primary);
    border: 1px solid var(--border);
    padding: 8px 12px;
    border-radius: 4px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 14px;
    transition: all 0.2s;
}

input:focus, select:focus, textarea:focus {
    outline: none;
    border-color: var(--accent-cyan);
    box-shadow: 0 0 0 2px rgba(0, 245, 255, 0.1);
}
```

### Table Cells

```css
/* Standard cell */
td {
    padding: 16px;
    border-bottom: 1px solid var(--border);
    font-size: 13px;
}

/* Number cell */
.positive {
    color: var(--accent-green);
    font-weight: 600;
}

.negative {
    color: var(--accent-red);
    font-weight: 600;
}

/* Header cell */
th {
    background: var(--bg-tertiary);
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 1px;
    font-size: 11px;
    color: var(--text-secondary);
    position: sticky;
    top: 0;
}
```

## Animation & Motion

### Timing Functions

```css
/* Default transition */
transition: all 0.2s ease;

/* Hover effects */
transition: all 0.2s ease-out;

/* Entrances */
animation: fadeIn 0.6s ease-out;
```

### Standard Animations

```css
/* Fade In (for content) */
@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Slide Down (for header) */
@keyframes slideDown {
    from {
        opacity: 0;
        transform: translateY(-20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Modal entrance */
@keyframes modalSlideIn {
    from {
        opacity: 0;
        transform: translateY(-50px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}
```

### Motion Rules

**DO:**
- Use subtle transforms (1-2px) on hover
- Apply staggered delays for sequential animations
- Keep durations between 0.2s - 0.6s
- Use ease-out for entrances, ease-in for exits

**DON'T:**
- Animate multiple properties separately (use `all`)
- Create bouncy or spring animations (keep it smooth)
- Apply transitions to color changes (instant is fine)
- Over-animate - motion should enhance, not distract

## Responsive Breakpoints

```css
/* Mobile */
@media (max-width: 768px) {
    /* Stack controls */
    .header {
        flex-direction: column;
        gap: 15px;
    }
    
    /* Reduce padding */
    th, td {
        padding: 12px 8px;
        font-size: 11px;
    }
    
    /* Full-width elements */
    .month-selector {
        width: 100%;
    }
}
```

## Icons & Symbols

### Custom Symbols

```css
/* Triangle accent */
◢  /* Used in headers: "◢ PORTFOLIO TRACKER" */

/* Drag handle */
⋮⋮  /* Used for row reordering */
```

### Usage
- Use sparingly - 1-2 decorative symbols maximum
- Keep them monochrome (match text color)
- Position for visual balance, not meaning

## Accessibility

### Contrast Ratios
- Text on bg-primary: 14:1 (WCAG AAA)
- Text on bg-secondary: 12:1 (WCAG AAA)
- Accent colors: 7:1+ (WCAG AA)

### Focus States
- Always show focus outlines
- Use cyan border + glow for consistency
- Never use `outline: none` without replacement

### Semantic HTML
- Use proper `<table>`, `<th>`, `<td>` tags
- Label all inputs
- Maintain heading hierarchy

## Design Don'ts

**Avoid these common pitfalls:**

1. **No white backgrounds** - Keep the dark theme consistent
2. **No rounded corners >10px** - Stay sharp and technical
3. **No gradients on buttons** - Solid colors only
4. **No shadows except hover** - Flat design with depth via color
5. **No emoji** - Use symbols or text only
6. **No decorative animations** - Functional motion only
7. **No mixing color schemes** - Stick to the cyan/green/red palette

## Extension Guidelines

When adding new features, maintain consistency:

### New Interactive Elements
1. Use same button styles
2. Apply hover states with cyan border
3. Add 0.2s transitions
4. Follow spacing system

### New Data Displays
1. Use monospace font
2. Apply positive/negative colors
3. Maintain table cell padding
4. Keep borders subtle

### New Sections
1. Use bg-secondary for containers
2. Add 8px border-radius
3. Include section header with ◢ symbol
4. Maintain 30px spacing between sections

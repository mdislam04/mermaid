---
name: portfolio-tracker
description: Single-page responsive portfolio tracking application with financial terminal aesthetic. Use when users want to create, modify, or enhance a portfolio tracking app for day-wise trading/investment monitoring. Features include drag-and-drop row reordering, month-specific data persistence, cell editing with formatting, auto color-coding (green for positive, red for negative), calculated Standing AT row, local storage persistence, and notes section. Supports operations like add/edit/delete contracts, hide/show rows and columns, and tracks UnRealized P&L and Realized P&L separately by month.
---

# Portfolio Tracker Skill

This skill provides a complete single-page responsive portfolio tracking application with a professional financial terminal aesthetic.

## Architecture Overview

### Data Structure
The application uses a month-specific data architecture:

```javascript
// Global structure
let allMonthsData = {}; // Format: { "YYYY-M": { contracts, cells, dates, ... } }

// Each month contains:
{
  contracts: [],           // Array of contract names
  dates: [],              // Array of date strings (DD-MMM format)
  cells: {},              // Key-value pairs: "rowIndex-colIndex" or "unrealized-colIndex" or "realized-colIndex"
  hiddenRows: Set(),      // Indices of hidden contract rows
  hiddenCols: Set(),      // Indices of hidden date columns
  notes: ''               // Trading notes for the month
}

// Cell structure
cells[key] = {
  value: '',              // Cell content (text or number)
  bold: false,            // Bold formatting flag
  larger: false           // Larger font flag
}
```

### Key Features

1. **Month-Specific Persistence**
   - Each month stores independent data
   - Switching months auto-saves current and loads selected month
   - New months start blank with only P&L rows

2. **Drag-and-Drop Reordering**
   - Contracts can be reordered via drag handle (⋮⋮)
   - All cell data moves with the contract
   - Hidden rows and cell formatting preserved during reordering

3. **Cell Editing**
   - Click any cell to edit
   - Formatting toolbar appears for bold/larger font
   - Auto color-coding: positive = green, negative = red

4. **Calculated Rows**
   - UnRealized P&L row (editable)
   - Net Realised P&L row (editable)
   - Standing AT row (auto-calculated: unrealized + realized)

## Design System

### Color Palette
```css
--bg-primary: #0a0e1a        /* Dark navy background */
--bg-secondary: #151b2e      /* Card/container background */
--bg-tertiary: #1e2840       /* Input/hover background */
--accent-cyan: #00f5ff       /* Primary accent */
--accent-green: #00ff88      /* Positive values */
--accent-red: #ff3366        /* Negative values */
--text-primary: #e0e6f0      /* Main text */
--text-secondary: #8b96b0    /* Secondary text */
--border: #2a3752            /* Border color */
```

### Typography
- Primary font: `JetBrains Mono` (monospace)
- Display elements: Space Mono
- All text uses monospace for financial terminal feel

### Animations
- Page load: slideDown header, fadeIn content
- Row hover: opacity transitions for actions
- Drag: opacity changes and border highlights
- Formatting toolbar: slide-in from bottom right

## Implementation Guidelines

### When Modifying This Application

**Adding new features:**
1. Maintain month-specific data structure
2. Always call `saveCurrentMonthData()` after data changes
3. Use `renderTable()` to refresh the display
4. Follow the financial terminal aesthetic (dark theme, cyan/green accents)

**Data persistence rules:**
- Month data format: `"YYYY-M"` (e.g., "2026-0" for January 2026)
- Always convert Sets to Arrays before saving to localStorage
- Always convert Arrays back to Sets after loading from localStorage
- Auto-save interval: 5 seconds

**Cell key formats:**
- Contract cells: `"rowIndex-colIndex"` (e.g., "0-5")
- UnRealized P&L: `"unrealized-colIndex"`
- Realized P&L: `"realized-colIndex"`
- Contract names: `"contract-rowIndex"`

### Critical Functions

**Month Management:**
```javascript
loadMonthData(month, year)      // Load data for specific month
saveCurrentMonthData()          // Save current month to allMonthsData
getMonthKey(month, year)        // Generate month key "YYYY-M"
```

**Drag and Drop:**
```javascript
handleDragStart(e, rowIndex)    // Initialize drag
handleDragOver(e)               // Show drop target
handleDrop(e, targetIndex)      // Complete reorder
handleDragEnd()                 // Clean up drag state
```

**Cell Operations:**
```javascript
editCell(element, cellKey, currentValue)  // Start editing
saveCellEdit(input, cellKey)              // Save changes
formatCell(type)                          // Apply bold/larger formatting
```

**Row/Column Management:**
```javascript
addContract()                   // Open modal to add new contract
confirmAddContract()            // Save new contract
deleteRow(index)                // Remove contract and its data
hideRow(index) / hideColumn(index)  // Toggle visibility
```

### Responsive Behavior

**Desktop (>768px):**
- Full table with all columns visible
- Hover actions on rows and columns
- Formatting toolbar in bottom-right

**Mobile (≤768px):**
- Horizontal scroll for table
- Stacked header controls
- Full-width formatting toolbar at bottom
- Reduced padding on cells

### Common Customization Requests

**Adding new calculated rows:**
1. Add row in `renderTableBody()` after existing calculated rows
2. Create cell keys following pattern: `"rowtype-colIndex"`
3. Implement calculation logic (see `addStandingRow()` example)
4. Style appropriately (background, borders)

**Changing color scheme:**
1. Update CSS variables in `:root`
2. Maintain contrast ratios for readability
3. Update positive/negative colors if needed

**Adding column types:**
1. Extend `generateDatesForMonth()` or create separate column generation
2. Update `renderTableHead()` to handle new column types
3. Ensure cell keys remain unique

**Exporting data:**
1. Access `allMonthsData` object containing all months
2. Convert to desired format (CSV, JSON, Excel)
3. Use `present_files` tool to provide download

### Best Practices

1. **Always test drag-and-drop** after modifications to cell structure
2. **Preserve month isolation** - never leak data between months
3. **Maintain visual consistency** with the financial terminal aesthetic
4. **Test mobile responsiveness** for table scrolling and touch interactions
5. **Validate data persistence** by switching months and refreshing browser
6. **Keep Standing AT calculation** synchronized with P&L rows
7. **Use semantic HTML** for accessibility (th, td, proper table structure)

## File Structure

```
portfolio-tracker.html          (Single-file application)
├── HTML Structure
│   ├── Header (title + controls)
│   ├── Table container
│   ├── Notes section
│   └── Modals (add contract)
├── CSS Styles
│   ├── Dark theme variables
│   ├── Table styling
│   ├── Drag-and-drop states
│   ├── Responsive breakpoints
│   └── Animations
└── JavaScript
    ├── Data structures
    ├── Month management
    ├── Table rendering
    ├── Drag-and-drop handlers
    ├── Cell editing
    └── LocalStorage persistence
```

## Usage Examples

**Adding month-specific features:**
```javascript
// Add quarterly summary across months
function calculateQuarterlyTotal(quarter, year) {
    const months = [0, 1, 2]; // Q1
    let total = 0;
    months.forEach(month => {
        const key = getMonthKey(month, year);
        if (allMonthsData[key]) {
            // Process data
        }
    });
    return total;
}
```

**Custom cell validation:**
```javascript
// In saveCellEdit function, add:
function saveCellEdit(input, cellKey) {
    const value = input.value.trim();
    
    // Custom validation
    if (cellKey.startsWith('unrealized') || cellKey.startsWith('realized')) {
        const numValue = parseFloat(value.replace(/,/g, ''));
        if (isNaN(numValue) && value !== '') {
            alert('Please enter a valid number');
            return;
        }
    }
    
    // Continue with save...
}
```

## Integration Points

**Export to Excel:**
Use xlsx skill to convert allMonthsData to spreadsheet format

**Import from CSV:**
Parse CSV and populate portfolioData structure, then call renderTable()

**API Integration:**
Fetch live prices and auto-populate cells with market data

**Charts/Visualization:**
Use recharts library to create performance charts from monthly data

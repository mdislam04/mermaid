# Portfolio Tracker Implementation Reference

This document provides detailed implementation patterns and code examples for common modifications to the portfolio tracker application.

## Table of Contents
1. [Complete Application Code](#complete-application-code)
2. [Data Flow Diagrams](#data-flow-diagrams)
3. [Common Modification Patterns](#common-modification-patterns)
4. [Troubleshooting Guide](#troubleshooting-guide)

## Complete Application Code

The portfolio tracker is a single-file HTML application. The complete current implementation is stored in `/home/claude/portfolio-tracker.html` and can be viewed or copied as needed.

## Data Flow Diagrams

### Month Switching Flow
```
User selects new month
    ↓
saveCurrentMonthData()
    ↓
Update localStorage with current month data
    ↓
loadMonthData(newMonth, newYear)
    ↓
Check if allMonthsData[monthKey] exists
    ↓
    Yes → Load existing data
    No → Create blank portfolioData with empty contracts[]
    ↓
generateDatesForMonth()
    ↓
renderTable()
```

### Drag-and-Drop Reordering Flow
```
User drags contract row
    ↓
handleDragStart() - Store draggedRow index
    ↓
handleDragOver() - Show visual feedback
    ↓
handleDrop() - Reorder arrays and data
    ├── Reorder contracts[]
    ├── Update cell keys (rowIndex changes)
    └── Update hiddenRows indices
    ↓
saveCurrentMonthData()
    ↓
renderTable()
```

### Cell Editing Flow
```
User clicks cell
    ↓
editCell() - Replace content with input field
    ↓
Show formatting toolbar
    ↓
User types and presses Enter
    ↓
saveCellEdit()
    ├── Update contracts[] (if contract cell)
    └── Update cells[cellKey] (if data cell)
    ↓
Hide formatting toolbar
    ↓
saveCurrentMonthData()
    ↓
renderTable()
```

## Common Modification Patterns

### Pattern 1: Adding a New Calculated Row

To add a new calculated row (like "Total Profit" or "Average Return"):

```javascript
// 1. Add function similar to addCalculatedRow
function addTotalProfitRow() {
    const tbody = document.getElementById('table-body');
    const tr = document.createElement('tr');
    tr.className = 'total-profit-row';

    const tdLabel = document.createElement('td');
    tdLabel.className = 'contract-cell';
    tdLabel.style.background = 'rgba(0, 255, 136, 0.1)';
    tdLabel.textContent = 'Total Profit';
    tr.appendChild(tdLabel);

    portfolioData.dates.forEach((date, colIndex) => {
        if (portfolioData.hiddenCols.has(colIndex)) return;

        const td = document.createElement('td');
        const cellKey = `totalprofit-${colIndex}`;
        
        // Calculate from other cells
        let totalProfit = 0;
        portfolioData.contracts.forEach((contract, rowIdx) => {
            const cellData = portfolioData.cells[`${rowIdx}-${colIndex}`];
            if (cellData?.value) {
                const numValue = parseFloat(cellData.value.toString().replace(/,/g, ''));
                if (!isNaN(numValue)) {
                    totalProfit += numValue;
                }
            }
        });
        
        const cellDiv = document.createElement('div');
        cellDiv.textContent = totalProfit !== 0 ? totalProfit.toLocaleString() : '';
        cellDiv.style.fontWeight = '700';
        cellDiv.className = totalProfit >= 0 ? 'positive' : 'negative';
        
        td.appendChild(cellDiv);
        tr.appendChild(td);
    });

    tbody.appendChild(tr);
}

// 2. Call in renderTableBody() after other rows
function renderTableBody() {
    // ... existing code ...
    
    addCalculatedRow('UnRealized P & L', 'unrealized');
    addCalculatedRow('Net realised P&L (as of today)', 'realized');
    addTotalProfitRow();  // ADD THIS LINE
    addStandingRow();
}
```

### Pattern 2: Adding Export to CSV

```javascript
function exportToCSV() {
    const monthKey = getMonthKey(currentMonth, currentYear);
    let csv = 'Contract,' + portfolioData.dates.join(',') + '\n';
    
    // Contract rows
    portfolioData.contracts.forEach((contract, rowIdx) => {
        let row = [contract];
        portfolioData.dates.forEach((date, colIdx) => {
            const cellKey = `${rowIdx}-${colIdx}`;
            const value = portfolioData.cells[cellKey]?.value || '';
            row.push(value);
        });
        csv += row.join(',') + '\n';
    });
    
    // UnRealized row
    let unrealizedRow = ['UnRealized P&L'];
    portfolioData.dates.forEach((date, colIdx) => {
        const value = portfolioData.cells[`unrealized-${colIdx}`]?.value || '';
        unrealizedRow.push(value);
    });
    csv += unrealizedRow.join(',') + '\n';
    
    // Realized row
    let realizedRow = ['Net Realised P&L'];
    portfolioData.dates.forEach((date, colIdx) => {
        const value = portfolioData.cells[`realized-${colIdx}`]?.value || '';
        realizedRow.push(value);
    });
    csv += realizedRow.join(',') + '\n';
    
    // Download
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `portfolio-${monthKey}.csv`;
    a.click();
}

// Add button in HTML controls section
// <button class="btn" onclick="exportToCSV()">Export CSV</button>
```

### Pattern 3: Adding Column Totals

```javascript
// Add a total column at the end of each row
function renderTableBody() {
    const tbody = document.getElementById('table-body');
    tbody.innerHTML = '';

    portfolioData.contracts.forEach((contract, rowIndex) => {
        if (portfolioData.hiddenRows.has(rowIndex)) return;

        const tr = document.createElement('tr');
        // ... existing row code ...
        
        // Add total cell at the end
        const tdTotal = document.createElement('td');
        tdTotal.style.background = 'var(--bg-tertiary)';
        tdTotal.style.fontWeight = '700';
        
        let rowTotal = 0;
        portfolioData.dates.forEach((date, colIndex) => {
            if (portfolioData.hiddenCols.has(colIndex)) return;
            const cellKey = `${rowIndex}-${colIndex}`;
            const value = portfolioData.cells[cellKey]?.value;
            if (value) {
                const numValue = parseFloat(value.toString().replace(/,/g, ''));
                if (!isNaN(numValue)) rowTotal += numValue;
            }
        });
        
        const totalDiv = document.createElement('div');
        totalDiv.textContent = rowTotal !== 0 ? rowTotal.toLocaleString() : '';
        totalDiv.className = rowTotal >= 0 ? 'positive' : 'negative';
        tdTotal.appendChild(totalDiv);
        tr.appendChild(tdTotal);
        
        tbody.appendChild(tr);
    });
}

// Also add "Total" header in renderTableHead()
```

### Pattern 4: Auto-Calculate from Formula

```javascript
// Add formula support for cells
function evaluateFormula(formula, rowIndex, colIndex) {
    // Example: "=SUM(R1:R5)" sums rows 1-5 for this column
    if (!formula.startsWith('=')) return null;
    
    const formulaBody = formula.substring(1);
    
    // Handle SUM
    if (formulaBody.startsWith('SUM(')) {
        const range = formulaBody.match(/SUM\(R(\d+):R(\d+)\)/);
        if (range) {
            const startRow = parseInt(range[1]);
            const endRow = parseInt(range[2]);
            let sum = 0;
            
            for (let r = startRow; r <= endRow; r++) {
                const cellKey = `${r}-${colIndex}`;
                const value = portfolioData.cells[cellKey]?.value;
                if (value) {
                    const numValue = parseFloat(value.toString().replace(/,/g, ''));
                    if (!isNaN(numValue)) sum += numValue;
                }
            }
            return sum;
        }
    }
    
    return null;
}

// Modify renderTableBody to check for formulas
// In the cell rendering loop:
if (cellData.value?.toString().startsWith('=')) {
    const result = evaluateFormula(cellData.value, rowIndex, colIndex);
    cellDiv.textContent = result !== null ? result.toLocaleString() : cellData.value;
} else {
    cellDiv.textContent = cellData.value;
}
```

### Pattern 5: Adding Multi-Month Comparison

```javascript
function addMonthComparisonView() {
    // Create a comparison table showing Standing AT across months
    const comparisonDiv = document.createElement('div');
    comparisonDiv.className = 'comparison-section';
    comparisonDiv.innerHTML = '<h3>◢ Monthly Comparison</h3>';
    
    const table = document.createElement('table');
    table.className = 'comparison-table';
    
    // Header
    const thead = document.createElement('thead');
    const headerRow = document.createElement('tr');
    headerRow.innerHTML = '<th>Month</th><th>Standing AT</th><th>Change</th>';
    thead.appendChild(headerRow);
    table.appendChild(thead);
    
    // Body - iterate through months
    const tbody = document.createElement('tbody');
    const months = Object.keys(allMonthsData).sort();
    let previousTotal = null;
    
    months.forEach(monthKey => {
        const monthData = allMonthsData[monthKey];
        
        // Calculate total Standing AT for the month
        let monthTotal = 0;
        monthData.dates.forEach((date, colIdx) => {
            const unrealized = parseFloat(monthData.cells[`unrealized-${colIdx}`]?.value || 0);
            const realized = parseFloat(monthData.cells[`realized-${colIdx}`]?.value || 0);
            monthTotal += unrealized + realized;
        });
        
        const tr = document.createElement('tr');
        
        // Month cell
        const [year, month] = monthKey.split('-');
        const monthName = new Date(year, month, 1).toLocaleDateString('en-US', { 
            month: 'long', 
            year: 'numeric' 
        });
        tr.innerHTML = `<td>${monthName}</td>`;
        
        // Total cell
        const totalCell = document.createElement('td');
        totalCell.textContent = monthTotal.toLocaleString();
        totalCell.className = monthTotal >= 0 ? 'positive' : 'negative';
        tr.appendChild(totalCell);
        
        // Change cell
        const changeCell = document.createElement('td');
        if (previousTotal !== null) {
            const change = monthTotal - previousTotal;
            changeCell.textContent = (change >= 0 ? '+' : '') + change.toLocaleString();
            changeCell.className = change >= 0 ? 'positive' : 'negative';
        } else {
            changeCell.textContent = '-';
        }
        tr.appendChild(changeCell);
        
        tbody.appendChild(tr);
        previousTotal = monthTotal;
    });
    
    table.appendChild(tbody);
    comparisonDiv.appendChild(table);
    
    // Insert before notes section
    const notesSection = document.querySelector('.notes-section');
    notesSection.parentNode.insertBefore(comparisonDiv, notesSection);
}

// Call this in init() or add a button to toggle it
```

## Troubleshooting Guide

### Issue: Data Not Persisting After Month Switch

**Cause:** saveCurrentMonthData() not being called before switching
**Fix:** Ensure month selector change event calls saveCurrentMonthData() first

```javascript
select.addEventListener('change', (e) => {
    saveCurrentMonthData(); // MUST be first
    // ... rest of code
});
```

### Issue: Drag and Drop Breaking Cell Data

**Cause:** Cell keys not updating correctly during reorder
**Fix:** Verify the reindexing logic in handleDrop()

```javascript
// Debug by adding console.logs
function handleDrop(e, targetIndex) {
    console.log('Before reorder:', portfolioData.cells);
    // ... reordering logic ...
    console.log('After reorder:', newCells);
}
```

### Issue: Standing AT Not Calculating

**Cause:** UnRealized or Realized cell keys malformed
**Fix:** Ensure cell keys follow exact pattern

```javascript
// Correct format
const unrealizedKey = `unrealized-${colIndex}`;  // NOT unrealized-0-5
const realizedKey = `realized-${colIndex}`;      // NOT realized-0-5
```

### Issue: Mobile Table Not Scrolling

**Cause:** table-container overflow not set correctly
**Fix:** Ensure CSS has proper overflow

```css
.table-container {
    overflow-x: auto;  /* REQUIRED */
    -webkit-overflow-scrolling: touch;  /* Smooth on iOS */
}
```

### Issue: LocalStorage Quota Exceeded

**Cause:** Too much data stored
**Fix:** Implement data cleanup or compression

```javascript
function cleanupOldMonths() {
    const monthKeys = Object.keys(allMonthsData);
    const now = new Date();
    
    monthKeys.forEach(key => {
        const [year, month] = key.split('-').map(Number);
        const monthDate = new Date(year, month, 1);
        const monthsOld = (now - monthDate) / (1000 * 60 * 60 * 24 * 30);
        
        // Remove data older than 12 months
        if (monthsOld > 12) {
            delete allMonthsData[key];
        }
    });
    
    localStorage.setItem('allMonthsPortfolioData', JSON.stringify(allMonthsData));
}
```

## Performance Optimization

### Lazy Loading for Large Datasets

```javascript
// Only render visible rows
let visibleRowsRange = { start: 0, end: 50 };

function renderVisibleRows() {
    const tbody = document.getElementById('table-body');
    tbody.innerHTML = '';
    
    for (let i = visibleRowsRange.start; i < visibleRowsRange.end; i++) {
        if (i >= portfolioData.contracts.length) break;
        if (portfolioData.hiddenRows.has(i)) continue;
        
        // Render row i
        // ... row rendering code ...
    }
}

// Add scroll listener to update visible range
```

### Debounced Auto-Save

```javascript
let saveTimeout;

function debouncedSave() {
    clearTimeout(saveTimeout);
    saveTimeout = setTimeout(() => {
        saveCurrentMonthData();
    }, 1000); // Save 1 second after last change
}

// Use debouncedSave() instead of saveCurrentMonthData() in frequent operations
```

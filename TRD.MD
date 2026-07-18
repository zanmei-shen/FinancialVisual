# Technical Requirements Document
## Financial Statement Visualizer with Income Statement Sankey Diagram

**Version:** 1.0  
**Date:** 2026-07-18  
**Status:** Draft for Review  

---

## 1. Executive Summary

This document defines the technical requirements for building a responsive web application that allows users to query financial data by stock ticker, select financial statement types, and visualize the data. A core feature is an interactive **Income Statement Sankey Diagram** that visualizes the flow of revenue through cost and expense nodes to final net income, using width-proportional flows and a specific color-coded design system.

---

## 2. Functional Requirements

### 2.1 User Input & Controls
| ID     | Requirement                                                                                                                                            | Priority  |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------- |
| FR-001 | **Stock Input Field**: Free-text input accepting stock ticker symbols (e.g., `AAPL`, `MSFT`) or company names with autocomplete/typeahead suggestions. | Must Have |
| FR-002 | **Statement Type Dropdown**: Selectable options including: Income Statement, Balance Sheet, Cash Flow Statement.                                       | Must Have |
| FR-003 | **Submit Button**: Triggers data fetch and visualization update. Must include loading state and disable duplicate submissions while fetching.          | Must Have |
| FR-004 | **Input Validation**: Validate ticker format (1-5 uppercase letters) and provide user feedback for invalid inputs.                                     | Must Have |


### 2.2 Data Fetching
| ID     | Requirement                                                                                                                             | Priority   |
|--------|-----------------------------------------------------------------------------------------------------------------------------------------|------------|
| FR-005 | **API Integration**: Fetch financial data from a reliable financial data provider (e.g., Alpha Vantage, Finnhub, Yahoo Finance, Polygon.io). | Must Have  |
| FR-006 | **Data Parsing**: Parse JSON response into structured financial statement objects with standardized field names.                        | Must Have  |
| FR-007 | **Error Handling**: Handle API failures (rate limits, invalid tickers, network errors) with user-friendly error messages.               | Must Have  |
| FR-008 | **Caching**: Client-side caching of recent queries to reduce API calls and improve performance.                                         | Should Have |


### 2.3 Data Display (General)
| ID     | Requirement                                                                                                              | Priority   |
|--------|---------------------------------------------------------------------------------------------------------------------------|------------|
| FR-009 | **Tabular View**: Display raw financial data in a responsive, sortable table for all statement types.                    | Must Have  |
| FR-010 | **Unit Formatting**: Display monetary values in billions/millions with `$` prefix and appropriate decimal places (e.g., `$27.2B`, `$450M`). | Must Have  |
| FR-011 | **YoY Calculation**: Calculate and display Year-over-Year percentage changes where applicable.                            | Should Have |


### 2.4 Income Statement Sankey Diagram (Core Feature)
| ID     | Requirement                                                                                                                                                                                                 | Priority   |
|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------|
| FR-012 | **Conditional Rendering**: Sankey diagram renders **only** when "Income Statement" is selected. Other statements show tabular/chart views.                                                                  | Must Have  |
| FR-013 | **Node Structure - Left (Revenue)**: Display total revenue node with segmented revenue sources (e.g., Product Revenue, Service Revenue, Geographic segments if available).                                   | Must Have  |
| FR-014 | **Node Structure - Center (Costs & Expenses)**: Display COGS node and Operating Expenses node. Gross Profit must be calculated and displayed as the residual flow between Revenue and COGS.                  | Must Have  |
| FR-015 | **Node Structure - Right (Profits)**: Display Operating Income, Taxes, Other Income/Expenses, and final Net Income node. Operating Income = Gross Profit - OpEx. Net Income = Operating Income - Taxes - Other. | Must Have  |
| FR-016 | **Flow Width Proportionality**: The width of each Sankey link must be directly proportional to the absolute dollar amount it represents.                                                                    | Must Have  |
| FR-017 | **Node Labels**: Each node must display:<br>• **Value**: Dollar amount (e.g., `$27.2B`)<br>• **Margin**: Percentage relative to total revenue (e.g., `68%`)<br>• **YoY Change**: Optional indicator (e.g., `↑ 12%`) | Must Have  |
| FR-018 | **Color Palette**: Strict adherence to:<br>• Revenue segments: Cool colors (blues, teals, purples)<br>• COGS: Red/Orange tones<br>• Operating Expenses: Yellow/Orange tones<br>• Profits: Green tones<br>• Taxes/Other: Neutral/gray tones | Must Have  |
| FR-019 | **Interactivity**: Hover states on nodes and links showing exact values. Click‑to‑drill‑down optional for revenue segments.                                                                                 | Should Have |
| FR-020 | **Responsiveness**: Sankey diagram must reflow and remain readable on mobile devices (320px+), tablets, and desktops.                                                                                       | Must Have  |


---

## 3. Non-Functional Requirements

### 3.1 Performance
| ID       | Requirement                                               |
|----------|-----------------------------------------------------------|
| NFR-001  | Initial page load < 2 seconds on 4G connection.           |
| NFR-002  | API data fetch and render < 3 seconds for standard queries. |
| NFR-003  | Sankey diagram animation/transition < 500ms after data load. |
| NFR-004  | Support for last 2 years of major browser versions.       |


### 3.2 Responsive Design
| ID       | Requirement                                                                                                      |
|----------|------------------------------------------------------------------------------------------------------------------|
| NFR-005  | **Mobile (320px–767px)**: Stacked layout, Sankey diagram scrollable horizontally or simplified vertical flow.    |
| NFR-006  | **Tablet (768px–1024px)**: Side‑by‑side layout with adapted Sankey node spacing.                                 |
| NFR-007  | **Desktop (1025px+)**: Full three‑column Sankey layout with ample spacing and hover tooltips.                    |


### 3.3 Accessibility
| ID       | Requirement                                                                                                      |
|----------|------------------------------------------------------------------------------------------------------------------|
| NFR-008  | **WCAG 2.1 AA compliance** for color contrast (especially important for red/green profit/loss indicators).       |
| NFR-009  | Keyboard‑navigable controls and ARIA labels for screen readers.                                                  |
| NFR-010  | Sankey diagram must include text alternatives or data tables for screen‑reader users.                            |

---

## 4. Technical Architecture

### 4.1 Tech Stack Recommendations

| Layer                   | Technology                                   | Justification                                                                                     |
|-------------------------|-----------------------------------------------|---------------------------------------------------------------------------------------------------|
| **Frontend Framework**  | React 18+ / Vue 3 / Svelte                   | Component-based architecture, reactive state management for data fetching and visualization updates. |
| **Visualization Library** | D3.js (d3-sankey)                          | Industry standard for custom Sankey diagrams with precise control over node positioning, flow widths, and interactivity. |
| **UI Component Library** | Tailwind CSS + Headless UI / Material-UI    | Rapid responsive development, accessible form controls.                                           |
| **State Management**     | React Query (TanStack Query) / SWR          | Excellent for server state caching, background refetching, and loading/error state handling.      |
| **Build Tool**           | Vite                                         | Fast development server and optimized production builds.                                          |
| **API**                  | RESTful JSON                                 | Standard financial data APIs (Alpha Vantage, Finnhub).                                            |


### 4.2 Data Flow Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│   User      │────▶│   Frontend   │────▶│  Financial API  │
│  Input      │     │   (React)    │     │  (Alpha Vantage)│
└─────────────┘     └──────────────┘     └─────────────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │  Transform   │
                    │   Layer      │
                    │ (Normalize   │
                    │  API data)   │
                    └──────────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌─────────┐  ┌──────────┐  ┌──────────┐
        │  Table  │  │  Sankey  │  │  Error   │
        │  View   │  │  Diagram │  │  Handler │
        └─────────┘  └──────────┘  └──────────┘
```

---

## 5. Data Model & API Specification

### 5.1 Financial Data Schema (Normalized)

```typescript
interface FinancialStatement {
  ticker: string;
  statementType: 'income' | 'balance' | 'cashflow';
  fiscalDate: string;
  currency: string;
  data: {
    // Income Statement Fields
    totalRevenue: number;
    revenueSegments?: { name: string; amount: number }[];
    costOfRevenue: number;
    grossProfit: number;
    operatingExpenses: number;
    operatingIncome: number;
    taxProvision: number;
    otherExpenses: number;
    netIncome: number;
    
    // Balance Sheet / Cash Flow fields...
  };
}
```

### 5.2 Sankey Data Transformer

The application must include a transformer module that converts the normalized income statement into D3-sankey compatible nodes and links:

```typescript
interface SankeyNode {
  id: string;
  name: string;
  value: number;       // Absolute dollar amount
  margin: number;      // Percentage of total revenue
  yoyChange?: number;  // Optional YoY percentage
  category: 'revenue' | 'cogs' | 'opex' | 'profit' | 'tax' | 'other';
}

interface SankeyLink {
  source: string;      // Node ID
  target: string;      // Node ID
  value: number;       // Dollar amount flowing between nodes
}
```

### 5.3 Sankey Node Mapping Logic

| Source Node        | Target Node        | Value Calculation                                               |
|--------------------|--------------------|------------------------------------------------------------------|
| Revenue Segments   | Total Revenue      | Segment amount                                                   |
| Total Revenue      | COGS               | `costOfRevenue`                                                  |
| Total Revenue      | Gross Profit       | `totalRevenue - costOfRevenue`                                   |
| Gross Profit       | Operating Expenses | `operatingExpenses`                                              |
| Gross Profit       | Operating Income   | `grossProfit - operatingExpenses`                                |
| Operating Income   | Taxes              | `taxProvision`                                                   |
| Operating Income   | Other              | `otherExpenses`                                                  |
| Operating Income   | Net Income         | `operatingIncome - taxProvision - otherExpenses`                 |


---

## 6. UI/UX Specifications

### 6.1 Layout Structure

```
┌─────────────────────────────────────────────┐
│  Header: Financial Statement Visualizer     │
├─────────────────────────────────────────────┤
│  [Stock Input ▼] [Statement ▼] [Submit]     │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │                                     │   │
│  │      SANKEY DIAGRAM (if Income)     │   │
│  │                                     │   │
│  │   [Revenue]───>[COGS]────>[OpInc]   │   │
│  │      │              │         │     │   │
│  │      └──>[Gross]──>[OpEx]    [Tax]  │   │
│  │              │              [Other] │   │
│  │              └──────────>[Net Inc]  │   │
│  │                                     │   │
│  └─────────────────────────────────────┘   │
│                                            │
│  ┌─────────────────────────────────────┐   │
│  │      DATA TABLE (all types)         │   │
│  └─────────────────────────────────────┘   │
│                                            │
└────────────────────────────────────────────┘
```

### 6.2 Sankey Diagram Visual Specifications

| Element           | Specification                              |
|-------------------|----------------------------------------------|
| **Node Width**    | 20px fixed                                   |
| **Node Padding**  | 15px vertical between nodes in same column   |
| **Link Opacity**  | 0.4 default, 0.7 on hover                    |
| **Link Color**    | Source node color with gradient to target    |
| **Label Font**    | 12px sans-serif, white text on dark nodes    |
| **Value Format**  | `$XX.XB` / `$XXXM` / `$XXXK`                 |
| **Margin Format** | `XX.X%` displayed below value                |
| **YoY Indicator** | `↑ XX%` green / `↓ XX%` red, right of node   |


### 6.3 Color Palette (Exact)

| Category            | Hex Codes                                                                 | Usage                     |
|---------------------|---------------------------------------------------------------------------|---------------------------|
| **Revenue (Cool)**  | `#3B82F6` (Blue), `#06B6D4` (Cyan), `#8B5CF6` (Purple)                    | Revenue nodes and flows   |
| **COGS**            | `#EF4444` (Red), `#F97316` (Orange)                                       | Cost of Revenue nodes     |
| **OpEx**            | `#EAB308` (Yellow), `#F97316` (Orange)                                    | Operating Expense nodes   |
| **Profits**         | `#22C55E` (Green), `#10B981` (Emerald)                                    | Profit-related nodes      |
| **Taxes**           | `#6B7280` (Gray)                                                          | Tax provision nodes       |
| **Other**           | `#9CA3AF` (Light Gray)                                                    | Other income/expense      |

---

## 7. API Integration Details

### 7.1 Recommended Data Providers

| Provider        | Endpoint Pattern                                           | Rate Limit            | Notes                               |
|-----------------|-------------------------------------------------------------|------------------------|--------------------------------------|
| **Alpha Vantage** | `query=INCOME_STATEMENT&symbol={ticker}`                 | 5 calls/min (free)     | Reliable, requires API key           |
| **Finnhub**       | `/api/v1/stock/financials-reported?symbol={ticker}`      | 60 calls/min (free)    | Real-time data, good coverage        |
| **Polygon.io**    | `/vX/reference/financials?ticker={ticker}`               | 5 calls/min (free)     | High quality, US-focused             |

### 7.2 API Request/Response Example

**Request:**
```http
GET https://www.alphavantage.co/query?function=INCOME_STATEMENT&symbol=AAPL&apikey=XXX
```

**Key Response Fields (Mapped):**
```json
{
  "annualReports": [{
    "fiscalDateEnding": "2023-09-30",
    "totalRevenue": "383285000000",
    "costOfRevenue": "214137000000",
    "grossProfit": "169148000000",
    "operatingExpenses": "26844000000",
    "operatingIncome": "114301000000",
    "incomeTaxExpense": "16741000000",
    "netIncome": "96995000000"
  }]
}
```

---

## 8. Implementation Milestones

| Phase       | Deliverables                                                                                   | Duration |
|-------------|-------------------------------------------------------------------------------------------------|----------|
| **Phase 1** | Project setup, UI shell, input controls, responsive layout                                      | 1 week   |
| **Phase 2** | API integration, data fetching, error handling, caching                                         | 1 week   |
| **Phase 3** | Data table component for all statement types                                                    | 3 days   |
| **Phase 4** | Sankey diagram core: D3 setup, node/link rendering, flow widths                                 | 1 week   |
| **Phase 5** | Sankey styling: colors, labels, margins, YoY indicators, tooltips                               | 3 days   |
| **Phase 6** | Responsive Sankey adaptation (mobile/tablet)                                                    | 2 days   |
| **Phase 7** | Accessibility audit, performance optimization, testing                                          | 3 days   |


---

## 9. Testing Requirements

### 9.1 Test Cases (Sankey Specific)

| Test ID | Scenario                               | Expected Result                                                             |
|---------|-----------------------------------------|------------------------------------------------------------------------------|
| TC-001  | Valid ticker, Income Statement selected | Sankey renders with correct node values and proportional flows               |
| TC-002  | Revenue = $100B, COGS = $60B            | Gross Profit node width = 40% of Revenue width                               |
| TC-003  | Negative net income (loss)              | Net Income node displays in red, flow direction maintained                   |
| TC-004  | Mobile viewport (375px)                 | Diagram scrolls horizontally or switches to vertical layout                  |
| TC-005  | API returns missing segment data        | Sankey falls back to single Total Revenue node                               |
| TC-006  | Colorblind user (WCAG)                  | Pattern/texture overlays or labels ensure distinguishability                 |


---

## 10. Dependencies & External Libraries

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "d3": "^7.8.0",
    "d3-sankey": "^0.12.3",
    "@tanstack/react-query": "^5.0.0",
    "tailwindcss": "^3.4.0",
    "lucide-react": "^0.300.0",
    "date-fns": "^3.0.0"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "vite": "^5.0.0",
    "vitest": "^1.0.0",
    "playwright": "^1.40.0"
  }
}
```

---

## 11. Risk Assessment

| Risk                               | Impact | Mitigation                                                                                     |
|------------------------------------|--------|-------------------------------------------------------------------------------------------------|
| API rate limiting                  | High   | Implement client-side caching, request queuing, and user feedback for limits                   |
| Data inconsistency across providers| Medium | Normalize data schema internally; validate incoming data against expected structure             |
| Complex Sankey layout on mobile    | Medium | Implement progressive disclosure; offer simplified view for small screens                       |
| Color accessibility failures       | Medium | Use patterns/textures alongside colors; test with colorblind simulators                         |


---

## 12. Appendices

### Appendix A: D3-Sankey Configuration Reference

```javascript
const sankey = d3.sankey()
  .nodeWidth(20)
  .nodePadding(15)
  .extent([[0, 0], [width, height]])
  .nodeAlign(d3.sankeyLeft); // or sankeyCenter, sankeyRight

// Node alignment: Left for Revenue, Right for Net Income
```

### Appendix B: Margin Calculation Formula

```
Node Margin = (Node Value / Total Revenue) × 100
```

### Appendix C: Responsive Breakpoints

| Breakpoint | Width      | Sankey Behavior                                   |
|------------|------------|---------------------------------------------------|
| Mobile     | < 768px    | Vertical scroll or simplified 2-column layout     |
| Tablet     | 768–1024px | 3-column layout, reduced node padding             |
| Desktop    | > 1024px   | Full 3-column layout, full interactivity          |


---

**Document Owner:** Development Team  
**Reviewers:** Product Manager, UI/UX Designer, Tech Lead  
**Next Review Date:** 2026-07-25  

---

This TRD provides the complete technical foundation for your project. The Sankey diagram implementation using **D3.js** will give you the precise control needed for the proportional flow widths, custom color palette, and node labeling requirements you've specified.


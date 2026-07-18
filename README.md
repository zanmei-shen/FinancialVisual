# Financial Statement Visualizer

A responsive web application that fetches financial data via the Finnhub API and displays it in interactive formats. For income statements, the app generates a **Sankey diagram** that visualizes the flow of revenue through costs and expenses to final net income.

![Income Statement Sankey Diagram](docs/sankey-preview.png)

## Features

- **Stock Search**: Input any stock ticker or company name to fetch financial data
- **Statement Selection**: Choose between Income Statement, Balance Sheet, or Cash Flow Statement
- **Interactive Sankey Diagram** (Income Statement only):
  - Width-proportional flows representing dollar amounts
  - Revenue segments on the left, costs in the center, profits on the right
  - Node labels show value, margin %, and YoY change
  - Color-coded: cool blues for revenue, red/orange for COGS, yellow for OpEx, green for profits
- **Responsive Design**: Works on mobile, tablet, and desktop
- **Data Table**: Sortable tabular view for all statement types
- **Client-side Caching**: Reduces API calls and improves performance

## Tech Stack

| Layer | Technology |
|-------|------------|
| Framework | React 18 + TypeScript |
| Styling | Tailwind CSS |
| Visualization | D3.js + d3-sankey |
| Data Fetching | TanStack Query (React Query) |
| Build Tool | Vite |
| Testing | Vitest + Playwright |

## Getting Started

### Prerequisites

- Node.js 18+ and npm/yarn/pnpm
- A free Finnhub API key ([get one here](https://finnhub.io/register))

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/financial-statement-visualizer.git
cd financial-statement-visualizer

# Install dependencies
npm install

# Create environment file
cp .env.example .env.local
# Edit .env.local and add your Finnhub API key:
# VITE_FINNHUB_API_KEY=your_api_key_here

# Start development server
npm run dev
```

The app will be available at `http://localhost:5173`.

### Build for Production

```bash
npm run build
```

Output will be in the `dist/` directory.

## Project Structure

```
├── public/
├── src/
│   ├── api/
│   │   ├── finnhub.ts          # Finnhub API client
│   │   └── types.ts            # API response types
│   ├── components/
│   │   ├── SearchBar.tsx       # Ticker input + statement dropdown
│   │   ├── SubmitButton.tsx    # Fetch trigger with loading state
│   │   ├── DataTable.tsx       # Sortable financial data table
│   │   └── sankey/
│   │       ├── SankeyDiagram.tsx    # Main D3 Sankey wrapper
│   │       ├── SankeyNode.tsx       # Individual node component
│   │       ├── SankeyLink.tsx       # Flow link component
│   │       └── SankeyTooltip.tsx    # Hover tooltip
│   ├── hooks/
│   │   ├── useFinancialData.ts      # TanStack Query hook
│   │   └── useSankeyData.ts         # Transform hook: statement → sankey nodes/links
│   ├── utils/
│   │   ├── formatters.ts            # $XX.XB, margin %, YoY formatting
│   │   ├── sankeyTransformer.ts     # Income statement → D3-sankey data
│   │   └── validators.ts            # Ticker input validation
│   ├── styles/
│   │   └── sankey-colors.ts         # Color palette constants
│   ├── App.tsx
│   └── main.tsx
├── tests/
│   ├── unit/
│   └── e2e/
├── .env.example
├── vite.config.ts
├── tailwind.config.js
└── package.json
```

## Sankey Diagram Layout

```
Left Column                    Center Column                  Right Column
┌─────────────┐                ┌─────────────┐               ┌─────────────┐
│  Revenue    │───────────────▶│    COGS     │               │             │
│  Segments   │                │  (red)      │               │             │
│  (cool)     │───────────────▶│             │               │             │
└─────────────┘                └──────┬──────┘               │             │
       │                              │                      │   Operating │
       │                              ▼                      │   Income    │
       │                       ┌─────────────┐               │   (green)   │
       │                       │ Gross Profit│──────────────▶│             │
       │                       │  (green)    │               │             │
       │                       └──────┬──────┘               │             │
       │                              │                      │             │
       │                              ▼                      │             │
       │                       ┌─────────────┐               │             │
       │                       │ Operating   │               │             │
       │                       │  Expenses   │               │             │
       │                       │ (yellow)    │               │             │
       │                       └──────┬──────┘               │             │
       │                              │                      │   Net       │
       │                              ▼                      │   Income    │
       │                       ┌─────────────┐               │   (green)   │
       │                       │   Taxes     │──────────────▶│             │
       │                       │   (gray)    │               │             │
       │                       └─────────────┘               │             │
       │                                                     │             │
       │                       ┌─────────────┐               │             │
       │                       │    Other    │──────────────▶│             │
       │                       │   (gray)    │               │             │
       │                       └─────────────┘               └─────────────┘
```

### Flow Rules

| Source | Target | Value |
|--------|--------|-------|
| Revenue Segments | Total Revenue | Segment amount |
| Total Revenue | COGS | `costOfRevenue` |
| Total Revenue | Gross Profit | `totalRevenue - costOfRevenue` |
| Gross Profit | Operating Expenses | `operatingExpenses` |
| Gross Profit | Operating Income | `grossProfit - operatingExpenses` |
| Operating Income | Taxes | `taxProvision` |
| Operating Income | Other | `otherExpenses` |
| Operating Income | Net Income | `operatingIncome - taxes - other` |

### Color Palette

| Category | Color | Hex |
|----------|-------|-----|
| Revenue (cool) | Blue | `#3B82F6` |
| Revenue (cool) | Cyan | `#06B6D4` |
| Revenue (cool) | Purple | `#8B5CF6` |
| COGS | Red | `#EF4444` |
| COGS | Orange | `#F97316` |
| Operating Expenses | Yellow | `#EAB308` |
| Operating Expenses | Orange | `#F97316` |
| Profits | Green | `#22C55E` |
| Profits | Emerald | `#10B981` |
| Taxes | Gray | `#6B7280` |
| Other | Light Gray | `#9CA3AF` |

## API Configuration

### Finnhub Free Tier Limits

| Limit | Value |
|-------|-------|
| Rate | 60 calls / minute |
| Daily | ~300 calls / day |
| WebSocket | Up to 50 symbols |
| Commercial use | ❌ Not allowed |

### Endpoints Used

```
GET https://finnhub.io/api/v1/stock/financials-reported
  ?symbol={TICKER}
  &freq=annual
  &token={API_KEY}

GET https://finnhub.io/api/v1/stock/profile2
  ?symbol={TICKER}
  &token={API_KEY}
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `VITE_FINNHUB_API_KEY` | Yes | Your Finnhub API key |
| `VITE_API_BASE_URL` | No | Override Finnhub base URL |
| `VITE_CACHE_STALE_TIME` | No | TanStack Query stale time (ms), default 300000 |

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Vite dev server |
| `npm run build` | Build for production |
| `npm run preview` | Preview production build locally |
| `npm run test` | Run unit tests (Vitest) |
| `npm run test:e2e` | Run end-to-end tests (Playwright) |
| `npm run lint` | Run ESLint |
| `npm run typecheck` | Run TypeScript compiler |

## Testing

### Unit Tests

```bash
npm run test
```

Covers:
- Ticker input validation
- Sankey data transformation logic
- Number formatting utilities
- Margin and YoY calculations

### E2E Tests

```bash
npm run test:e2e
```

Scenarios:
- Search for valid ticker → data loads → Sankey renders
- Search for invalid ticker → error message shown
- Switch statement type → view updates correctly
- Mobile responsive layout verification

## Responsive Breakpoints

| Breakpoint | Width | Sankey Behavior |
|------------|-------|-----------------|
| Mobile | < 768px | Vertical scroll or simplified 2-column layout |
| Tablet | 768–1024px | 3-column layout, reduced node padding |
| Desktop | > 1024px | Full 3-column layout, hover tooltips |

## Roadmap

- [ ] Multi-year comparison (side-by-side Sankey)
- [ ] Export Sankey as PNG/SVG
- [ ] Dark mode support
- [ ] Additional chart types (waterfall, treemap)
- [ ] Commercial tier upgrade path

## License

MIT — Free for personal and non-commercial use.

> **Note:** Finnhub's free tier is non-commercial only. If you plan to monetize this project, you will need to upgrade to a paid Finnhub plan or switch to a commercial-friendly provider.

## Acknowledgments

- [D3.js](https://d3js.org/) and [d3-sankey](https://github.com/d3/d3-sankey) for the visualization engine
- [Finnhub](https://finnhub.io/) for financial data
- [Tailwind CSS](https://tailwindcss.com/) for styling
- [TanStack Query](https://tanstack.com/query) for data fetching

---

Built with React, D3, and financial curiosity.

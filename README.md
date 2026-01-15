# Top Performing Stocks Analysis - 2025

## Overview

This project identifies the top performing investable US stocks for 2025 by analyzing 6000+ tickers using a liquidity-aware ranking methodology. The enhanced pipeline uses a **multi-tier ticker selection strategy** combined with the proven 4-phase processing approach to efficiently identify the best performers while ensuring practical investability.

### Enhanced Features

- **Universe**: Analyzes 6000+ tickers from multiple sources (S&P 500, NASDAQ 100, Russell 1000, high-volume SEC tickers)
- **Multi-Tier Selection**: Intelligent pre-filtering to focus on liquid, investable stocks
- **Robust Processing**: Enhanced batching, checkpointing, and error recovery for large datasets
- **Comprehensive Ranking**: Composite score (70% return, 30% liquidity) to balance performance and tradability

## Ticker Selection Strategy

The pipeline uses a **multi-tier approach** to select 6000+ high-quality tickers:

### Tier 1: Major Index Constituents (~5000 tickers)
- **S&P 500**: Large-cap US stocks representing ~80% of US market cap
- **NASDAQ 100**: Leading tech and growth companies
- **Russell 1000**: Broad coverage of US large/mid-cap stocks

### Tier 2: High-Volume Filter (~200+ tickers)
- Additional tickers from SEC list filtered by:
  - Consistent trading volume (minimum daily dollar volume threshold)
  - Valid ticker format (≤ 4 characters for common stocks)
  - Active trading status

### Tier 3: Quality Filters (Applied to all tickers)
- **Price Filter**: Minimum $1.00 end-of-year price (survival/compliance filter)
- **Liquidity Filter**: Minimum $500K average daily dollar volume (Q4 2025)
- **Data Quality**: Valid price history and no suspicious data artifacts

---

## Phase-by-Phase Explanation

### Phase 1: Survival Probe
Identifies stocks that survived to the end of 2025 and meet the minimum price threshold ($1.00). This filters out:
- Delisted stocks
- Stocks trading below exchange compliance thresholds
- Data artifacts and invalid tickers

### Phase 2: Origin Probe
Retrieves starting prices from end of 2024 to calculate annual returns. Stocks without valid 2024 prices are flagged as potential IPOs.

### Phase 3: IPO Discovery
For stocks that didn't exist in 2024, downloads full 2025 history to:
- Find exact IPO listing date
- Capture IPO price (first trade)
- Calculate return from IPO to year-end

### Phase 4: Liquidity Audit & Ranking
1. Calculates **avg daily $ volume (Q4 2025: Oct–Dec 2025)** for the top 200 by return
2. Filters stocks below $500K minimum liquidity threshold
3. Creates composite score: 70% return + 30% liquidity
4. Returns top 100 ranked by composite score

---

## Enhanced Market Analysis Pipeline

The pipeline efficiently processes 60000+ tickers using a proven 4-phase approach with enhanced scalability:

### Pipeline Overview

1. **Ticker Universe Creation**
   - Combines multiple sources (S&P 500, NASDAQ 100, Russell 1000, high-volume SEC tickers)
   - Deduplicates and validates against SEC master list
   - Exports curated universe to `market_universe_checkpoint.csv`

2. **Phase 1: Survival Probe** (with progress tracking)
   - Identifies stocks still trading at end of 2025 (price > $1.00)
   - Filters out delisted stocks and data artifacts
   - **Batching**: Processes 20 tickers per batch with rate limiting
   - **Checkpoint**: Saves results to `phase1_survival.csv`

3. **Phase 2: Origin Probe** (with resume capability)
   - Gets starting prices from end of 2024 
   - Filters data glitches and validates price history
   - Flags potential 2025 IPOs for Phase 3
   - **Checkpoint**: Saves results to `phase2_origin.csv`

4. **Phase 3: IPO Discovery** (with detailed logging)
   - Downloads full 2025 history for IPO candidates
   - Identifies exact listing date and IPO price
   - Calculates return from IPO to year-end
   - **Checkpoint**: Saves results to `phase3_ipo.csv`

5. **Phase 4: Liquidity Audit & Ranking** (top 200 → filter → rank)
   - Computes **avg daily $ volume (Q4 2025: Oct–Dec 2025)** for top 200 by return
   - Filters stocks below $500K liquidity threshold
   - Creates composite score: **70% Return + 30% Liquidity Score**
   - Returns top 100 ranked by composite score (with return as tiebreaker)
   - **Checkpoint**: Saves results to `phase4_liquidity.csv`

### Enhanced Features

- **Robust Batching**: Smaller batch sizes (20 vs 50) for better rate limit handling
- **Exponential Backoff**: Automatic retry logic for API failures
- **Resume Capability**: Phase-specific checkpoints allow pipeline restart from any phase
- **Memory Optimization**: Chunk processing and garbage collection for large datasets
- **Progress Tracking**: Detailed logging with ETAs and ticker counts per phase

---

## Understanding the Output

The CSV export includes:
- **Return**: Price multiple (P_end / P_start)
- **Return_Score**: Normalized return score (0-1) for composite calculation
- **Avg_Daily_Dollar_Vol**: Avg daily $ volume (Q4 2025: Oct–Dec 2025)
- **Liquidity_Score**: Log-normalized liquidity score (0-1) for composite calculation
- **Composite_Score**: Final ranking score (70% return + 30% liquidity)

**Ranking Logic**: Stocks are ranked by Composite_Score, which balances high returns with sufficient liquidity for practical investability. This ensures the top stocks are both high-performing AND tradeable.

---

## Export Results and Comprehensive Reporting

The analysis generates multiple output files for thorough review:

### Primary Outputs

1. **Top 100 Ranked List** (`universe/top_100_2025.csv`)
   - Final ranked performers by composite score
   - Includes: Symbol, Return, Start/End Price, Liquidity Metrics, Composite Score
   
2. **Full 6000+ Analysis** (`reports/top_6000_analysis_2025.xlsx`)
   - Multi-sheet Excel workbook with:
     - **Top 100**: Final ranked list with all metrics
     - **Full 6000**: All processed tickers and their performance
     - **Sector Breakdown**: Performance distribution by sector
     - **Source Comparison**: Returns by ticker source (S&P 500, NASDAQ 100, etc.)
     - **Methodology**: Documentation of selection and ranking approach

### Visualizations

The dashboard includes:
- **Top 30 Performers**: Bar chart with color-coded liquidity scores
- **Return Distribution**: Histogram of returns across all 6000+ tickers
- **Liquidity vs Return**: Scatter plot showing the tradeoff frontier
- **Sector Performance**: Box plots by industry sector
- **IPO vs Established**: Separate analysis of 2025 IPOs vs existing stocks
- **Source Breakdown**: Performance comparison across ticker sources

### Checkpoint Files

Phase-specific checkpoints saved in `universe/phase_checkpoints/`:
- `phase1_survival.csv` - Stocks surviving to end of 2025
- `phase2_origin.csv` - Starting prices and return calculations
- `phase3_ipo.csv` - IPO discoveries with listing dates
- `phase4_liquidity.csv` - Top 200 candidates with liquidity metrics

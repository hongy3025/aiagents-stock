# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A multi-AI agent stock analysis system for Chinese A-share markets built with Python + Streamlit + DeepSeek. The system simulates an analyst team providing comprehensive stock investment analysis and trading decision support.

**Primary Language:** Chinese (UI, comments, documentation)

## Build & Run Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run locally (two options)
streamlit run app.py
# or
python run.py

# Docker deployment
docker-compose up -d
# Access at: http://localhost:8503
```

## Architecture

### Core Application Flow
```
Streamlit UI (app.py)
    ↓
Feature Modules (longhubang_ui.py, sector_strategy_ui.py, etc.)
    ↓
Engine Layer (longhubang_engine.py, smart_monitor_engine.py, etc.)
    ↓
AI Agents (ai_agents.py, longhubang_agents.py, sector_strategy_agents.py)
    ↓
DeepSeek Client (deepseek_client.py)
    ↓
Data Layer (stock_data.py, data_source_manager.py)
    ↓
Database (SQLite via peewee ORM)
```

### Module Pattern
Each feature follows a consistent structure:
- `*_data.py` - Data fetching
- `*_db.py` - Database model (SQLite)
- `*_agents.py` - AI agent definitions
- `*_engine.py` - Business logic
- `*_ui.py` - Streamlit UI component
- `*_scheduler.py` - Scheduled tasks (if applicable)
- `*_pdf.py` - PDF report generation

### Key Components
- **StockAnalysisAgents** (`ai_agents.py`): Coordinates 4 analysis agents (technical, fundamental, fund flow, risk)
- **DeepSeekClient** (`deepseek_client.py`): OpenAI SDK wrapper for DeepSeek API with reasoner model support
- **StockDataFetcher** (`stock_data.py`): Multi-source data with fallback (TDX → AKShare → Tushare → YFinance)
- **NotificationService** (`notification_service.py`): Email + Webhook (DingTalk/Feishu)

### Data Sources Priority
1. TDX API (local, fastest) - configurable via `TDX_ENABLED`
2. AKShare (Chinese market primary)
3. Tushare (backup)
4. YFinance (international)

## Key Environment Variables

```env
DEEPSEEK_API_KEY          # Required - AI analysis
TUSHARE_TOKEN             # Optional - backup data source
TDX_ENABLED               # Optional - local TDX data
MINIQMT_ENABLED           # Optional - trading execution
EMAIL_ENABLED             # Optional - email notifications
WEBHOOK_ENABLED           # Optional - DingTalk/Feishu webhooks
```

## Feature Modules

| Module | Purpose |
|--------|---------|
| Stock Analysis | Multi-agent stock analysis with PDF export |
| 主力选股 (Main Force) | Institutional capital flow analysis |
| 智瞰龙虎 (Longhubang) | Dragon Tiger List analysis |
| 智策 (Sector Strategy) | Sector rotation strategy |
| AI盯盘 (Smart Monitor) | Real-time AI trading decisions |
| 持仓分析 (Portfolio) | Portfolio batch analysis with scheduling |
| 实时监测 (Monitor) | Price alert monitoring |

## Code Conventions

- Snake_case for files and functions
- CamelCase for class names
- Private methods prefixed with `_`
- Chinese comments for domain-specific logic
- JSON serialization for database storage
- Pandas DataFrames for data processing
- Plotly for interactive charts

## Important Files

- `app.py` - Main entry point, routing, ~3000 lines
- `ai_agents.py` - Core multi-agent coordinator
- `stock_data.py` - Data fetching with fallback logic
- `deepseek_client.py` - LLM API wrapper
- `config.py` / `config_manager.py` - Configuration management
- `.streamlit/config.toml` - Streamlit settings (port 8503)

## A-Share Market Rules

The system implements T+1 trading rules:
- Stocks bought today cannot be sold until next trading day
- Trading hours: 9:30-11:30, 13:00-15:00
- `SmartMonitorEngine` enforces these constraints

## Documentation

Extensive documentation in `docs/` folder (60+ markdown files). Key docs:
- `docs/QUICK_START.md` - Getting started
- `docs/DOCKER_DEPLOYMENT.md` - Container deployment
- `docs/PORTFOLIO_USAGE.md` - Portfolio analysis
- Feature-specific guides for each module
